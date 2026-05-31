---
name: photo-editing-person-removal
description: "Professionelle Foto-Pipeline: Person entfernen, Personen in neue Szenen setzen. Korrekte Methode: KI füllt NUR die Lücke, Originalpersonen werden pixel-genau drüber geklebt."
version: 3.0.0
author: Luna
platforms: [macos, linux]
prerequisites:
  python_env: "/Users/aven/comfy_venv/bin/python3"
  comfyui: "/Users/aven/Documents/comfy/ComfyUI"
  openai_key_env: "VOICE_TOOLS_OPENAI_KEY"
---

# Foto-Bearbeitung: Person entfernen

## Das Wichtigste zuerst (aus schmerzhafter Erfahrung)

**GOLDENE REGEL: Originalpersonen kommen IMMER aus dem Originalbild.**  
KI-Output (SD, DALL-E, LaMa) darf NIEMALS für Personen verwendet werden — nur für Hintergrund/Lücke.

```
Original-Foto
    ├── Person links  →  direkt aus Original pixel-kopieren ✅
    ├── LÜCKE         →  KI füllt hier (Bank, Hintergrund) ✅
    └── Person rechts →  direkt aus Original pixel-kopieren ✅
```

## Korrekte Methode (bewährt)

### Schritt 1: Lücke per OpenAI API füllen

```python
import os, json, base64, urllib.request
from PIL import Image, ImageFilter
import numpy as np

OPENAI_KEY = None
with open(os.path.expanduser("~/.hermes/.env")) as f:
    for line in f:
        if line.startswith("VOICE_TOOLS_OPENAI_KEY="):
            OPENAI_KEY = line.strip().split("=",1)[1]
            break

ORIG = "/path/to/foto.jpeg"
img = Image.open(ORIG).convert("RGBA")
OW, OH = img.size

# OpenAI braucht 1024x1024 PNG
SIZE = 1024
img_sq = img.resize((SIZE, SIZE), Image.LANCZOS)

# Maske: NUR die Lücke transparent (NICHT die Personen!)
# Wichtig: Koordinaten sorgfältig messen — Personen müssen KOMPLETT außerhalb liegen
mx1 = int(LEFT_CUT / OW * SIZE)
mx2 = int(RIGHT_CUT / OW * SIZE)

arr = np.array(img_sq).copy()
arr[:,:,3] = 255              # alles opak = preserve
arr[:, mx1:mx2, 3] = 0       # nur Lücke transparent = inpaint
alpha = Image.fromarray(arr[:,:,3]).filter(ImageFilter.GaussianBlur(5))
arr[:,:,3] = np.array(alpha)
masked = Image.fromarray(arr)

# Originalbild opak speichern
orig_arr = np.array(img_sq).copy()
orig_arr[:,:,3] = 255
Image.fromarray(orig_arr).save("/tmp/bg_orig.png")
masked.save("/tmp/bg_mask.png")

# API Call
boundary = "----BoundaryXXX"
def field(n,v): return (f"--{boundary}\\r\\nContent-Disposition: form-data; name=\\"{n}\\"\\r\\n\\r\\n{v}\\r\\n").encode()
def ffile(n,fn,d): return (f"--{boundary}\\r\\nContent-Disposition: form-data; name=\\"{n}\\"; filename=\\"{fn}\\"\\r\\nContent-Type: image/png\\r\\n\\r\\n").encode() + d + b"\\r\\n"

with open("/tmp/bg_orig.png","rb") as f: od=f.read()
with open("/tmp/bg_mask.png","rb") as f: md=f.read()

body = (ffile("image","image.png",od) + ffile("mask","mask.png",md) +
    field("model","gpt-image-1") +
    field("prompt","Fill the transparent area with the natural background visible in the image (bench, rocks, grass). No people.") +
    field("n","1") + field("size","1024x1024") +
    f"--{boundary}--\\r\\n".encode())

req = urllib.request.Request("https://api.openai.com/v1/images/edits", data=body,
    headers={"Authorization":f"Bearer {OPENAI_KEY}",
             "Content-Type":f"multipart/form-data; boundary={boundary}"})

resp = urllib.request.urlopen(req, timeout=120)
result = json.loads(resp.read())
bg_bytes = base64.b64decode(result["data"][0]["b64_json"])
with open("/tmp/bg_only.png","wb") as f: f.write(bg_bytes)
```

### Schritt 2: Originalpersonen drüber kleben

