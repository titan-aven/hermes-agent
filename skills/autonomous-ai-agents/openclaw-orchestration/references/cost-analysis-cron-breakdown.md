# OpenClaw Cost Analysis: Cron-Job Breakdown

## Kategorisierungs-Logik (Mai 2026)

Alle Sessions mit `[cron:...]` oder `[CRON]` im ersten User-Message-Text
nach Typ kategorisieren:

```python
def categorize(session):
    msg = session["first_user_msg"]
    if "Prüfe ob der Google OAuth Token" in msg: return "cron_token_check"
    if "Prüfe alle Cron-Jobs auf Fehler" in msg: return "cron_error_check"
    if "Nightly Backup" in msg: return "cron_backup"
    if "Morning Briefing" in msg or "PHAI Morning" in msg: return "cron_morning_briefing"
    if "Evening Scan" in msg or "PHAI Evening" in msg: return "cron_evening_scan"
    if "wöchentlichen" in msg or "System-Review" in msg or "Agent Review" in msg: return "cron_weekly_review"
    if "[cron:" in msg or "[CRON]" in msg: return "cron_other"
    if "Subagent Context" in msg: return "subagent"
    if not msg.strip(): return "delivery_mirror"
    # Inter-session messages / high exec-count = autonomous coding sessions
    if "Inter-session" in msg or tools.get("exec", 0) > 20: return "autonomous_coding"
    return "direct_conversation"
```

## Mai 2026 Ist-Kosten — main Agent (verfeinerte Kategorisierung)

| Kategorie | Sessions | Kosten | Haupttreiber |
|-----------|----------|--------|--------------|
| **autonomous_coding** | 5 | **$22.22** | 200+ exec-Calls, Project-Mgmt/Infra-Entwicklung |
| phai_evening | 15 | $11.33 | Opus bis 15.5 → Sonnet |
| subagent | 13 | $6.01 | Delegierte Tasks |
| phai_morning | 16 | $5.79 | Opus bis 15.5 → Sonnet |
| weekly_review | 8 | $3.55 | Sonnet |
| direct_conversation | 7 | $3.51 | Echte Gespräche (kurz!) |
| cron_backup | 24 | $2.66 | Haiku/Opus |
| cron_error_check | 28 | $0.75 | Haiku/Gemini |
| cron_token_check | 25 | $0.24 | Gemini Flash ✅ |
| **TOTAL main** | **145** | **$56.06** | |

**Wichtige Erkenntnis:** Direkte Gespräche kosten nur $3.51/Monat — sie sind NICHT das Problem.
Das Problem ist `autonomous_coding` ($22) = Aven entwickelt aktiv Infrastruktur (Mission Control, OpenClaw-Setup).

## Mai 2026 Ist-Kosten — alle Agents zusammen

| Kategorie | Sessions | Kosten | Modelle |
|-----------|----------|--------|---------| 
| direct_conversation | 30 | $31.13 | Opus/Sonnet/Haiku |
| cron_other | 193 | $13.99 | Sonnet/Haiku/Gemini |
| cron_evening_scan | 15 | $11.33 | Opus (bis 15.5) → Sonnet |
| subagent | 13 | $6.01 | Opus/Sonnet/Haiku |
| cron_morning_briefing | 16 | $5.79 | Opus (bis 15.5) → Sonnet |
| cron_weekly_review | 8 | $3.55 | Sonnet |
| cron_backup | 24 | $2.66 | Opus/Haiku |
| cron_error_check | 28 | $0.75 | Haiku/Gemini |
| cron_token_check | 25 | $0.24 | Gemini Flash ✅ |
| **TOTAL** | **357** | **$75.45** | |

## Vollständige Cron-Job-Liste (jobs.json, Mai 2026)

| Job-Name | Agent | Modell Wochentag | Modell Weekend | Status |
|----------|-------|-----------------|----------------|--------|
| hera-morning-brief-weekday | hera | Sonnet | — | ✅ ok |
| hera-morning-brief-weekend | hera | — | Sonnet | ✅ ok |
| vera-morning-research | vera | Haiku | — | ✅ optimal |
| vera-morning-research-weekend | vera | — | Haiku | ✅ optimal |
| atlas-morning-finance | atlas | Sonnet | — | ✅ ok |
| atlas-morning-finance-weekend | atlas | — | ~~Gemini Pro~~ → **Sonnet** | ✅ gefixt 2026-05-25 |
| nova-morning-health | nova | Haiku | — | ✅ optimal |
| nova-morning-health-weekend | nova | — | Haiku | ✅ optimal |
| nightly-backup | main | Haiku | — | ✅ optimal |
| gog-token-check-daily | main | Gemini Flash | — | ✅ optimal |
| hera-booking-check | hera | Gemini Flash | — | ✅ optimal |
| nft-morning-weekday | atlas | Haiku | — | ✅ optimal |
| nft-morning-weekend | atlas | — | Haiku | ✅ optimal |
| phai-evening-scan | main | Sonnet | — | ⚠️ Haiku testen? |
| phai-morning-briefing | main | Sonnet | — | ⚠️ Haiku testen? |
| Mission Control Kosten-Log | titan | Sonnet | — | ⚠️ Haiku testen |
| weekly-system-review | main | Sonnet | — | ok |
| monthly-model-check | main | Sonnet | — | ok (1x/Monat) |

