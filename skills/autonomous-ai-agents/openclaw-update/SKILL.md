---
name: openclaw-update
description: "OpenClaw vorsichtige Update-Routine: Backup, Gateway stoppen, update, doctor, restart, verify."
tags: [openclaw, update, maintenance, gateway]
triggers:
  - user asks to update OpenClaw
  - user mentions OpenClaw update or upgrade
  - OpenClaw version is outdated
---

# OpenClaw Update-Routine

Vorsichtige Update-Routine für OpenClaw auf dem Mac Mini.

## Wichtige Hinweise

- **OpenClaw läuft lokal auf dem MacBook** — nicht auf dem Mac Mini. Das `openclaw`-Binary ist unter `/opt/homebrew/bin/openclaw`, der Gateway-LaunchAgent (`ai.openclaw.gateway`) läuft lokal. SSH zum Mac Mini ist für das Update **nicht nötig**.
- **Kein** `openclaw aven ...` verwenden — der Aven-Command existiert nicht mehr
- Agent main läuft innerhalb des Gateways (nicht als separater LaunchAgent)
- Aktiver LaunchAgent: `ai.openclaw.gateway`
- Falls `ai.openclaw.aven` auftaucht: veralteter Rest, entfernen nicht reparieren
- `openclaw -m "Hi"` funktioniert nicht — stattdessen `openclaw agent --agent main --message "Hi"`
- `openclaw update --yes` startet den Gateway automatisch neu — kein separater `openclaw gateway restart` nötig
- Nach Update: CLI-Version und Gateway-Version müssen übereinstimmen (prüfen mit `openclaw gateway status`)
- Nolans Update-Routine ist die empfohlene Referenz: Backup → Gateway stop → Node prüfen → brew update (kein upgrade) → openclaw update → doctor → verify

## Diagnose Before Acting — Critical Rule

**Kai's explicit correction (2026-05-26):** "Du arbeitest zu oberflächlich. Ich halte es für unwahrscheinlich, dass die zweimal den gleichen Fehler machen."

Before rolling back, reinstalling, or blaming a package:
1. **Isolate the variable** — test the binary directly: `paperclipai run` (not via LaunchAgent)
2. **Read the error carefully** — "same error twice across two releases" is a strong signal the environment is wrong, not the package
3. **Identify the exact call path** — what is actually being executed? (`launchctl list`, `cat plist`, trace the binary)
4. **Only then act** — with a specific hypothesis, not a shotgun rollback

Skipping diagnosis to "do something" wastes money (multiple failed attempts) and forces Kai to correct you. The cost of one wrong rollback exceeds the cost of 60 seconds of analysis.



## Versionscheck (vor dem Update)

Zuerst prüfen ob überhaupt ein Update nötig ist — alles lokal auf dem MacBook:

```bash
openclaw --version                  # installierte Version
npm show openclaw version           # neueste verfügbare Version
paperclipai --version               # installierte Version
npm show paperclipai version        # neueste verfügbare Version
```

Wenn installiert = verfügbar → kein Update nötig. Nur wenn Versionen abweichen weiter unten fortfahren.

> ⚠️ SSH zum Mac Mini ist **nicht nötig** — OpenClaw und Paperclip laufen beide auf dem MacBook (lokale Installation via Homebrew/npm).

---

### 1. Backup erstellen
```bash
cp -r ~/.openclaw ~/.openclaw_backup_$(date +%Y%m%d_%H%M%S)
ls -la ~/ | grep openclaw  # Backup bestätigen
```

### 2. Gateway sauber stoppen
```bash
openclaw gateway stop
```

### 3. Node-Version prüfen
```bash
node -v
# Sollte mindestens v24 sein. Wenn v24.x vorhanden: kein brew upgrade!
```

### 4. Homebrew update (nur update, KEIN upgrade)
```bash
brew update
# NICHT: brew upgrade
```

### 5. OpenClaw aktualisieren
```bash
openclaw update --yes
# Gibt Before/After Version aus, updated Plugins, startet Gateway neu
```

### 6. Doctor / Reparaturcheck
```bash
openclaw doctor --fix
# Bereinigt stale Sessions, prüft Plugins, zeigt Warnungen
```

### 7. Gateway-Status prüfen
```bash
openclaw gateway status
# CLI version und Gateway version sollten übereinstimmen
```

### 8. Deep Status-Check
```bash
openclaw status --deep
# Zeigt Channels (Discord/Telegram), Sessions, Health
```

### 9. Funktionstest
```bash
openclaw agent --agent main --message "Hi"
# Agent sollte antworten
```

> ⚠️ **NICHT** `openclaw -m "Hi"` verwenden — das ist kein gültiger Command und gibt einen Fehler zurück (`Unknown command: openclaw Hi`). Immer `openclaw agent --agent main --message "..."` nutzen.

## Verifikation nach Update

Checkliste:
- [ ] Gateway version = CLI version
- [ ] Discord: OK (alle Accounts)
- [ ] Telegram: OK (alle Accounts)
- [ ] Agent main antwortet auf Test-Message