```python
bg_full   = Image.open("/tmp/bg_only.png").convert("RGB").resize((OW, OH), Image.LANCZOS)
orig_rgb  = Image.open(ORIG).convert("RGB")
bg_arr    = np.array(bg_full)
orig_arr2 = np.array(orig_rgb)

# Nur Lücke von KI, alles andere 100% original
result_arr = bg_arr.copy()
result_arr[:, :LEFT_CUT]  = orig_arr2[:, :LEFT_CUT]   # Person links — 100% original
result_arr[:, RIGHT_CUT:] = orig_arr2[:, RIGHT_CUT:]  # Person rechts — 100% original

# Zusammenschieben (optional): Dame + kleiner Lücke + Herr
GAP = 80
left  = result_arr[:, :LEFT_CUT]
mid   = result_arr[:, LEFT_CUT:LEFT_CUT+GAP]
right = result_arr[:, RIGHT_CUT:]
final = np.concatenate([left, mid, right], axis=1)

Image.fromarray(final).save("/tmp/result.jpeg", quality=95)
```

## Schnittgrenzen finden (kritisch)

Vor dem Schneiden das Originalbild **visuell analysieren**:

```python
# Debug-Crops für Koordinaten-Check
img.crop((0, 80, 900, 550)).save("/tmp/debug_left_heads.png")    # Kopf-Zone links
img.crop((1100, 80, 2000, 550)).save("/tmp/debug_right_heads.png") # Kopf-Zone rechts
```

- LEFT_CUT: rechte Kante der linken Person **inklusive Kopf** — großzügig wählen
- RIGHT_CUT: linke Kante der rechten Person **inklusive Kopf** — großzügig wählen
- Beide Köpfe müssen **komplett außerhalb** der Maske liegen

## Maske-Sicherheitscheck

```python
# Vorschau: roten Overlay auf Maskenbereich zeigen
vis = np.array(img_sq.convert("RGB"))
vis[:, mx1:mx2, 0] = np.clip(vis[:, mx1:mx2, 0].astype(int)*0.5 + 127, 0, 255)
Image.fromarray(vis.astype(np.uint8)).save("/tmp/mask_preview.png")
# → Prüfen: Sind beide Personen AUSSERHALB des roten Bereichs?
```

## Alternative: LaMa (lokal, kostenlos)

Für einfache Fälle ohne starke Überlappung:

```python
# Nur mit /Users/aven/comfy_venv/bin/python3 — NICHT System-Python 3.9!
import sys
sys.path.insert(0, '/Users/aven/comfy_venv/lib/python3.13/site-packages')
from simple_lama_inpainting import SimpleLama
from PIL import Image, ImageDraw, ImageFilter

lama = SimpleLama()
# Bild auf 1024-1200px skalieren für LaMa
img_s = img.resize((1200, 900), Image.LANCZOS)
# ... Maske erstellen ...
result = lama(img_s, mask)
# Dann gleich: Originalpersonen drüber kleben (s.o.)
```

## Personen in neue Szene (rembg + Compositing)

```python
from rembg import remove
person = Image.open("person.jpg")
person_rgba = remove(person)  # transparenter Hintergrund
bg = Image.open("new_background.jpg").convert("RGBA")
bg.paste(person_rgba, position, person_rgba)
bg.convert("RGB").save("result.jpg")
```

Für identitätstreue KI-Generierung: IP-Adapter in ComfyUI (weight=0.8, denoise=0.3).

## Pitfalls (alle aus echten Fehlern)

1. **KI-Output für Personen nutzen** → generiert immer andere Personen → NIEMALS
2. **gpt-image-1 ohne Maske** → generiert komplett neues Bild → immer `mask` Feld mitgeben
3. **dall-e-2** → funktioniert nicht mit `sk-proj-` Keys → `gpt-image-1` nutzen
4. **Maske zu groß** → schneidet Köpfe/Körper an → Debug-Crop vorher machen
5. **System-Python 3.9** für LaMa → CUDA-Fehler → immer comfy_venv nutzen
6. **SD denoise=1.0** → generiert neue Personen → 0.2-0.35 für Personenerhalt
7. **Naht sichtbar** → Farbabgleich: Durchschnittsfarbe links/rechts messen, Lücke angleichen
8. **`response_format` Parameter** → bei gpt-image-1 nicht unterstützt → weglassen

## Setup (Mac Mini)

```bash
# Python-Umgebung
/Users/aven/comfy_venv/bin/python3  # ← immer diese!

# LaMa + rembg (bereits installiert)
/Users/aven/comfy_venv/bin/pip install simple-lama-inpainting rembg --no-deps
/Users/aven/comfy_venv/bin/pip install opencv-python-headless onnxruntime pooch pymatting scikit-image

# OpenAI Key
grep VOICE_TOOLS_OPENAI_KEY ~/.hermes/.env
```
