---
name: photo-person-removal
description: "Remove a person from a photo and close the gap — PIL/OpenCV for simple cases, SD inpainting for overlapping subjects."
version: 1.0.0
tags: [photo-editing, inpainting, stable-diffusion, comfyui, opencv, pillow]
---

# Photo Person Removal

Remove one or more people from a photo and make the remaining subjects appear naturally together.

## Critical Decision Point: Does the subject overlap others?

**Check this first before writing any code.**

If the person to remove **overlaps** the people being kept (arms touching, shared bench, bodies in front of each other), then:
- ❌ Programmatic pixel blending WILL produce ghost artefacts — no amount of feathering fixes this
- ❌ Alpha compositing WILL produce ghost artefacts
- ❌ Seamless cloning WILL blend overlapping pixels in
- ✅ **AI Inpainting is the only correct approach**

If the person to remove is **cleanly separated** (pure gap between them, no pixel overlap):
- ✅ Simple crop + splice works
- ✅ Background fill + seamless clone works

## Path A: Clean Separation (no overlap)

```python
import cv2, numpy as np

img = cv2.imread(src)
H, W = img.shape[:2]

# Hard cut — NO blending over person pixels, ever
left  = img[:, :LEFT_CUT]
right = img[:, RIGHT_CUT:]

# Gap fill: sample from a clean background region (far edge of frame)
bg_strip = img[:, 0:GAP].copy()  # pure BG from left edge

canvas = np.zeros((H, LEFT_CUT + GAP + right.shape[1], 3), dtype=np.uint8)
canvas[:, :LEFT_CUT]          = left
canvas[:, LEFT_CUT:LEFT_CUT+GAP] = bg_strip
canvas[:, LEFT_CUT+GAP:]      = right

# Blend ONLY in pure background zones (sky, grass at top) — not over clothing/skin
cv2.imwrite(dst, canvas)
```

**Pitfall:** Do NOT blend at the person boundary. Only blend where there is 100% background on both sides of the seam.

## Path B: Overlapping Subjects → SD1.5 Inpainting (ComfyUI)

See the `comfyui` skill for setup. Quick checklist:

1. **Create mask** — white = inpaint region, black = preserve
   ```python
   from PIL import Image, ImageFilter
   mask = Image.new("RGB", img.size, "black")
   draw.rectangle([x1, y1, x2, y2], fill="white")
   mask.convert("L").filter(ImageFilter.GaussianBlur(15)).save("mask.png")
   ```

2. **Model:** `sd-v1-5-inpainting.ckpt` (HuggingFace: runwayml/stable-diffusion-inpainting, ~4 GB)
   ```bash
   curl -L -o ~/Documents/comfy/ComfyUI/models/checkpoints/sd-v1-5-inpainting.ckpt \
     "https://huggingface.co/runwayml/stable-diffusion-inpainting/resolve/main/sd-v1-5-inpainting.ckpt"
   ```

3. **Upload to ComfyUI:**
   ```bash
   curl -X POST "http://127.0.0.1:8188/upload/image" -F "image=@input.png" -F "type=input" -F "overwrite=true"
   curl -X POST "http://127.0.0.1:8188/upload/image" -F "image=@mask.png"  -F "type=input" -F "overwrite=true"
   ```

4. **Workflow nodes** (API format):
   - `CheckpointLoaderSimple` → `sd-v1-5-inpainting.ckpt`
   - `LoadImage` → input image
   - `LoadImageMask` → mask, channel=`red`
   - `VAEEncodeForInpaint` (NOT regular VAEEncode) → `grow_mask_by=6`
   - `KSampler` → steps=30, cfg=7.5, sampler=`euler_a`, scheduler=`karras`, denoise=1.0
   - Positive prompt: describe the background that should appear (bench, rocks, grass, etc.)
   - Negative prompt: `person, human, people, face, body`

5. **After inpainting:** composite the two kept-persons back onto the inpainted background using hard crop + OpenCV seamlessClone.

## Typical Workflow for Group Photo (3 → 2 persons)

1. Diagnose overlap → use inpainting
2. Scale image to 768×576 (SD1.5 native; 2000×1500 causes MPS INT_MAX crash)
3. Create mask over middle person only — keep outer persons fully outside mask
4. Run SD1.5 inpaint → get clean background in the masked zone
5. Hard-crop left person from **original full-res** photo
6. Hard-crop right person from **original full-res** photo
7. Scale inpainted background strip up to full-res height
8. Composite: left_orig + bg_strip + right_orig (hard paste — no blend on persons)
9. Blend ONLY where pure background meets pure background at seams (top sky/grass zone)

