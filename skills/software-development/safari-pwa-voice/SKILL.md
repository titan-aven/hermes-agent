---
name: safari-pwa-voice
description: "Build hands-free voice PWAs for iPhone Safari: Whisper STT, AudioContext playback, Tailscale HTTPS, LaunchAgent autostart."
tags: [pwa, safari, iphone, voice, whisper, tts, ios, audio]
platforms: [macos]
---

# Safari PWA Voice Interface

Build a hands-free voice dialog PWA installable on iPhone via Safari "Add to Home Screen".
Validated on mac-mini + Tailscale + OpenAI Whisper + Anthropic Claude.

## When to Use

- User wants a voice interface with an AI agent on iPhone
- Building a ChatGPT Advanced Voice Mode equivalent
- Deploying a hands-free PWA accessible via Tailscale

## Architecture

```
iPhone Safari PWA
      ↓ WebSocket
FastAPI Server (Mac Mini, port 8765, HTTPS via tailscale cert)
      ↓ ffmpeg (absolute path!) → wav
OpenAI Whisper API (STT)
      ↓
Anthropic API (LLM — Claude Haiku for low latency)
      ↓
OpenAI TTS API (wav — NOT mp3, iOS AudioContext handles wav universally)
      ↓ base64 over WebSocket
Pre-unlocked AudioContext.decodeAudioData() → playback
```

## API Key Management

The Luna Voice server (`~/luna-voice-build/server/main.py`) reads `OPENAI_API_KEY` from the
**LaunchAgent environment**, not a `.env` file. There is no `.env` in `server/` by default.

**To rotate the key:**
```python
import plistlib, shutil
path = '/Users/aven/Library/LaunchAgents/com.aven.luna-voice.plist'
shutil.copy(path, path + '.bak')
with open(path, 'rb') as f:
    d = plistlib.load(f)
d.setdefault('EnvironmentVariables', {})['OPENAI_API_KEY'] = 'NEW_KEY'
with open(path, 'wb') as f:
    plistlib.dump(d, f)
```
Then: `launchctl kickstart -k gui/$(id -u)/com.aven.luna-voice`

**Do NOT copy Aven's key** from `~/.openclaw/openclaw.json` → `skills.entries.openai-whisper-api.apiKey` — it may be expired, and Aven will refuse to share it (Security Red Line).

## Critical Safari Pitfalls

### 1. Audio Recording Format — ffmpeg Conversion (MANDATORY)
Safari declares `audio/webm;codecs=opus` but sends non-standard containers.
Prefer ffmpeg conversion to WAV, but **always use an absolute path** and **check the return code** — LaunchAgent services run with no `$PATH`, so bare `"ffmpeg"` is silently not found, leading to `FileNotFoundError` on the output `.wav` file. Always fall back to raw bytes if ffmpeg fails (Whisper supports webm natively).

**ALWAYS capture stderr from ffmpeg** (use `asyncio.subprocess.PIPE`, not `DEVNULL`) so you can see why conversion failed. Silent discard masks the real error.

```python
proc = await asyncio.create_subprocess_exec(
    FFMPEG_PATH, "-y", "-i", tmp_in_path, "-ar", "16000", "-ac", "1", "-f", "wav", tmp_out_path,
    stdout=asyncio.subprocess.DEVNULL,
    stderr=asyncio.subprocess.PIPE,   # NOT DEVNULL — you need this for diagnosis
)
_, stderr_data = await proc.communicate()  # NOT proc.wait() — use communicate() to drain pipe
if proc.returncode != 0:
    log.warning(f"ffmpeg failed (rc={proc.returncode}): {stderr_data.decode(errors='replace')[-300:]}")
ffmpeg_ok = (proc.returncode == 0) and os.path.exists(tmp_out_path)
```

