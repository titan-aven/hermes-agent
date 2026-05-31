# Agent Onboarding Interview — Block Structure

When Kai asks to run an "interview" to update agent context, use this block structure.
Before asking, always check existing MEMORY.md files first to avoid re-asking known facts.

## Pre-Interview Checklist
```bash
# Read these before starting — avoid duplicate questions
cat ~/.openclaw/workspace/MEMORY.md
cat ~/.openclaw/workspace/shared/user_profile.md
cat ~/.openclaw/workspace-nova/MEMORY.md
cat ~/.openclaw/workspace-atlas/MEMORY.md
cat ~/.openclaw/workspace-hera/MEMORY.md
```

## Interview Blocks

### Block 1 — Familie & Alltag (Hera, Lexi, Nova)
- Kinder: Geburtsdaten, Schulen, Klassen
- Feste Termine/Routinen
- Morgenbriefing-Struktur → erst bestehende Cron-Payloads prüfen!

### Block 2 — Gesundheit (Nova)
- Körperdaten (Größe, Gewicht, Ziele)
- Verletzungen/Einschränkungen
- Supplement-Stack → Nova workspace schon prüfen
- Familie: Partner-Gesundheitsziele, Eltern

### Block 3 — Finanzen (Atlas)
- Broker, Sparpläne → Atlas MEMORY.md schon prüfen
- Übergeordnetes Finanzziel
- Crypto/NFT-Tracking-Strategie

### Block 4 — Ferienwohnung (Hera)
- Größe, Kapazität, Check-in-Regeln
- Buchungsregel (Wechseltage)
- Reinigung (Kontakt, Prozess)
- Kurtaxe-Abrechnung
- Automatisierungspotenziale notieren!

### Block 5 — Arbeit (Aven, Titan)
- Projektstatus, Tools, Zugänge
- Wichtigste anstehende Events/Deadlines
- Azure DevOps, GitHub, SharePoint

### Block 6 — Passive Einkommen / Northpeak (Zoe, Vera, Aria)
- Domain-Status, aktuelle Projekte
- Autonomiegrad, Budget
- → Bestehende Paperclip-Issues prüfen vor Fragen!

### Block 7 — Kommunikation & Präferenzen
- Kanäle (Discord/Telegram/Teams)
- Sprache (Deutsch default, Englisch für Firma)
- Tabu-Themen / Privatsphäre

### Block 8 — Präsentationen & Dokumente
- Brand Kit (VC Brand Kit.pptx — Drive ID: 1PDGluRzlUhiOIhWqfA7UCrU2oIqwGkwX)
- Dokumentenformate (.xlsx, .docx — kein Google-native!)
- Ablageorte

## Post-Interview — Memory Updates

1. `~/.openclaw/workspace/shared/user_profile.md` — vollständiges Profil
2. Jede Agent-MEMORY.md direkt beschreiben (kein LLM aufwecken)
3. Mein Hermes-Memory komprimiert aktualisieren

## Key Principle
- Frage immer nur einen Block auf einmal
- Wenn Antwort "das solltet ihr schon wissen" → prüfen ob es wirklich irgendwo steht
- Automatisierungspotenziale (manuelle Prozesse) immer notieren für spätere Tasks
