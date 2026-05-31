---
name: vibracoustic-presentation
description: Use when creating PowerPoint presentations in Vibracoustic corporate design for Kai's work context. Triggers on mentions of Vibracoustic, VBC, VC, Firma, Lenkungsausschuss, Freigabe, Steering Committee, or any slide deck request in a Kai business/work context.
version: 1.0.0
author: Luna
license: Proprietary
platforms: [macos]
metadata:
  hermes:
    tags: [vibracoustic, presentation, powerpoint, pptx, corporate, brand, firma]
    related_skills: [powerpoint, google-workspace]
---

# Vibracoustic Presentation Skill

## Overview

Erstellt PowerPoint-Präsentationen im Vibracoustic Corporate Design. Die Vorlagen liegen in Kais Google Drive (via `gog` mit Avens Account erreichbar) und wurden analysiert — alle Farben, Schriften, Layouts und Placeholder-Indizes sind hier dokumentiert.

**Template-Dateien (lokal, falls vorhanden):**
- `/tmp/vibracoustic-brand/VC_Brand_Kit.pptx` — 463 KB, Kern-Vorlage mit allen Layouts
- `/tmp/vibracoustic-brand/Vibracoustic_PPT-SlideLibrary.pptx` — 59 MB, 148 Beispielfolien

**Template aus Drive holen (falls /tmp nicht vorhanden):**
```bash
mkdir -p /tmp/vibracoustic-brand
gog drive download 1PDGluRzlUhiOIhWqfA7UCrU2oIqwGkwX \
  --account aven73.claw@gmail.com \
  --output /tmp/vibracoustic-brand/VC_Brand_Kit.pptx
```

## Trigger-Erkennung

Nutze diesen Skill wenn Kai:
- „Vibracoustic", „VBC", „VC" erwähnt
- „Präsentation für die Firma" / „Firmen-Deck" sagt
- Kontext: Lenkungsausschuss, Steering Committee, Freigabe, Management Update, Board, SCM
- Eine `.pptx` erstellen will und erkennbar im Arbeitskontext ist (nicht privat/Northpeak)

## Corporate Design

### Farben

| Rolle | Name | HEX | RGB |
|-------|------|-----|-----|
| **Primär** | VBC Dark Blue | `#024066` | 2/64/102 |
| **Akzent** | VBC Cyan | `#00BCF1` | 0/188/241 |
| **Sekundär** | VBC Medium Blue | `#67A2C0` | 103/162/192 |
| **Grau hell** | VBC Light Gray | `#C1CAD0` | 193/202/208 |
| **Grau dunkel** | VBC Dark Gray | `#5A6870` | 90/104/112 |
| **Teal** | VBC Teal | `#008B95` | 0/139/149 |
| **Grün** | VBC Green | `#009A6E` | 0/154/110 |
| Hintergrund hell | — | `#E8E8E8` | 232/232/232 |
| Text / Schwarz | — | `#000000` | — |
| Weiß | — | `#FFFFFF` | — |
| Rot (Alert) | — | `#B9003F` | — |
| Orange | — | `#F39200` | — |

**Hauptfarbe für Header-Balken/Akzente:** `#024066` (Dark Blue)
**Hintergrundfarbe Slides:** Weiß (`#FFFFFF`) mit hellgrauem Footer-Bereich
**Titeltext:** Schwarz auf weißem Hintergrund

### Schrift

**Einzige Schriftart: Tahoma** — für Titel, Überschriften und Fließtext.

| Element | Schrift | Größe | Stil |
|---------|---------|-------|------|
| Slide-Titel | Tahoma | 20–24pt | Bold |
| Topline (Kategorie) | Tahoma | 10–12pt | Regular, dunkelblau |
| Body Text Level 1 | Tahoma | 14–16pt | Regular |
| Body Text Level 2+ | Tahoma | 12–13pt | Regular |
| Footer / Datum | Tahoma | 9–10pt | Regular |
| Chapter Number | Tahoma | 36–48pt | Bold, dunkelblau |

### Foliengröße

**16:9 Widescreen:** 33,87 cm × 19,05 cm (13,33" × 7,50")

### Design-Prinzipien

