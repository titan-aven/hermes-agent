---
name: macos-audio-capture-transcription
description: "Capture system audio on macOS with BlackHole + transcribe locally with faster-whisper. For meeting bots, lecture recording, podcast transcription."
version: 1.0.0
author: Luna (Hermes Agent)
license: MIT
prerequisites:
  commands: [brew, pip3, python3]
metadata:
  hermes:
    tags: [audio, transcription, whisper, blackhole, macos, meetings]
---

# macOS Audio Capture + Transcription

Use this skill when you need to capture system audio on macOS and transcribe it locally — meeting bots, lecture recording, podcast/call transcription without sending audio to external APIs.

## Core Components

| Component | Purpose | Install |
|-----------|---------|---------|
| **BlackHole 2ch** | Virtual audio driver — routes system audio to a capture device | `brew install --cask blackhole-2ch` |
| **faster-whisper** | Local Whisper transcription — fast, no API key | `pip3 install faster-whisper` |
| **Playwright** | Browser automation (auto-join meetings) | `pip3 install playwright && python3 -m playwright install chromium` |

## Installation (Mac Mini / macOS)

```bash
# 1. BlackHole (requires GUI interaction for System Extension approval)
brew install --cask blackhole-2ch

# 2. faster-whisper
pip3 install faster-whisper

# 3. Playwright for auto-join
pip3 install playwright
python3 -m playwright install chromium
```

⚠️ **BlackHole Note:** After install, macOS shows a System Extension prompt. Must be approved in System Settings → Privacy & Security → Allow. If running headless/remote, Kai must do this manually once.

## Audio Capture Setup (macOS)

**Multi-Output Device** — lets you hear audio AND capture it:
1. Open **Audio MIDI Setup** (Spotlight: "Audio MIDI Setup")
2. Click `+` → "Create Multi-Output Device"
3. Check both: your real speakers/headphones + BlackHole 2ch
4. Set as system output in System Settings → Sound

**Recording with Python + sounddevice:**
```python
import sounddevice as sd
import soundfile as sf

# List devices — find BlackHole
print(sd.query_devices())

# Record from BlackHole (device index varies)
BLACKHOLE_DEVICE = 3  # check with query_devices()
SAMPLE_RATE = 44100
recording = sd.rec(int(duration_seconds * SAMPLE_RATE),
                   samplerate=SAMPLE_RATE,
                   channels=2,
                   device=BLACKHOLE_DEVICE,
                   dtype='float32')
sd.wait()
sf.write('meeting.wav', recording, SAMPLE_RATE)
```

## Transcription with faster-whisper

```python
from faster_whisper import WhisperModel

# Model sizes: tiny, base, small, medium, large-v3
# large-v3 = best quality, ~3GB RAM; base = fast, ~150MB RAM
model = WhisperModel("base", device="cpu", compute_type="int8")

segments, info = model.transcribe("meeting.wav", beam_size=5, language="de")
transcript = "\n".join([segment.text for segment in segments])
print(transcript)
```

**Mac Mini performance:** `base` model = real-time on Apple Silicon. `large-v3` = ~0.3x realtime (3h meeting → ~1h transcription).

## Meeting Auto-Join (Playwright)

```python
from playwright.sync_api import sync_playwright

def join_teams_meeting(join_url: str, display_name: str = "Luna"):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=False)  # headless=True often blocked by Teams
        page = browser.new_page()
        page.goto(join_url)
        # Click "Join on the web" or "Continue on this browser"
        page.click("text=Continue on this browser")
        # Enter display name
        page.fill('[placeholder="Type your name"]', display_name)
        # Mute mic + cam before joining
        page.click('[data-tid="toggle-mute"]')
        page.click('[data-tid="toggle-video"]')
        # Join
        page.click("text=Join now")
        # Keep alive
        page.wait_for_timeout(meeting_duration_ms)
        browser.close()
```

⚠️ Teams detects headless browsers — use `headless=False`. On Mac Mini (no display), use a virtual display or `Xvfb`.

## Kalender-Watcher (Graph API)

Luna's eigener Kalender (`Kai.ai2026@outlook.com`) via Microsoft Graph:

```python
import requests

def get_upcoming_meetings(access_token, lookahead_minutes=60):
    from datetime import datetime, timezone, timedelta
    now = datetime.now(timezone.utc)
    until = now + timedelta(minutes=lookahead_minutes)
    
    url = "https://graph.microsoft.com/v1.0/me/calendarView"
    params = {
        "startDateTime": now.isoformat(),
        "endDateTime": until.isoformat(),
        "$select": "subject,start,end,onlineMeeting,isOnlineMeeting"
    }
    headers = {"Authorization": f"Bearer {access_token}"}
    resp = requests.get(url, headers=headers, params=params)
    return resp.json().get("value", [])
```

## Kai / Luna Setup (Mai 2026)

- **Luna Account:** `Kai.ai2026@outlook.com` — eingeladen als Gast zu Vibracoustic-Meetings
- **Platform:** OpenClaw Mac Mini (Apple Silicon, macOS 26.5)
- **Whisper Model:** `base` für schnelle Transkription, `large-v3` für höchste Qualität
- **Warum dieser Ansatz:** Vibracoustic-IT blockiert Graph API Zugriff auf Firmen-Transcripts — Luna erstellt ihr eigenes Transcript via Audio-Capture

## Pitfalls

- **BlackHole erfordert manuelle GUI-Genehmigung** für System Extension — kann nicht per Script erteilt werden. Einmalig in System Settings → Privacy & Security.
- **Teams erkennt headless Browser** — `headless=False` nutzen, ggf. mit virtuellem Display (`Xvfb` auf Linux, direkt auf macOS mit echtem Display).
- **BlackHole Device-Index variiert** — immer `sd.query_devices()` nutzen um den Index zu ermitteln, nicht hardcoden.
- **Audio-Format:** WAV ist am sichersten für Whisper. MP3 funktioniert auch, aber WAV vermeidet Encoding-Verluste.
- **Meeting-Ende erkennen:** Teams zeigt kein zuverlässiges DOM-Signal. Entweder Kalender-Endzeit + Buffer nutzen, oder periodisch prüfen ob Join-Button wieder erscheint.