```python
import asyncio, tempfile, os, io

FFMPEG_PATH = "/opt/homebrew/bin/ffmpeg"  # absolute — LaunchAgent has no $PATH

# MIME type → file extension map — ffmpeg uses the extension for format detection!
# CRITICAL: do NOT use a generic suffix like ".input" — ffmpeg will return rc=183
# "Invalid data found when processing input" because it can't detect the format.
EXT_MAP = {
    "audio/webm": ".webm",
    "audio/webm;codecs=opus": ".webm",
    "audio/mp4": ".mp4",
    "audio/mp4;codecs=mp4a": ".mp4",
    "audio/aac": ".aac",
    "audio/ogg": ".ogg",
    "audio/ogg;codecs=opus": ".ogg",
}

async def transcribe(audio_bytes: bytes, mime_type: str = "audio/mp4") -> str:
    in_ext = EXT_MAP.get(mime_type.split(";")[0].strip(), ".webm")

    with tempfile.NamedTemporaryFile(suffix=in_ext, delete=False) as tmp_in:
        tmp_in.write(audio_bytes)
        tmp_in_path = tmp_in.name
    tmp_out_path = tmp_in_path.replace(in_ext, ".wav")
    try:
        ffmpeg_ok = False
        if os.path.exists(FFMPEG_PATH):
            proc = await asyncio.create_subprocess_exec(
                FFMPEG_PATH, "-y", "-i", tmp_in_path,
                "-ar", "16000", "-ac", "1", "-f", "wav", tmp_out_path,
                stdout=asyncio.subprocess.DEVNULL,
                stderr=asyncio.subprocess.PIPE,  # PIPE not DEVNULL — need stderr for diagnosis
            )
            _, stderr_data = await proc.communicate()  # communicate() not wait() — drains PIPE
            ffmpeg_ok = (proc.returncode == 0) and os.path.exists(tmp_out_path)
            if not ffmpeg_ok:
                log.warning(f"ffmpeg failed rc={proc.returncode}: {stderr_data.decode(errors='replace')[-200:]}")

        if ffmpeg_ok:
            with open(tmp_out_path, "rb") as f:
                buf = io.BytesIO(f.read())
            buf.name = "audio.wav"
        else:
            # Fallback: send raw bytes with correct extension
            buf = io.BytesIO(audio_bytes)
            buf.name = f"audio{in_ext}"

        return (await openai.audio.transcriptions.create(
            model="whisper-1", file=buf, language="de"
        )).text
    finally:
        for p in [tmp_in_path, tmp_out_path]:
            try: os.unlink(p)
            except: pass
```

**Root cause patterns:**
- `No such file or directory: '/tmp/tmp....input.wav'` → ffmpeg failed silently, output never created. Check `launchctl getenv PATH` — if empty, PATH missing (use absolute path).
- `ffmpeg failed (rc=183): Invalid data found when processing input` → temp file had wrong extension (e.g. `.input`). ffmpeg uses the extension for format detection. Fix: use correct extension from MIME type.
- `STT error: The audio file could not be decoded` → ffmpeg failed AND raw fallback also rejected → check both the extension and the audio data itself.

