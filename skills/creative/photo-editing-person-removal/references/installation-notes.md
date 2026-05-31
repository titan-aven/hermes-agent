# Installation Notes — Mac Mini (M4, 16GB, macOS 26.5)

## Funktionierende Installation

```bash
# comfy_venv hat Python 3.13 + PyTorch MPS (Apple Silicon)
/Users/aven/comfy_venv/bin/pip install simple-lama-inpainting --no-deps
/Users/aven/comfy_venv/bin/pip install rembg --no-deps
/Users/aven/comfy_venv/bin/pip install opencv-python-headless onnxruntime pooch pymatting scikit-image
```

## Was NICHT funktioniert

### `pip install simple-lama-inpainting` (ohne --no-deps)
→ Versucht Pillow 9.5.0 aus dem Quellcode zu bauen
→ Schlägt fehl mit `KeyError: '__version__'` in setup.py
→ Ursache: Python 3.13 inkompatibel mit Pillow 9.5 Build-System
→ Fix: `--no-deps` + neuere Pillow ist bereits im venv vorhanden ✅

### System-Python 3.9 (`/usr/bin/python3`)
→ Hat altes PyTorch das CUDA erwartet (kein MPS)
→ `SimpleLama()` crasht mit `NotImplementedError: Could not run 'aten::empty_strided' with arguments from the 'CUDA' backend`
→ Fix: Immer `/Users/aven/comfy_venv/bin/python3` nutzen ✅

### `simple-lama-inpainting` mit großen Bildern (2000x1500)
→ Kein Fehler, aber langsamer und weniger Qualität
→ Fix: Vorher auf 1200x900 skalieren, danach wieder hochskalieren ✅

### comfy-cli `--fast-deps` auf Python 3.9
→ `comfy-kitchen` benötigt Python ≥ 3.10
→ Fix: comfy-cli im comfy_venv (Python 3.13) installieren ✅

## Script-Header für alle LaMa-Scripts

```python
import sys
sys.path.insert(0, '/Users/aven/comfy_venv/lib/python3.13/site-packages')
```

Nötig wenn das Script via system `python3` aufgerufen wird statt via comfy_venv.

## Modell-Cache
- LaMa Modell: `~/.cache/torch/hub/checkpoints/big-lama.pt` (196MB)
- Wird beim ersten `SimpleLama()` Aufruf automatisch heruntergeladen
