# CLAUDE.md — Webapp Development Configuration

## Skills

Claude verwendet folgende Skills automatisch, wenn sie relevant sind.
Direkter Aufruf auch per Slash-Command möglich (z.B. `/frontend-design`).

### Dokumente & Dateien

| Skill | Wann automatisch aktiv |
|---|---|
| `docx` | Word-Dokumente erstellen, bearbeiten, Vorlagen, Berichte, Beschlüsse |
| `xlsx` | Excel-Tabellen erstellen oder bearbeiten (.xlsx, .csv, .tsv) |
| `pdf` | PDFs lesen, erstellen, zusammenführen, Formulare ausfüllen |
| `pdf-reading` | Inhalt aus PDF-Uploads lesen und extrahieren |
| `pptx` | PowerPoint-Präsentationen erstellen oder bearbeiten |
| `file-reading` | Beliebige Datei-Uploads lesen (Router für alle Formate) |

### Frontend & Design

| Skill | Wann automatisch aktiv |
|---|---|
| `frontend-design` | Webapps, Dashboards, Landing Pages, UI-Komponenten, React, HTML/CSS |

### Produktwissen

| Skill | Wann automatisch aktiv |
|---|---|
| `product-self-knowledge` | Fragen zu Claude Code, Claude API, Claude.ai Features/Preisen |

---

## Allgemeine Anweisungen

- Antworte auf Deutsch, wenn der Nutzer auf Deutsch schreibt
- Bevorzuge direkte, effiziente Antworten ohne unnötiges Hin-und-Her
- Bei Code: Erstelle immer vollständige, lauffähige Dateien (keine Fragmente)
- Bei Unsicherheit über Dateiinhalte: Nutze den `file-reading` Skill

---

## Projektkontext

Dieses Repository gehört zur **Nuscheler Unternehmensgruppe** (Landwirtschaft/Agrarservice).
Typische Aufgaben: Ernte-App, Datenverwaltung, Dokumentenerstellung, DATEV-Workflows.