### 2. Audio Playback — Safari Autoplay Policy
`new Audio().play()` is blocked by Safari unless triggered by direct user tap.
Use `AudioContext` pre-unlocked during startup (see Pitfall #7), with a separate context for input (analyser/VAD) and output (TTS playback):

```js
async playAudio(base64) {
    const ctx = this.playbackCtx;  // pre-unlocked output context
    if (!ctx) return;
    if (ctx.state === 'suspended') await ctx.resume();

    return new Promise(async (resolve) => {
      const timeout = setTimeout(() => resolve(), 15000);  // never get stuck
      try {
        const bytes    = base64ToUint8Array(base64);
        const arrayBuf = bytes.buffer.slice(0);  // slice(0) = copy — Safari detaches original
        const decoded  = await ctx.decodeAudioData(arrayBuf);
        const source   = ctx.createBufferSource();
        source.buffer  = decoded;
        source.connect(ctx.destination);
        source.onended = () => { clearTimeout(timeout); resolve(); };
        source.start(0);
      } catch (e) {
        clearTimeout(timeout);
        // Fallback: reuse pre-unlocked HTMLAudioElement
        try {
          const bytes2 = base64ToUint8Array(base64);
          const blob   = new Blob([bytes2], { type: 'audio/wav' });
          const url    = URL.createObjectURL(blob);
          const audio  = this.audioEl || new Audio();
          audio.src    = url;
          const done   = () => { URL.revokeObjectURL(url); resolve(); };
          audio.onended = done;
          audio.onerror = () => done();
          await audio.play();
        } catch { resolve(); }
      }
    });
}

### 3. HTTPS is Required
Safari blocks microphone access on non-HTTPS origins.
Use Tailscale's built-in cert provisioning:
```bash
tailscale cert mac-mini-von-aven.tail15c773.ts.net
# Outputs: mac-mini-von-aven.tail15c773.ts.net.crt + .key
```
Pass to uvicorn:
```bash
uvicorn main:app --ssl-keyfile ssl/key.pem --ssl-certfile ssl/cert.pem --port 8765
```

### 10. API Keys Must NEVER Live in Browser-Accessible Files ⚠️ SECURITY

**Anti-pattern:** Calling `fetch('https://api.openai.com/v1/audio/speech', { headers: { Authorization: 'Bearer sk-...' } })` directly from `app.js`, with the key loaded from a `config.local.js` served as a static file.

Even if the key is in `config.local.js` (gitignored), if the file is served by the same static file server, **anyone who opens DevTools → Network tab sees the key in plain text.** This includes the TTS call pattern:

```js
// ❌ WRONG — API key exposed in browser
fetch('https://api.openai.com/v1/audio/speech', {
  headers: { Authorization: `Bearer ${CONFIG.OPENAI_API_KEY}` },
  body: JSON.stringify({ model: 'tts-1', input: text, voice: 'echo' })
});
```

**Correct fix: server-side proxy for ALL third-party API calls:**

```js
// ✅ CORRECT — key stays server-side
const res = await fetch('/api/tts', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ text }),
});
const mp3Blob = await res.blob();
```

Server-side (`server.js` or `server.py`):
```js
if (req.method === 'POST' && req.url === '/api/tts') {
  const { text } = await readBody(req);
  // Forward to OpenAI with key from server environment — never sent to browser
  const ttsRes = await openaiPost('/v1/audio/speech', { model: 'tts-1', input: text, voice: TTS_VOICE }, OPENAI_API_KEY);
  res.writeHead(200, { 'Content-Type': 'audio/mpeg' });
  res.end(ttsRes.body);
}
```

**Security checklist for voice PWA architecture:**
- `/api/transcribe` → Whisper → ✅ server-proxied
- `/api/chat` → LLM → ✅ server-proxied  
- `/api/tts` → OpenAI TTS → must be server-proxied (easy to miss!)
- `config.local.js` → if served statically, contains keys visible in Network tab → move keys to `.env` / `process.env`, never serve them as files

**Detection:** In `dogfood` testing, always check if any `fetch()` calls in `app.js` go directly to third-party domains with `Authorization` headers populated from a `CONFIG.*` object. If yes → critical security finding.

### 4. Hermes is NOT an HTTP API
Hermes Agent has no `/v1/chat/completions` endpoint.
Call Anthropic API directly with a Luna system prompt:
```python
async def call_luna(messages):
    async with httpx.AsyncClient(timeout=90) as client:
        resp = await client.post(
            "https://api.anthropic.com/v1/messages",
            headers={"x-api-key": ANTHROPIC_KEY, "anthropic-version": "2023-06-01"},
            json={"model": "claude-haiku-4-5-20251001", "max_tokens": 1024,
                  "system": LUNA_SYSTEM_PROMPT, "messages": messages},
        )
        return resp.json()["content"][0]["text"]
```

### 5. MediaRecorder MIME Type Fallback
Safari iOS may not report `MediaRecorder.isTypeSupported()` correctly.
Force fallback to `audio/mp4`:
```js
const candidates = ['audio/webm;codecs=opus', 'audio/webm', 'audio/mp4', ''];
this.mimeType = candidates.find(t => t === '' || MediaRecorder.isTypeSupported(t));
if (!this.mimeType || this.mimeType === '') this.mimeType = 'audio/mp4';
```

### 0. Service Worker Cache — CHECK FIRST BEFORE ANY DEBUG ⚠️

**Symptom:** You deploy a fix, user reports "nothing changed" — old JS still cached on device.
**Trap:** Cache name `luna-v1` never changes → all fixes silently ignored. User tests old code forever.

**Fix: bump cache name AND cache-bust script tag on every deploy:**
```js
const CACHE = 'luna-v7'; // increment on every deploy
```
```html
<script src="/app.js?v=7"></script>
```
**Force-clear on iPhone:** Settings → Apps → Safari → Clear History and Website Data.
**If installed as PWA (home screen icon):** Delete the icon first — PWA cache is independent of Safari cache.
**During active development:** Disable Service Worker entirely, re-enable only once everything works.

### 6. WebM Multi-Turn STT — Use New MediaRecorder Per Utterance ⚠️
**This is the #1 silent killer of multi-turn voice conversations.**

Safari/Chrome's MediaRecorder writes the **WebM EBML container headers only into the very first `ondataavailable` chunk**. All subsequent chunks are raw "continuation data" — NOT valid standalone WebM files. ffmpeg and Whisper both fail with `Invalid data found when processing input` / HTTP 400.

**Symptom:** First utterance works, all subsequent utterances fail STT → user sees "Nachdenken..." loop forever with no response.

**CORRECT FIX — new MediaRecorder per utterance:**

Create a brand-new `MediaRecorder` for each utterance (start on speech detection, stop on silence). Every resulting Blob automatically contains the full WebM headers. No chunk-prepending tricks needed.

```js
// When speech is detected, start a fresh recorder:
startRecorder() {
  const opts = this.mimeType ? { mimeType: this.mimeType } : {};
  this.recorder = new MediaRecorder(this.stream, opts);

  this.recorder.ondataavailable = e => {
    if (e.data && e.data.size > 0) this.chunks.push(e.data);
  };

  this.recorder.onstop = () => {
    if (this.chunks.length === 0) return;
    const blob = new Blob(this.chunks, { type: this.mimeType || 'audio/mp4' });
    this.chunks = [];
    this.sendAudio(blob);  // every blob is a complete valid file
  };

  this.recorder.start();
}

