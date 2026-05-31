# OpenAI Images Edit API — Quirks & Pitfalls

Gesammelt aus Session 2026-05-28.

## Endpoint
`POST https://api.openai.com/v1/images/edits`

## Key-Typ auf Mac Mini
`VOICE_TOOLS_OPENAI_KEY` in `~/.hermes/.env` — ist ein `sk-proj-` Project Key.
→ Unterstützt NUR `gpt-image-1`, NICHT `dall-e-2`!

## gpt-image-1 Inpainting

### Pflichtfelder (multipart/form-data)
- `image` — PNG, RGBA, 1024x1024, max 4MB
- `mask` — PNG, RGBA, transparent = inpaint, opak = preserve
- `model` — `"gpt-image-1"` (Pflicht! Ohne → 400 "Missing required parameter")
- `prompt` — Beschreibung was in die transparente Zone soll
- `n` — Anzahl (meistens "1")
- `size` — `"1024x1024"`

### NICHT unterstützte Parameter (→ 400 Fehler)
- `response_format` → weglassen! Output ist immer b64_json
- `dall-e-2` als model → "model does not exist" mit sk-proj- Keys

### Response-Format
```json
{"data": [{"b64_json": "iVBORw0KGgo..."}]}
```
→ `result["data"][0]["b64_json"]` → base64 dekodieren → PNG Bytes

## Maske korrekt erstellen

```python
arr = np.array(img_sq).copy()
arr[:,:,3] = 255          # alles opak = PRESERVE
arr[:, mx1:mx2, 3] = 0   # nur Lücke transparent = INPAINT
# Weiche Ränder empfohlen:
alpha = Image.fromarray(arr[:,:,3]).filter(ImageFilter.GaussianBlur(5))
arr[:,:,3] = np.array(alpha)
```

## Wichtig: nur Hintergrund inpaiten, Personen 100% original

gpt-image-1 verändert Personen immer leicht, auch wenn sie außerhalb der Maske sind.
→ Lösung: Maske = nur echter Lückenbereich → KI füllt nur Hintergrund → Originalpersonen drüber kleben.

## Kosten
gpt-image-1 edit: ca. $0.04 pro Bild (1024x1024)