- Weißer Hintergrund für Content-Folien
- Dunkler Header-Balken (`#024066`) für Cover und Chapter Divider
- **Topline** (kleiner Text über dem Titel): Bereich/Kategorie der Folie
- **Footer** jeder Folie: Datum links | Präsentationstitel Mitte | Seitenzahl links-außen
- Keine dekorativen Linien unter Titeln — nur Whitespace
- Bullet-Zeichen: Pfeil in Primärblau

## Verfügbare Layouts

### 1. `Title Page` — Titelfolie (Cover)
Zwei Varianten: „Master Intern" (interne Präsentationen) und „Master Extern" (externe).

| Placeholder | idx | Inhalt |
|------------|-----|--------|
| Präsentationstitel | 0 | Titel der Präsentation |
| Untertitel / Anlass | 1 | Datum, Autor, Anlass |

---

### 2. `Chapter Divider` — Kapiteltrennfolie
Dunkler Hintergrund mit Bildplatzhalter, Kapitelnummer und Kapiteltitel.

| Placeholder | idx | Inhalt |
|------------|-----|--------|
| Bild | 13 | Hintergrundbild (optional) |
| Kapitel-Titel | 0 | „Kapitelname" |
| Kapitel-Nummer | 14 | „01", „02", … |
| Footer Titel | 10 | Präsentationstitel |
| Datum | 11 | Datum |
| Seitenzahl | 12 | Auto |

---

### 3. `1 Content` — Standard-Inhaltsfolie ⭐ (häufigste)
Voller Content-Bereich unter dem Titel. Für Text, Tabellen, Diagramme.

| Placeholder | idx | Position | Inhalt |
|------------|-----|----------|--------|
| Topline | 18 | oben, 1.7cm/1.1cm | Bereich/Kategorie |
| Titel | 0 | oben, 1.7cm/2.1cm | Folien-Überschrift |
| Content | 1 | 1.1cm/4.3cm, 31.6×12.6cm | Text / Bullets / Tabelle |
| Datum | 15 | Footer links | |
| Fußzeile | 16 | Footer Mitte | Präsentationstitel |
| Seitenzahl | 17 | Footer links-außen | |

---

### 4. `2 Contents` — Zwei-Spalten ohne Überschriften
| Placeholder | idx | Position |
|------------|-----|----------|
| Topline | 21 | oben |
| Titel | 0 | oben |
| Content Links | 19 | 1.1cm/4.3cm, 15.4×12.6cm |
| Content Rechts | 20 | 17.3cm/4.3cm, 15.4×12.6cm |

---

### 5. `2 Contents with Headlinebox` — Zwei Spalten mit je Überschrift
| Placeholder | idx | Inhalt |
|------------|-----|--------|
| Topline | 18 | Bereich |
| Titel | 0 | Folien-Titel |
| Überschrift Links | 15 | Spalten-Titel links |
| Überschrift Rechts | 28 | Spalten-Titel rechts |
| Content Links | 26 | 1.1cm/5.5cm, 15.4×11.4cm |
| Content Rechts | 27 | 17.3cm/5.5cm, 15.4×11.4cm |

---

### 6. `4 Contents with Headlinebox` — 2×2 Raster ⭐
Vier gleichgroße Bereiche, je mit eigener Überschrift.

| Placeholder | idx | Position |
|------------|-----|----------|
| Topline | 18 | oben |
| Titel | 0 | oben |
| Überschrift oben-links | 41 | 1.1cm/4.3cm |
| Überschrift oben-rechts | 28 | 17.3cm/4.3cm |
| Überschrift unten-links | 43 | 1.1cm/11.1cm |
| Überschrift unten-rechts | 42 | 17.3cm/11.1cm |
| Content oben-links | 37 | 1.1cm/5.5cm, 15.4×4.6cm |
| Content oben-rechts | 38 | 17.3cm/5.5cm, 15.4×4.6cm |
| Content unten-links | 39 | 1.1cm/12.3cm, 15.4×4.6cm |
| Content unten-rechts | 40 | 17.3cm/12.3cm, 15.4×4.6cm |

---

### 7. `6 Contents with Headlinebox` — 2×3 Raster
| Placeholder | idx | Position (cm) |
|------------|-----|---------------|
| Topline | 18 | oben |
| Titel | 0 | oben |
| Üschr. oben: links/mitte/rechts | 35/28/56 | Row 1 |
| Üschr. unten: links/mitte/rechts | 58/57/59 | Row 2 |
| Content oben: 1/2/3 | 47/48/49 | je 10×4.6cm |
| Content unten: 4/5/6 | 53/54/55 | je 10×4.6cm |

