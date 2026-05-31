# Luna STT Key Setup — Full Reference

## Problem
Luna (Hermes gateway) and Luna Voice PWA both use OpenAI Whisper for STT. If the key is
missing, expired, or revoked, voice messages produce garbled transcriptions
(e.g. „Fermat Marina" instead of actual speech). Error signature:

```
API error: Error code: 401 - {'error': {'message': 'Incorrect API key provided: sk-pro..*oqcA.'}}
```

## Architecture

```
Discord/Telegram voice msg
       ↓
Hermes gateway (STT)
  ~/.hermes/.env → VOICE_TOOLS_OPENAI_KEY
  ~/.hermes/config.yaml → stt.provider: openai

iPhone Safari
       ↓
Luna Voice PWA (port 8765)
  ~/Library/LaunchAgents/com.aven.luna-voice.plist
  EnvironmentVariables.OPENAI_API_KEY
  Server: ~/luna-voice-build/server/main.py
```

## Fix: Hermes Gateway STT

1. Verify what provider is configured:
   ```bash
   grep -A 5 'stt:' ~/.hermes/config.yaml
   ```
   Must be `provider: openai` (NOT `local` — faster-whisper not installed).

2. Insert/update the key:
   ```bash
   python3 -c "
   import re
   with open('/Users/aven/.hermes/.env') as f:
       content = f.read()
   new_content = re.sub(r'(?m)^#?VOICE_TOOLS_OPENAI_KEY=.*$',
       'VOICE_TOOLS_OPENAI_KEY=NEW_KEY_HERE', content)
   with open('/Users/aven/.hermes/.env', 'w') as f:
       f.write(new_content)
   "
   ```

3. Set provider to openai if not already:
   ```bash
   sed -i '' 's/  provider: local/  provider: openai/' ~/.hermes/config.yaml
   ```

4. Restart gateway:
   ```bash
   hermes gateway restart
   ```

## Fix: Luna Voice PWA

The server reads `OPENAI_API_KEY` from the LaunchAgent environment (not a .env file).

1. Update the plist:
   ```python
   import plistlib, shutil
   path = '/Users/aven/Library/LaunchAgents/com.aven.luna-voice.plist'
   shutil.copy(path, path + '.bak')
   with open(path, 'rb') as f:
       d = plistlib.load(f)
   d.setdefault('EnvironmentVariables', {})['OPENAI_API_KEY'] = 'NEW_KEY_HERE'
   with open(path, 'wb') as f:
       plistlib.dump(d, f)
   ```

2. Restart the service:
   ```bash
   launchctl kickstart -k gui/$(id -u)/com.aven.luna-voice
   ```

3. Verify server is running:
   ```bash
   ps aux | grep luna-voice | grep -v grep
   tail -10 ~/luna-voice-build/logs/luna-voice.log
   ```

## Key Sources

- **Kai provides the key directly** — ask Kai to create a new key at https://platform.openai.com/api-keys
- **Do NOT copy from Aven's OpenClaw config** (`~/.openclaw/openclaw.json` → `skills.entries.openai-whisper-api.apiKey`) — this key may be expired/revoked, and Aven will refuse to hand it over (Security Red Line)
- **Groq alternative (free):** If Kai has a Groq account, `gsk_...` key from https://console.groq.com/keys. Set `GROQ_API_KEY` in `~/.hermes/.env` and `stt.provider: groq` in config.yaml

## Why `provider: local` Fails

`faster-whisper` is not installed on the Mac Mini. When provider is `local` and the package
is missing, Hermes produces nonsense transcriptions instead of failing cleanly.
Fix: `provider: openai` or `provider: groq` (never `local` until faster-whisper is installed).

## Verified Working Config (May 2026)

```yaml
stt:
  enabled: true
  provider: openai
  local:
    model: base
    language: ''
  openai:
    model: whisper-1
```
