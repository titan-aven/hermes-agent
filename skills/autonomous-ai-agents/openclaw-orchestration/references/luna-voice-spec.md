# luna-voice — Hands-Free Voice Interface PWA Spec

**Status:** ✅ LIVE (built via Claude Code, 2026-05-25)
**GitHub:** `titan-aven/luna-voice` (private)
**Service URL:** `https://mac-mini-von-aven.tail15c773.ts.net:8765`
**Local dir:** `~/luna-voice-build/`
**LaunchAgent:** `~/Library/LaunchAgents/com.aven.luna-voice.plist`

## Concept

A Progressive Web App installable on Kai's iPhone via Safari "Add to Homescreen".
Enables hands-free voice dialog with Luna (Hermes Agent) — like ChatGPT Advanced Voice Mode.

## Dialog Loop

```
[One tap to activate]
  → VAD detects speech end
  → OpenAI Whisper API transcribes (DE primary, EN fallback)
  → POST to local Hermes API (Luna)
  → OpenAI TTS speaks response
  → VAD listens again
  → "Tschüss" / "Stop" / second tap ends session
```

## Technical Stack

- **Backend:** Python or Node (server/)
- **STT:** OpenAI Whisper API
- **LLM:** Local Hermes Agent Luna — check `~/.hermes/` config for API port
- **TTS:** OpenAI TTS (or Hermes-configured TTS)
- **HTTPS:** `tailscale cert <hostname>` for Safari microphone access
- **Autostart:** LaunchAgent on Mac Mini

## PWA Requirements

- `manifest.json` with Luna wing icon, display: standalone
- `service-worker.js` for offline shell
- Fullscreen, no browser chrome
- Only accessible via Tailscale (mac-mini-von-aven.tail15c773.ts.net)

## UI Design

- Dark theme matching Mission Control Board (dark blue/black, white text, blue accents)
- Single large activation button center screen
- Waveform animation while listening
- Status indicator: Idle / Lauscht / Denkt / Spricht

## Repo Structure

```
luna-voice/
├── README.md          → One-command setup
├── server/            → Backend (Whisper + Hermes bridge)
├── client/            → PWA (HTML/CSS/JS, manifest.json, service-worker.js)
├── scripts/           → LaunchAgent .plist + setup.sh
└── .env.example
```

## Dispatching to Titan — Working Command Pattern

```bash
# Fresh session key avoids stale thinking-block errors
openclaw agent --agent titan \
  --session-key "agent:titan:luna-voice-$(date +%s)" \
  --message "..." \
  --deliver \
  --reply-channel discord \
  --reply-to "channel:1487563493526732860" \
  --reply-account titan \
  --thinking off
```

Key: `--thinking off` prevents "Invalid signature in thinking block" errors.
Key: fresh `--session-key` avoids collision with stale sessions.

## Next: Memory Wiki Tab

Second `/goal` after luna-voice is complete:
- 5th tab in Mission Control Board (http://100.115.12.83:3737/)
- Titan builds it into the existing board
- Sources: Kai's Discord + Telegram messages only
- Exclusion: `#privat` keyword or channel-based opt-out
- Zoe/Paperclip decisions also feed in
- Dual navigation: chronological (timeline) + thematic (topic clusters)
- Not publicly accessible — Tailscale only