---

### 8. `Only Title` — Nur Titel (für Custom-Layouts)
Titel (idx=0) + Topline (idx=18), kein Content-Placeholder. Freifläche für manuell positionierte Shapes.

---

### 9. `Picture` / `2 Pictures` — Bildfolien
Bild-Placeholder(s) unter dem Titel. idx=13 (links/einzel), idx=18 (rechts).

---

### 10. `Contact` — Kontaktfolie
Letzte Folie mit Kontaktdaten. Text-Placeholders idx=19–30 für Name, Titel, E-Mail usw.

---

### 11. `3 Contents with Headlinebox` — 3 Bereiche (T-Form)
Zwei oben nebeneinander (idx=38, 40) + ein breiter unten (idx=26).

### 12. `1 Content 2 Contents` — Groß links + 2 klein rechts
Links eine große Fläche (idx=26), rechts zwei gestapelte (idx=37, 38).

### 13. `Content and 2 details` — Groß oben + 2 klein unten
Großer Content-Bereich oben (idx=41), zwei Details unten (idx=39, 40).

## Workflow: Präsentation erstellen

### ⚠️ WICHTIGSTE REGEL: Script verwenden, keinen Code generieren!

**Das validierte Script `scripts/create_vbc_presentation.py` ist die einzige erlaubte Methode.**
Niemals python-pptx-Code neu generieren — das ist die Quelle aller bisherigen Fehler.

### Schritt 1: Script laden und parametrieren

```python
# Script befindet sich in:
# ~/.hermes/skills/productivity/vibracoustic-presentation/scripts/create_vbc_presentation.py

# 1. META anpassen:
META = {
    "titel":     "Meine Vibracoustic Präsentation",
    "topline":   "Bereich | Thema | Datum",
    "untertitel": "Kai Voges  |  01.06.2026",
    "datum":     "06/01/2026",
}

# 2. SLIDES befüllen — nur diese Layouts erlaubt:
#    "Title Page", "1 Content", "2 Contents", "2 Contents with Headlinebox",
#    "4 Contents with Headlinebox", "6 Contents with Headlinebox", "Chapter Divider"

SLIDES = [
    {"layout": "Title Page", "titel": "...", "untertitel": "..."},
    {"layout": "1 Content",  "topline": "...", "titel": "...", "content": "..."},
    # usw. — siehe Script für alle Felder
]
```

### Schritt 2: Ausführen

```bash
uv run --with python-pptx python3 \
  ~/.hermes/skills/productivity/vibracoustic-presentation/scripts/create_vbc_presentation.py
```

Der eingebaute Validator prüft **automatisch** vor dem Speichern:
- Kein Blank / Only Title Layout
- Keine freien Shapes (> 2 pro Slide)
- Keine Nicht-Tahoma-Schriften

Schlägt die Validierung an → Datei wird **nicht** gespeichert, Fehler werden ausgegeben.

### Schritt 3: In Drive hochladen

```bash
gog drive upload /tmp/vibracoustic-brand/output.pptx \
  --account aven73.claw@gmail.com
# Link aus dem Output posten
```

## Workshop-Präsentation Pattern

Für Workshoptage (Working Session, Strategie-Offsite, Workstream-Meetings) gibt es ein bewährtes Slide-Gerüst das Kai nutzt. Dieses Pattern wurde im Orga-2.0-Workshop (Juni 2026) etabliert.

### Confirmed live colors (VC_v1 live ausgelesen, Mai 2026)
| Rolle | HEX | Verwendung |
|-------|-----|-----------|
| `#024066` | DARK_BLUE | Header-Balken, Chapter-Panel links, Footer |
| `#00BCF1` | LIGHT_BLUE | Akzent-Streifen, Card-Nummern, Trennlinien |
| `#F26B43` | ORANGE | STATUS QUO Label, Alert-Karten |
| `#008B95` | TEAL | DIRECTION Label, positive Karten |
| `#5A6870` | GRAY | Sekundärtext, Beschreibungen |
| `#F5F8FB` | LIGHT_GRAY | Card-Hintergründe |
| `#7F1D1D` | DARK_RED | Problemtext (✕ Punkte) |
| `#06422C` | DARK_GREEN | Lösungstext (→ Punkte) |

