# OpenClaw Update History

## 2026-05-24: 2026.5.19 → 2026.5.22
- Backup erstellt: `.openclaw_backup_20260524_214249`
- Node v24.15.0 war bereits aktuell — kein Upgrade nötig
- `openclaw update --yes` hat Gateway automatisch neu gestartet
- Doctor: 2 stale Google Session-Routing-States bereinigt
- Discord: 7/7 Accounts OK, Telegram: 7/7 Accounts OK
- Routine basiert auf Nolans Empfehlung (Kais ChatGPT-Assistent)

## Weekend-Briefing Error (bekannt)
- hera-morning-brief-weekend zeigt `error` wegen fehlendem `/tmp/nova_morning_input.txt`
- Nova-Cron liefert nur Mo-Fr — am Wochenende fehlt die Datei
- Workaround: `2>/dev/null || echo ''` bereits im Hera-Cron eingebaut, aber noch nicht überall

---

# Paperclip Update History

## 2026-05-24: Update 2026.428.0 → 2026.517.0 FEHLGESCHLAGEN → Rollback
- Backup erstellt: `.paperclip_backup_20260524_233603`
- Update via `npm update -g paperclipai` erfolgreich installiert
- **Bug in 2026.517.0:** Service startet nicht — sucht `/opt/homebrew/packages/shared/package.json` (existiert nicht bei globaler npm-Installation)
- Zwischenfehler: `Directory already exists: .../run` — durch `rm -rf ~/.paperclip/instances/default/run` behoben
- Danach zweiter Fehler: `Package package.json not found at /opt/homebrew/packages/shared/package.json`
- **Rollback auf 2026.428.0** via `npm install -g paperclipai@2026.428.0`
- Nach Rollback: PID 25341, Service läuft normal
- Empfehlung: Auf nächstes Bugfix-Release warten
