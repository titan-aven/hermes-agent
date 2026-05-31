# Debugging Session — 2026-05-26

## Symptom
Luna Voice PWA: first utterance transcribed + responded correctly. All subsequent utterances: "Nachdenken..." loop, no STT, no response, no audio.

## Root Causes Found (in order of discovery)

### 1. Service Worker Cache — Fixes Never Loaded
**Impact:** Every code fix deployed over ~2 hours was silently ignored. iPhone was serving `app.js` from `luna-v1` cache throughout the entire session.
**Evidence:** User reported "exactly zero change" across many fixes.
**Fix:** Bump `CACHE = 'luna-v1'` → `'luna-v6'` in service-worker.js + add `?v=6` to `<script src="/app.js">` in index.html.
**Lesson:** Always bump SW cache version AND add query param on every client change. If user says nothing changed, confirm which JS version is running before doing anything else.

### 2. ffmpeg temp file wrong extension — rc=183
**Error log:**
```
ffmpeg failed (rc=183): Invalid data found when processing input
Error opening input file /tmp/tmp5o36o5x6.input.
```
**Cause:** `tempfile.NamedTemporaryFile(suffix=".input")` — ffmpeg uses the file extension for format detection. `.input` is unknown, so ffmpeg returns rc=183.
**Fix:** Map MIME type → correct extension (.webm, .mp4 etc.) and use that as the suffix.

### 3. WebM initChunk approach — unreliable in practice
**Attempted approach:** Save first `ondataavailable` chunk as `initChunk`, prepend to every subsequent utterance's Blob.
**Problem:** Even with initChunk prepended, ffmpeg still returned rc=183 on most utterances. The chunk boundary doesn't reliably correspond to the EBML header end; timing and Safari quirks make the prepend approach fragile.
**Final fix:** Create a new `MediaRecorder` for each utterance. `.start()` on speech detection, `.stop()` on silence. Every resulting Blob automatically contains complete WebM headers. Robust and simple.

## Timeline of Fixes
1. ffmpeg absolute path (LaunchAgent no PATH) → fixed STT for first utterance
2. ffmpeg stderr logging → revealed rc=183 error
3. TTS format mp3→wav → correct for iOS AudioContext
4. AudioContext pre-unlock + HTMLAudioElement pre-unlock → correct approach
5. initChunk approach → attempted, failed due to rc=183
6. **SW cache bump** → all prior fixes now actually loaded
7. **ffmpeg correct extension** (.webm not .input) → STT works consistently
8. **New MediaRecorder per utterance** → multi-turn works reliably

## Key Insight
When debugging a PWA and "nothing changes" despite server-side fixes, **check the Service Worker cache first** before writing any more code. Open `/app.js` directly in Safari, search for a known-new string. If not found, the phone is running old code.