### Workshop Slide-Typen

**STATUS QUO / DIRECTION** (2-Spalten-Karte):
- Links orange Header "STATUS QUO" + Problemliste mit `✕` in DARK_RED
- Rechts teal Header "DIRECTION" + Lösungen mit `→` in DARK_GREEN
- Beide Karten: LIGHT_GRAY Hintergrund, gleiche Höhe

**Chapter Divider**:
- Linkes Panel 8cm breit, DARK_BLUE
- Große Kapitelnummer (72pt, LIGHT_BLUE)
- Vertikaler LIGHT_BLUE Akzentstreifen rechts vom Panel
- Titel (28pt, bold, DARK_BLUE) auf weißem Hintergrund

**Discussion Cards**:
- Nummerierte Karten mit LIGHT_BLUE Nummer-Box
- Frage in bold DARK_BLUE, Hinweistext in GRAY

**Typische Workshop-Gliederung:**
1. Cover (linkes Panel DARK_BLUE, Titel rechts)
2. Agenda (nummerierte Zeilen, abwechselnd LIGHT_GRAY / heller Blau)
3. Chapter Divider → Context & Mandate
4. Business-Kontext-Folien (Ursachen/Treiber)
5. Chapter Divider → Was nicht funktioniert
6. STATUS QUO / DIRECTION Folien (3 Themenblöcke) + Diskussionsrunden
7. Chapter Divider → Greenfield Vision
8. Prinzipien-Folie (6-Karten Grid) + Diskussionsrunde
9. Accountability-Modell (Tabelle mit Schichten)
10. Sacred Cows / Rules to Challenge + Diskussionsrunde
11. Chapter Divider → Work Packages
12. Work-Package-Tabelle (ausfüllbar)
13. Open Questions für andere Workstreams
14. Timeline (Meilenstein-Leiste)
15. Closing (vollflächig DARK_BLUE)

## Standard-Präsentationsstruktur (VBC-Konvention)

1. **Titelfolie** (`Title Page`) — Thema, Autor, Datum, Anlass
2. **Agenda** (`1 Content`) — Nummerierte Gliederung
3. **Chapter Divider** pro Kapitel — Kapitel-Nr. + Titel
4. **Content-Folien** — je nach Inhalt Layout wählen (s. oben)
5. **Abschlussfolie** — Nächste Schritte oder Kontakt (`Contact`)

Für komplexe Workshop-Decks: `references/workshop_orga20_jun2026.md`

## Common Pitfalls

1. **⛔ NIEMALS `Blank`- oder `Only Title`-Layout als Basis für Inhaltsfolien verwenden** — beide Layouts liefern keinen Content-Placeholder. Wer sie trotzdem nutzt, baut zwangsläufig freie Shapes — was sofort zu falschem Layout führt. `Only Title` ist ausschließlich für Custom-Visualisierungen reserviert die wirklich kein Raster brauchen (z.B. Organigramm-SVG). Im Zweifel: `1 Content` nehmen.

2. **⛔ NIEMALS freie TextBoxes, Rectangles oder Shapes für Inhalte, Header oder Footer bauen** — das komplette visuelle Rahmenwerk (blauer Header-Balken oben, Cyan-Trennlinie, Footer-Bereich unten) kommt automatisch vom Slide Master. Sobald mehr als 2–3 freie Shapes pro Slide entstehen, ist das ein Warnsignal: falsches Layout gewählt. Stopp — Layout wechseln statt weiterbauen.

2a. **✅ Selbstcheck vor dem Speichern:** Zähle freie Shapes pro Slide (`[s for s in slide.shapes if s.shape_type != 14]`). Mehr als 2 freie Shapes = Layout-Fehler. Sofort korrigieren.

3. **⛔ NIEMALS Calibri verwenden** — python-pptx setzt Calibri als Default-Schrift wenn man TextBoxes manuell erstellt. Immer explizit `Tahoma` setzen oder besser: nur Template-Placeholders nutzen (die erben Tahoma automatisch vom Master). Prüfen: `run.font.name = "Tahoma"`.

