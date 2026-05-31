# Titan Delegation Pattern — Long Build Tasks

## Root Cause (luna-voice Session, 2026-05-25)

Titan baute luna-voice PWA erfolgreich (08:25–08:30 Uhr), wurde aber durch 60s-CLI-Timeout gekillt.
Error in Session: `"This operation was aborted"` — HTTP-Verbindung gekappt durch meinen Timeout.
Claude Code hat danach von vorne neu gebaut (11:05–11:15 Uhr) und fertiggestellt.

## Korrekte Vorgehensweise

### 1. Nachricht senden mit korrekten Flags
```bash
openclaw agent --agent titan \
  --session-key "agent:titan:discord:channel:1487563493526732860" \
  --message "Bauauftrag: ..." \
  --deliver \
  --reply-channel discord \
  --reply-to "channel:1487563493526732860" \
  --reply-account titan \
  --thinking off \
  --timeout 600
```

### 2. Titan soll sofort bestätigen + entkoppeln
Titan antwortet in <5 Sekunden:
> "Auftrag empfangen, starte Build als Hintergrundprozess."

Dann startet Titan `nohup`:
```bash
source ~/.hermes/.env
nohup bash -c "
  ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY claude -p \"$(cat /tmp/build-task.md)\" \
    --dangerously-skip-permissions \
    --max-turns 80 \
    --output-format json > /tmp/build-result.json 2>/tmp/build-log.txt
" > /tmp/build-wrapper.log 2>&1 &
```

### 3. Titan meldet Ergebnis selbst
```bash
openclaw message send --channel discord \
  --target "1487563493526732860" \
  --account titan \
  -m "✅ Build fertig! URL: https://..."
```

### 4. Verifikation durch Luna
```bash
# Session-Datei wächst?
ls -lt ~/.openclaw/agents/titan/sessions/*.jsonl | head -3

# Mission Control Board: Titan grün?
# Kosten-Tab: Output Tokens steigen?
```

## BUILD_JOBS.md
Liegt in `~/.openclaw/workspace-titan/BUILD_JOBS.md` — Titan liest das beim Session-Start.

## Fallback: Claude Code direkt
Wenn Titan-Delegation scheitert → Claude Code direkt:
```bash
source ~/.hermes/.env
# Im Terminal (background=true, notify_on_complete=true)
ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY claude -p "$(cat ~/task.md)" \
  --dangerously-skip-permissions \
  --max-turns 80 \
  --output-format json > /tmp/result.json 2>/tmp/log.txt
```
Nachteil: Titan nicht involviert, Code landet in Luna's Kontext statt Titans Workspace.
