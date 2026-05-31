# Memory Wiki Konzept — Voges AI (2026-05-25)

## Status
Konzipiert, noch nicht gebaut. /goal 2 steht noch aus.

## Entscheidungen (aus Gespräch mit Kai)

### Architektur
- **Speicherort:** Mac Mini (`~/wiki/`)
- **Obsidian-kompatibel** — als Vault öffnen, iPhone-App via iCloud-Sync
- **Web-Oberfläche:** Als 5. Tab im Mission Control Board (`http://100.115.12.83:3737/`)
- **Kein separates Tool** — alles im bestehenden Board einbetten

### Datenschutz & Zugriffsrechte
- **Nur Kais Dialoge** werden geloggt (Discord + Telegram)
- **Ausschluss per Keyword:** `#privat` → nie geloggt
- **Ausschluss per Channel:** bestimmte Channels grundsätzlich wiki-frei (whitelist)
- **Lina & Inga:** Grundsätzlich ausgeschlossen — kein Logging ohne explizite Freigabe
- **Nicht öffentlich:** Nur über Tailscale erreichbar

### Navigation
- **Chronologisch:** Timeline / Daily Log (scrollbar)
- **Thematisch:** Cluster/Tags (Northpeak, Fitness, Finanzen, Technik...)
- **Suche:** Volltext über alle Einträge

### Agent-Integration
- **Zoe (Paperclip):** Business-Entscheidungen fließen mit ein
- **Alle Agents:** Können wiki-relevante Outputs in `raw/agent-outputs/` schreiben
- **Luna:** Hauptpflegerin — Daily Logs, Entities, Cross-References

### Struktur
```
~/wiki/
├── SCHEMA.md
├── index.md
├── log.md
├── raw/conversations/    ← Kais Dialoge (gefiltert)
├── raw/agent-outputs/    ← Agent-Ergebnisse
├── entities/             ← Kai, Familie, Northpeak, Agents...
├── projects/             ← Aktive Projekte
├── concepts/             ← Technik, Entscheidungen
└── agents/               ← Eine Seite pro Agent
```

### Geplante Cron Jobs
| Job | Agent | Zeitplan |
|-----|-------|----------|
| Daily Log | Luna | 22:00 täglich |
| Morning Priority | Luna | 09:00 täglich |
| Weekly Review | Luna + Zoe | Sonntags |

## Nächster Schritt
/goal an Titan: Memory Wiki Tab ins Mission Control Board bauen + Backend-Logik (Daily Log, Keyword-Filter, Discord/Telegram-Source).
