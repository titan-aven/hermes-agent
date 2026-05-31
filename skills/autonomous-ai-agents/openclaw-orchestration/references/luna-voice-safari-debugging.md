# Luna Voice — Safari iOS PWA Audio Debugging

## Problem: Whisper STT rejects audio from iPhone Safari

**Symptom:** App listens, transcription never comes back, loops back to listening.  
**Log entry:** `STT error: Invalid file format. Supported formats: ['flac', 'm4a', 'mp3', 'mp4', ...]`

**Root cause:** Safari iOS does not correctly report `MediaRecorder.isTypeSupported()` for any format,
so the client falls through to an empty string mimeType. The server then names the file `audio.` 
(no extension) or an unrecognized format, and Whisper rejects it.

## Fix — Two parts

### 1. Client-side (app.js): Force mp4 fallback for Safari
```javascript
setupRecorder() {
  const candidates = [
    'audio/webm;codecs=opus',
    'audio/webm',
    'audio/mp4',
    'audio/mp4;codecs=mp4a',
    'audio/aac',
    'audio/ogg;codecs=opus',
    '',
  ];
  // Safari iOS fallback: doesn't report support but records in mp4/m4a
  this.mimeType = candidates.find(t => t === '' || MediaRecorder.isTypeSupported(t));
  if (!this.mimeType || this.mimeType === '') {
    this.mimeType = 'audio/mp4';
  }
  const opts = this.mimeType ? { mimeType: this.mimeType } : {};
  this.mediaRecorder = new MediaRecorder(this.stream, opts);
```

### 2. Server-side (main.py): Map all Safari mime types to Whisper extensions
```python
async def transcribe(audio_bytes: bytes, mime_type: str = "audio/mp4") -> str:
    mime_to_ext = {
        "audio/mp4": "mp4",
        "audio/m4a": "m4a",
        "audio/webm": "webm",
        "audio/webm;codecs=opus": "webm",
        "audio/ogg": "ogg",
        "audio/ogg;codecs=opus": "ogg",
        "audio/wav": "wav",
        "audio/mpeg": "mp3",
    }
    mime_base = mime_type.split(";")[0].strip().lower()
    ext = mime_to_ext.get(mime_base, mime_to_ext.get(mime_type.lower(), "m4a"))
    buf = io.BytesIO(audio_bytes)
    buf.name = f"audio.{ext}"
    result = await openai.audio.transcriptions.create(
        model="whisper-1",
        file=buf,
        language="de",
    )
    return result.text
```

## Problem 2: "Hermes error: All connection attempts failed"

**Symptom:** STT works (transcript logged), but no response from Luna.  
**Root cause:** The server tried to call `http://127.0.0.1:8642/v1/chat/completions` — 
Hermes is NOT an HTTP server. It's a CLI tool with no OpenAI-compatible API endpoint.

**Fix:** Call Anthropic API directly with Luna's system prompt:
```python
async def call_hermes(messages: list[dict]) -> str:
    system = (
        "Du bist Luna — eine brillante, warmherzige und empathische KI-Assistentin. "
        "Du bist weiblich, sprichst Deutsch und antwortest direkt, klar und auf Augenhöhe. "
        "Du hilfst Kai Voges bei allem — Projekte, Ideen, Planung, Alltag. "
        "Halte Antworten für gesprochene Dialoge kurz und natürlich (1-3 Sätze)."
    )
    anthropic_key = os.getenv("ANTHROPIC_API_KEY", "")
    async with httpx.AsyncClient(timeout=90.0) as client:
        resp = await client.post(
            "https://api.anthropic.com/v1/messages",
            headers={
                "x-api-key": anthropic_key,
                "anthropic-version": "2023-06-01",
                "content-type": "application/json",
            },
            json={
                "model": "claude-haiku-4-5-20251001",
                "max_tokens": 1024,
                "system": system,
                "messages": messages,
            },
        )
        resp.raise_for_status()
        return resp.json()["content"][0]["text"]
```

Use `claude-haiku-4-5-20251001` for voice (low latency, cheap). ANTHROPIC_API_KEY from `~/.hermes/.env`.

## Deployment notes
- Service: `~/luna-voice-build/` 
- LaunchAgent: `~/Library/LaunchAgents/com.aven.luna-voice.plist`
- Restart: `launchctl unload ~/Library/LaunchAgents/com.aven.luna-voice.plist && launchctl load ~/Library/LaunchAgents/com.aven.luna-voice.plist`
- Logs: `~/luna-voice-build/logs/luna-voice-error.log`
- GitHub: `titan-aven/luna-voice` (private)
- URL: `https://mac-mini-von-aven.tail15c773.ts.net:8765`
- OpenAI key: from `~/.openclaw/openclaw.json` → `messages.tts.providers.openai.apiKey`
