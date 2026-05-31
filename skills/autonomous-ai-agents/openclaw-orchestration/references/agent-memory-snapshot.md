# Aven (OpenClaw main agent) Memory Snapshot — 2026-05-24

Condensed from `~/.openclaw/memory/main.sqlite` at Hermes onboarding.

## User Profile
- **Kai Voges**: Senior Controlling Leader @ Vibracoustic, global team
- Building AI Project House (Controlling, Sales, SCM, HR, Accounting)
- Pragmatist: execution > discussion, results > theory
- Prefers German, direct communication
- Family: Inga, Lina (both on Discord with bot access)

## Vibracoustic / AI Project House
- **2026 Targets** (file: config/kai_targets_2026.md):
  - T1 (20%) AI Project House
  - T2 (30%) MVP agentic RFQ — **critical**: 8 Agents MVP-ready in 2026
  - T3 (20%) Copilot Adoption
  - T4 (15%) Pipeline 2027
  - T5 (15%) KPI Tracking
- Kai leads CO Chatbot + Sales Forecast Agent
- Google Drive "Project House AI" folder: 1QUCmPM7m9q6rGKGjyxIFGyiC79zCVIxf

## Ferienwohnung (Holiday Apartment) — Harz
- iCal feeds: Travanto (primary, has guest names), Airbnb, Booking.com (no guest names)
- Booking confirmation template: workspace-hera/config/buchungsbestaetigung_vorlage.txt
- Kurtaxe: 2.50€ × guests × nights
- Google Sheets booking data 2026: 1m2QnJU806nmzFR61ktkjIfYtOCQo87SUHY5NsOFxfng
- FeWo Google Drive folder: 16YyZ4mZLKrs_cq_wIfuTcg-1q0SIMoLb
- Hera handles booking confirmations, calendar management, cleaning coordination

## Google Calendar IDs
- Kai personal: kai.voges@googlemail.com
- Family: family07448130255997084405@group.calendar.google.com
- Ferienwohnung: f810f63ca576432d45a405a4cdd98870cd9cfa0428e181c187f9330ef6f16c8c@group.calendar.google.com
- Waste calendar Sprötze: 5cb09815a92d2c6c31d9b65ddc294e3f23636a31a94237c590dc91b4c1614180@group.calendar.google.com

## Recovery / Disaster Recovery
- Nolan = Kai's ChatGPT assistant for fixing Aven if broken
- Recovery prompts posted to Discord #recovery-prompts (1489124428821430323)
- Nightly backup: 23:00 Europe/Berlin → Google Drive → Synology NAS
- Disaster Recovery Guide: Google Drive → Aven/DISASTER_RECOVERY.md
- Google Drive backup folder: Aven/backups/ (1ruvhyblRXQM_sCX2tgAzBqtS1zPcjLJ3)

## Key Lessons from Aven's Memory
- Previous setup burned ~€500 because Aven never delegated → strict delegation rules now
- `openclaw gateway stop` deinstalls LaunchAgent — NEVER in active session
- Gemini Flash needs instructions at TOP of AGENTS.md, very direct, with ⚠️❌ markers
- Gemini Flash can't reliably cross-channel post → use `openclaw message send`
- Vision models refuse to read bot tokens from photos (security policy)
- After gateway restart, cache-hit drops to 0% → expensive until cache rebuilds