## ComfyUI Setup on macOS Apple Silicon (M-Series)

**Critical: use Python ≥ 3.10. The system Python 3.9 fails with `comfy-kitchen` dependency.**

```bash
# 1. Create venv with Homebrew Python 3.13
/opt/homebrew/bin/python3.13 -m venv ~/comfy_venv
~/comfy_venv/bin/pip install comfy-cli

# 2. Disable analytics
~/comfy_venv/bin/comfy --skip-prompt tracking disable

# 3. Install ComfyUI
~/comfy_venv/bin/comfy --skip-prompt install --m-series

# If already cloned (e.g. partial install from a failed run):
~/comfy_venv/bin/comfy --skip-prompt install --m-series --restore
# Note: --restore may fail too if comfy-cli picks up the wrong python.
# Fallback: install deps manually:
~/comfy_venv/bin/python3 -m pip install --pre torch torchvision torchaudio
~/comfy_venv/bin/python3 -m pip install -r ~/Documents/comfy/ComfyUI/requirements.txt

# 4. Launch
cd ~/Documents/comfy/ComfyUI && ~/comfy_venv/bin/python3 main.py --listen 127.0.0.1 --port 8188 &

# 5. Download SD1.5 inpainting model (~4 GB)
curl -L -o ~/Documents/comfy/ComfyUI/models/checkpoints/sd-v1-5-inpainting.ckpt \
  "https://huggingface.co/runwayml/stable-diffusion-inpainting/resolve/main/sd-v1-5-inpainting.ckpt"
```

## Submitting Jobs via Python (not curl+shell)

Shell JSON escaping is unreliable for ComfyUI payloads. Always use Python:

```python
import json, urllib.request

HOST = "http://127.0.0.1:8188"
with open("workflow.json") as f:
    workflow = json.load(f)

payload = json.dumps({"prompt": workflow, "client_id": "my-client"}).encode()
req = urllib.request.Request(f"{HOST}/prompt", data=payload,
    headers={"Content-Type": "application/json"})
try:
    resp = json.loads(urllib.request.urlopen(req).read())
    print("Prompt ID:", resp["prompt_id"])
except urllib.error.HTTPError as e:
    print("Error:", e.code, e.read().decode())  # always read error body
```

## Polling for completion

```python
import json, urllib.request, time

def wait_for_result(host, prompt_id, timeout=600):
    for i in range(timeout // 5):
        time.sleep(5)
        hist = json.loads(urllib.request.urlopen(f"{host}/history/{prompt_id}").read())
        if hist:
            data = hist[prompt_id]
            status = data.get("status", {})
            if status.get("completed"):
                outputs = {}
                for node_id, node_data in data.get("outputs", {}).items():
                    outputs[node_id] = node_data.get("images", [])
                return outputs
            if status.get("status_str") == "error":
                for m in status.get("messages", []):
                    if m[0] == "execution_error":
                        raise RuntimeError(m[1].get("exception_message"))
    raise TimeoutError("Job did not complete in time")
```

## Pitfalls

- **1 failed PIL/OpenCV attempt on overlapping persons → immediately switch to inpainting.** Don't try feathering, seamless clone, or alpha blend. They all produce ghosts.
- **Input image must be ≤ 512–768px wide for SD1.5 on MPS.** 2000×1500 causes `MPSGraph does not support tensor dims larger than INT_MAX`. Scale down before uploading: `img.resize((768, 576), Image.LANCZOS)`.
- **Sampler name is `euler_ancestral`, not `euler_a`** — ComfyUI rejects `euler_a` with a `value_not_in_list` 400 error.
- **VAEEncodeForInpaint vs VAEEncode** — always use `VAEEncodeForInpaint` or the mask is ignored.
- **LoadImageMask reads `red` channel by default** — save mask as RGB (not alpha).
- **Mask should only cover the person being removed** — if it bleeds onto kept subjects, SD regenerates those persons too (completely different face/clothes). After inpainting, always paste kept persons back from the original.
- **Positive prompt = the background that should appear** — describe bench material, rocks, grass, etc. Negative prompt = `person, human, people, face, body`.
- **Don't blend at person-pixel boundaries** — blend only where 100% background is on both sides of the seam (typically top 200px of outdoor photos).
- **Shell JSON payload escaping breaks silently** — the prompt gets submitted with zero errors but the job never appears in history. Always use Python `urllib.request` for submissions.
- **comfy-cli installs to wrong python** — if `comfy --skip-prompt install --m-series` crashes on `pip_install_comfyui_dependencies` with CalledProcessError, comfy-cli picked up the wrong python. Fix: create a dedicated venv and install everything manually.