// When silence detected, stop — onstop fires and sends the blob:
flushUtterance() {
  if (this.recorder && this.recorder.state !== 'inactive') {
    this.recorder.stop();
  }
}
```

**Why NOT the initChunk prepend approach:** The initChunk approach (saving first chunk and prepending to subsequent utterances) is unreliable in practice — timing issues, Safari quirks, and the chunk boundary not always aligning with the EBML end cause rc=183 "Invalid data" errors. New-recorder-per-utterance is robust and simple.

**Log pattern that reveals this bug:**
```
ffmpeg failed (rc=183): Invalid data found when processing input
STT error: The audio file could not be decoded or its format is not supported.
```

### 7. iOS Safari AudioContext Unlock — Must Happen in User Gesture

`AudioContext.decodeAudioData()` and `HTMLAudioElement.play()` are **silently blocked** on iOS outside a direct user gesture. By the time TTS audio arrives over WebSocket, the gesture is long gone. You must pre-unlock both during the button click:

```js
// Inside the startSession() handler (called from button click — IS a user gesture):

// 1. Unlock AudioContext with a silent buffer
const playbackCtx = new AudioContext();
await playbackCtx.resume();
const silence = playbackCtx.createBuffer(1, 1, 22050);
const src = playbackCtx.createBufferSource();
src.buffer = silence;
src.connect(playbackCtx.destination);
src.start(0);  // This "uses" the context with a user gesture — it's now unlocked

// 2. Pre-unlock HTMLAudioElement (fallback path)
const audioEl = new Audio();
audioEl.src = 'data:audio/wav;base64,UklGRiQAAABXQVZFZm10IBAAAAABAAEAgD4AAAB9AAACABAAZGF0YQAAAAA=';
await audioEl.play().catch(() => {});
audioEl.src = '';  // now this element is unlocked for future .play() calls
```

Later in `playAudio()`, reuse `this.playbackCtx` and `this.audioEl` — **do not create new instances** outside the gesture.

### 8. TTS Format: Use WAV not MP3 for iOS Safari

OpenAI TTS supports multiple output formats. Use `response_format="wav"` — iOS Safari's `AudioContext.decodeAudioData()` handles WAV universally. MP3 decoding via AudioContext is unreliable on iOS (codec support varies by device/iOS version).

```python
resp = await openai.audio.speech.create(
    model="tts-1",
    voice=TTS_VOICE,
    input=text,
    response_format="wav",   # NOT "mp3" — iOS AudioContext decodes WAV reliably
)
```

Also update the Blob MIME type in the HTML Audio fallback path to `audio/wav` to match.

### 9. Service Worker Cache — MUST Bump Version on Every Client Change ⚠️

The PWA Service Worker caches `app.js` and other assets. If the cache version string is not changed, Safari will serve the **old cached JS indefinitely** — no matter how many server-side fixes you deploy. This is the most insidious debugging trap: you fix the code, restart the server, test again, still broken — because the phone never loaded the new code.

**Rule:** Every time you change `app.js`, `styles.css`, or `index.html`, bump the cache version in `service-worker.js`:

```js
// service-worker.js — bump this on EVERY client file change
const CACHE = 'luna-v7';  // ← increment this
```

**Also add a cache-busting query param to the script tag in index.html:**
```html
<script src="/app.js?v=7"></script>  <!-- matches cache version -->
```

**How to force a fresh load on the user's device:**
1. Bump CACHE version in `service-worker.js`
2. Add/bump `?v=N` query param on `<script src="/app.js">` in `index.html`
3. User must: iPhone Settings → Apps → Safari → "Verlauf und Websitedaten löschen"
4. Or: delete PWA from home screen and reinstall

**Diagnosis:** If you've deployed a fix but the user says nothing changed, ALWAYS suspect the Service Worker cache first. Ask the user to open `/app.js` directly in Safari and search for a known-new string (e.g. a log message you just added) to confirm which version is running.

## LaunchAgent Autostart

```xml
<!-- scripts/com.aven.luna-voice.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "...">
<plist version="1.0"><dict>
  <key>Label</key><string>com.aven.luna-voice</string>
  <key>ProgramArguments</key><array>
    <string>/path/to/.venv/bin/uvicorn</string>
    <string>main:app</string>
    <string>--host</string><string>0.0.0.0</string>
    <string>--port</string><string>8765</string>
    <string>--ssl-keyfile</string><string>/path/to/ssl/key.pem</string>
    <string>--ssl-certfile</string><string>/path/to/ssl/cert.pem</string>
  </array>
  <key>RunAtLoad</key><true/>
  <key>KeepAlive</key><true/>
  <key>WorkingDirectory</key><string>/path/to/project/server</string>