**Bereits gefixt:** `atlas-morning-finance-weekend` war auf `google/gemini-2.5-pro` (teurer als Sonnet) — auf Sonnet umgestellt am 2026-05-25.

## Optimierungs-Simulationen (geschätzte Einsparungen)

| Kategorie | Ist | Einsparung | Maßnahme |
|-----------|-----|------------|----------|
| autonomous_coding | $22.22 | ~$10 | Kürzere/kompaktere Prompts, Haiku für Zwischen-Steps |
| cron_evening_scan | $11.33 | ~$8 | Haiku testen |
| cron_other | $13.99 | ~$10 | Haiku/Gemini Flash |
| cron_backup/titan-log | $2.66+ | ~$2.50 | Haiku only |
| subagent | $6.01 | ~$3 | Haiku für mechanische Tasks |

**Einsparungspotenzial gesamt: ~$33/Monat** auf OpenClaw-Seite allein.

## Evening Scan Prompt-Struktur (PHAI/Vibracoustic)

6 Schritte, ~28 Tool Calls pro Abend:
1. Google Drive scannen (Ordner: `Firma/Project House AI/Projects`, Parent: `1TODomNrm--Qu9sBJvlRlIPZYWtlcnv58`)
2. Transkripte auswerten → Action Items / erledigte Tasks / Entscheidungen
3. Google Sheet pflegen (ID: `1lEBu3JDWmt9WrHRigHbaJNXzPCvIrjImWy5dGJY21rY`, Tab: `Action Items`)
4. Azure DevOps prüfen (Org: `vc-projecthouse-ai`, Projekt: `Projecthouse AI`)
5. GitHub Issues prüfen (Repo: `vibracoustic-global/projecthouse-ai.planning`)
6. Zusammenfassung → `memory/phai-scan-latest.md` (Input für Morning Briefing)

**Modell-Empfehlung:** Haiku testen. Das ist sequentielles Tool-Calling,
kein kreatives Denken. Opus war hier völlig überdimensioniert.

## Modellwechsel-Timeline (main agent)

- **1.–15. Mai:** Morning Briefing + Evening Scan auf **Opus** (~$0.57/Session)
- **18. Mai+:** Aven hat selbst auf **Sonnet** umgestellt (~$0.38/Session)
- **Nächster Schritt:** Haiku testen (~$0.05/Session geschätzt)

## Titan-Kosten Klarstellung

Titan ($50.52 gesamt) schaut schlimmer aus als es ist:
- **April:** $28.86 + $14.37 = $43 aus **zwei einzelnen langen Entwicklungs-Sessions** (Mission Control Board Aufbau)
- **Mai laufend:** Nur $5.31 — täglicher Kosten-Log-Cron ($0.07/Tag auf Sonnet) + 3 neue Sessions
- Der tägliche Kosten-Log-Cron läuft auf Sonnet, könnte auf Haiku → ~$0.002/Tag statt $0.07/Tag

## SSH-Zugang Mac Mini

SSH-Keys existieren nicht in `~/.ssh/` (nur `agent` und `known_hosts`).
Für direkte SSH-Befehle auf den Mac Mini: keys erst einrichten oder
OpenClaw-CLI als Brücke verwenden statt SSH.

## Anthropic Billing-Zugang

Anthropic Console (console.anthropic.com) erfordert Google-Login mit `kai.voges@googlemail.com` + Passwort + 2FA.
Luna kann den Login-Flow nicht selbst durchführen — Kai muss sich einloggen und Screenshot der Billing/Usage-Seite teilen.
Die Anthropic API (`/v1/usage`, `/v1/organizations/costs`) gibt keine Billing-Daten zurück — nur über die Console erreichbar.

**Kai's Anthropic-Account:** `kai.voges@googlemail.com` (persönlich, nicht aven73.claw@gmail.com)

## Gesamtkosten-Einordnung Mai 2026

Rechnung: ~579€ gesamt.
Aufschlüsselung soweit rekonstruierbar:
- OpenClaw alle Agents: ~$90 (hauptsächlich main $56 + titan $5 Mai-laufend)
- Luna (Hermes): unbekannt — erst seit 24. Mai aktiv, $26 in 2 Tagen (hohe Cache-Token-Kosten)
- Differenz zu 579€: vermutlich weitere Anthropic-API-Calls außerhalb OpenClaw/Luna

**Cache-Token-Erkenntnis:** Bei Luna/Hermes sind Cache-Write ($3.75/M) und Cache-Read ($0.30/M) die dominanten Kostentreiber — nicht Output-Tokens. Das System-Prompt (Memory + Skills) wird bei jeder Nachricht als Cache geschrieben/gelesen. 38M Cache-Read-Tokens in 2 Tagen = $11.58 allein für Cache-Reads.