## Troubleshooting

```bash
openclaw status
openclaw logs --follow
launchctl list | grep -i openclaw
```

## Referenzen

- `references/update-history.md` — Versionshistorie + bekannte Fehler

## Paperclip parallel updaten?

Paperclip läuft ebenfalls auf dem Mac Mini (`com.paperclipai.server`, Port 3100).
Nach einem OpenClaw-Update immer prüfen ob auch Paperclip ein Update braucht:

```bash
paperclipai --version          # installierte Version
npm show paperclipai version   # neueste verfügbare Version
```

Wenn Update verfügbar — analog zu OpenClaw:
```bash
# 1. Backup
cp -r ~/.paperclip ~/.paperclip_backup_$(date +%Y%m%d_%H%M%S)
# 2. Stop
launchctl unload ~/Library/LaunchAgents/com.paperclipai.server.plist
# 3. Update
npm update -g paperclipai
# 4. Doctor
paperclipai doctor
# 5. Start
launchctl load ~/Library/LaunchAgents/com.paperclipai.server.plist
# 6. Verify — PID muss vorhanden sein (nicht nur "-")
launchctl list | grep paperclip
# Erwartung: "12345  0  com.paperclipai.server" (erste Spalte = PID, nicht "-")
# Falls "-": Logs prüfen (Schritt 7)
# 7. Bei Fehler: Logs prüfen
tail -20 ~/.paperclip/instances/default/logs/launchd-stderr.log
```

### Bekannte Paperclip-Update-Fehler

**Fehler: `Directory already exists: .../run`**
```
Error: Directory already exists: /Users/aven/.paperclip/instances/default/run
```
→ Stale `/run` Verzeichnis vom letzten Start. Fix:
```bash
rm -rf ~/.paperclip/instances/default/run
launchctl unload ~/Library/LaunchAgents/com.paperclipai.server.plist
sleep 2
launchctl load ~/Library/LaunchAgents/com.paperclipai.server.plist
### Bekannte Paperclip-Update-Fehler

**Fehler: `Package package.json not found at /opt/homebrew/packages/shared/`**
```
Error: Package package.json not found at /opt/homebrew/packages/shared/package.json
```
→ **NICHT ein Paperclip-Bug!** Ursache: LaunchAgent rief `node dist/index.js run` direkt auf statt die `paperclipai` Binary. Dadurch greift `import.meta.url === process.argv[1]` und ruft `runCli()` (Plugin-Scaffolding) statt `main()` auf.
→ **Diagnose-Test ZUERST:** `paperclipai run` manuell ausführen. Wenn das funktioniert → LaunchAgent ist das Problem, nicht die Software. Nicht sofort Rollback!
→ **Fix: LaunchAgent auf `paperclipai run` umstellen:**
```xml
<array>
    <string>/opt/homebrew/bin/paperclipai</string>
    <string>run</string>
</array>
```
Statt: `node /opt/homebrew/lib/node_modules/paperclipai/dist/index.js run`
→ **Rollback auf letzte funktionierende Version (nur falls nötig, wenn auch Binary-Aufruf scheitert):**
→ **Rollback auf letzte funktionierende Version (nur falls nötig):**
```bash
npm install -g paperclipai@<letzte-version>
# z.B.: npm install -g paperclipai@2026.428.0
launchctl unload ~/Library/LaunchAgents/com.paperclipai.server.plist
sleep 2
launchctl load ~/Library/LaunchAgents/com.paperclipai.server.plist
```
→ Auf nächstes Bugfix-Release warten bevor erneut upgraden.

## Pitfalls

1. **`openclaw -m "Hi"` funktioniert nicht** — nur `openclaw agent --agent main --message "Hi"`. Der `-m` Flag existiert nicht; OpenClaw kennt kein direktes Chat-Flag auf dem Root-Command.
2. **Gateway startet automatisch neu** beim `openclaw update --yes` — kein manueller Restart nötig
3. **Doctor-Warnungen** über `groupPolicy=allowlist` sind pre-existing und unkritisch
4. **Node nicht upgraden** wenn bereits v24.x — vermeidet unnötige Risiken
5. **`openclaw update --yes` übernimmt alles** — updated Plugins, führt doctor aus, und restartet Gateway in einem Schritt. Kein separater `openclaw gateway restart` nötig.
6. **Stale Google session routing state** wird von `doctor --fix` automatisch bereinigt — kein manuelles Eingreifen nötig
7. **Kein SSH zum Mac Mini nötig** — `openclaw` und `paperclipai` sind lokale Binaries auf dem MacBook (`/opt/homebrew/bin/`). SSH-Verbindungsversuche scheitern wegen zu vieler Identitäten im Agent und sind unnötig.
8. **`openclaw status` hängt** wenn der Gateway nicht läuft oder keine Verbindung besteht — stattdessen `openclaw --version` für die reine CLI-Version und `openclaw gateway status` für den Gateway-Zustand verwenden.