</dict></plist>
```

Install/reload:
```bash
launchctl unload ~/Library/LaunchAgents/com.aven.luna-voice.plist 2>/dev/null
launchctl load   ~/Library/LaunchAgents/com.aven.luna-voice.plist
```

## iPhone Installation

1. Tailscale active on iPhone
2. Open Safari → `https://<tailscale-hostname>:8765`
3. Share button → "Add to Home Screen"
4. Done — icon appears, no App Store needed

## Debugging

```bash
# Live logs
tail -f ~/luna-voice-build/logs/luna-voice-error.log

# Key log lines to look for:
# ✅ "recorder started" → new MediaRecorder per utterance (correct approach)
# ✅ "ffmpeg OK → WAV"
# ✅ "User: <transcribed text>"
# ✅ "Luna: <response>"
# ✅ "HTTP/1.1 200 OK" for TTS
# ✅ "decoded, duration=Xs" + "playback ended" → audio actually played
# ❌ "ffmpeg failed (rc=183): Invalid data found" → wrong temp file extension (use .webm/.mp4, NOT .input)
# ❌ "Invalid file format" → ffmpeg failed AND raw fallback rejected
# ❌ "No such file or directory: /tmp/tmp....input.wav" → ffmpeg not found (LaunchAgent no PATH)
#    → Diagnose: launchctl getenv PATH → empty = confirmed; fix = absolute FFMPEG_PATH
# ❌ "All connection attempts failed" → Hermes URL wrong (not an HTTP API server)
# ❌ Silent after TTS 200, no "playback ended" → AudioContext not pre-unlocked (Pitfall #7)
# ❌ Multi-turn: first works, rest fail STT → use new-MediaRecorder-per-utterance (Pitfall #6)

# Confirm correct JS version is loaded (service worker cache check):
# Open https://<host>:8765/app.js in Safari — search for a known string from latest code
# If not found → cache not updated → bump service-worker.js CACHE version + ?v=N on script tag
```

## Project Structure

```
luna-voice/
├── server/
│   ├── main.py          # FastAPI WebSocket server
│   └── requirements.txt # fastapi, uvicorn, openai, httpx, python-dotenv
├── client/
│   ├── index.html       # PWA shell — remember to bump ?v=N on app.js script tag
│   ├── app.js           # AudioContext VAD + WebSocket logic
│   ├── styles.css       # Dark UI matching Mission Control
│   ├── manifest.json    # PWA manifest (apple-mobile-web-app-capable)
│   ├── service-worker.js  # CACHE version must be bumped on every client change
│   └── icons/           # luna-192.png, luna-512.png
├── scripts/
│   ├── setup.sh         # deps + tailscale cert + icons + LaunchAgent
│   └── com.aven.luna-voice.plist
├── ssl/                 # cert.pem + key.pem (gitignored)
├── .env                 # OPENAI_API_KEY + ANTHROPIC_API_KEY (gitignored)
└── .env.example
```

## References

- `references/debugging-session-2026-05-26.md` — Full root-cause analysis: SW cache trap, ffmpeg extension bug, initChunk vs new-recorder approach