4. **Placeholder-Indizes sind layout-spezifisch** — nie blind `placeholders[0]` ohne Layout-Kenntnis verwenden. Immer `ph.placeholder_format.idx` prüfen.

5. **Template-Slides müssen entfernt werden** — `Presentation(TEMPLATE)` enthält 7 Beispiel-Slides; alle löschen bevor neue hinzugefügt werden (s. Schritt 2).

6. **Footer-Placeholders immer befüllen** — Datum und Fußzeile in jedem Slide befüllen, sonst bleiben Placeholder-Texte (`04/05/2026`, `|  Title of Presentation`) stehen.

7. **Topline (idx=18) immer befüllen** — gehört zum VBC-Standard, nie weglassen.

8. **`/tmp/vibracoustic-brand/` kann fehlen** — vor Verwendung prüfen; Drive-ID für Download: `1PDGluRzlUhiOIhWqfA7UCrU2oIqwGkwX`.

9. **QA — visuell prüfen**: LibreOffice nicht auf diesem System → Datei in Drive hochladen und in Google Slides visuell prüfen. Checkliste: Tahoma? Richtiger Header-Balken? Cyan-Linie? Footer mit Datum/Titel/Seitenzahl? Kein Calibri?

10. **Überlagerungen vermeiden — Textmenge begrenzen:**
    - `1 Content` Placeholder fasst max. ~400 Zeichen komfortabel (31.6×12.6cm)
    - `4 Contents` Zellen: max. ~150 Zeichen pro Zelle (15.4×4.6cm)
    - `6 Contents` Zellen: max. ~100 Zeichen pro Zelle (10×4.6cm)
    - Bei mehr Text: auf mehrere Slides aufteilen oder kleineren Schriftgrad setzen (`Pt(11)`)
    - Niemals Text erzwingen der nicht in den Placeholder passt — lieber kürzen

11. **Schrift explizit setzen wenn nötig:**
    ```python
    for para in placeholder.text_frame.paragraphs:
        for run in para.runs:
            run.font.name = "Tahoma"
            run.font.size = Pt(12)
    ```

8. **Bei Workshop-Präsentationen: frisch bauen statt Template-Slides kopieren** — für komplexe Workshop-Decks (Diskussionsrunden, Accountability-Tabellen, Kapitel-Divider mit Nummern) ist es sauberer, eine `NewPresentation()` zu erstellen und alle Shapes manuell zu platzieren. Das Template wird dann nur als Farb-/Font-Referenz genutzt.

9. **python-pptx über pip3 installieren, nicht uv** — auf diesem System (macOS, python3.11 via CommandLineTools) klappt `pip3 install python-pptx openpyxl` zuverlässig. `uv pip install` liefert kein importierbares Modul für den System-python3.

10. **Inline-`&` in `python3 -c` String blockiert** — Terminal blockiert wenn der Python-Code ein `&` Zeichen enthält (wird als Shell-Backgrounding interpretiert). Stattdessen Code in `/tmp/script.py` schreiben und mit `python3 /tmp/script.py` aufrufen.

11. **`~/Documents/` via terminal `ls` kann >60s timeouten** (macOS Spotlight/Indexierung). Stattdessen: `file://` im Browser navigieren oder direkte absolute Pfade zu bekannten Unterordnern verwenden.

12. **`gog drive download` lädt nach `~/Library/Application Support/gogcli/drive-downloads/` herunter** (nicht ins Arbeitsverzeichnis). Dateiname = `<fileId>_<originalName>`. Immer mit vollem Pfad referenzieren.

## Kontext-Erkennung

**VBC-Design verwenden bei:** Vibracoustic / VBC / VC / SCM / Steering / Lenkungsausschuss / Freigabe / Management Update / Orga-Deck / Firma-Kontext / **Orga 2.0 / Workstream / Workshop**

**Kein VBC-Design bei:** Northpeak/Paperclip-Präsentationen, private Decks (Schule, Familie), OpenClaw-Statusberichte

**WICHTIG — Skill MUSS automatisch geladen werden:** Sobald Kai ein Slide-Deck in seinem Arbeitskontext anfordert, diesen Skill **vor** jeder anderen Aktion laden. Nicht erst nach dem Lesen der Drive-Dateien beginnen — Brand-Farben und Layout-Patterns sind hier vollständig dokumentiert.
