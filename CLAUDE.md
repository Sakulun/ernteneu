# CLAUDE.md — Ernte 2026 · Nuscheler Unternehmensgruppe

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
| `frontend-design` | Webapps, Dashboards, Landing Pages, UI-Komponenten, HTML/CSS/JS |

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
- Die Hauptanwendung ist eine **Single-File-App** — Änderungen erfolgen immer in `index.html`
- Niemals Build-Tools, Frameworks oder npm-Pakete einführen — die App läuft ohne Build-Prozess
- Keine TypeScript-Migration — alles bleibt Vanilla JS

---

## Projektkontext

Dieses Repository gehört zur **Nuscheler Unternehmensgruppe** (Landwirtschaft/Agrarservice).

**Anwendungsname:** Ernte 2026
**Zweck:** Echtzeit-Ernteverwaltung — Koordination von Mähdreschern, Transportfahrzeugen, Silos und Administration während der Erntekampagne.

Typische Aufgaben: Ernte-App, Datenverwaltung, Dokumentenerstellung, DATEV-Workflows.

---

## Repository-Struktur

```
ernteneu/
├── index.html              # Gesamte Webanwendung (~6.500 Zeilen, Single-File SPA)
├── themes/
│   ├── README.md           # Anleitung zum Theme-Wechsel
│   ├── agrarmonitor.css    # Hell-Theme (Salbeigrün)
│   ├── dark.css            # Dunkel-Theme (Neongrün)
│   └── industrial.css      # Industrial-Theme (Bernstein) — AKTIV
├── waage-bridge/
│   ├── index.js            # Node.js TCP→Supabase Bridge für Waage
│   ├── package.json        # Abhängigkeiten (nur @supabase/supabase-js v2)
│   ├── .env.example        # Konfigurationsvorlage
│   └── start.bat           # Windows-Startskript
└── .claude/
    └── launch.json         # Dev-Server: npx serve auf Port 3000
```

---

## Technologie-Stack

### Frontend
- **Vanilla JavaScript** — kein Framework, kein Build-Prozess
- **Leaflet 1.9.4** — Karten & Feldgrenzen (Polygone)
- **PDF.js 3.11.174** — PDF-Anzeige und -Import
- **Google Fonts** — IBM Plex Mono, Barlow, Barlow Condensed (Industrial-Theme)

### Backend / Datenbank
- **Supabase** (PostgreSQL-BaaS)
  - URL: `https://fijfxmjtoexpuxxjqqbf.supabase.co`
  - Echtzeit-Subscriptions auf allen 15 Tabellen via `postgres_changes`
- **Node.js Bridge** (`waage-bridge/`) — TCP↔Supabase-Sync für Schenck Disomat Opus Waage (MinProz-Protokoll)

### Deployment
- Statischer Datei-Server (kein Backend nötig für die Hauptanwendung)
- `waage-bridge` läuft als separater Node.js-Prozess auf dem Betriebsgelände

---

## Datenbank-Schema (Supabase-Tabellen)

| Tabelle | Beschreibung |
|---|---|
| `nutzer` | Benutzerkonten mit Rollen |
| `felder` | Felder / Schläge |
| `fuhren` | Erntefahrten (eine Fuhre = eine Transportfahrt) |
| `lieferungen` | Lieferdatensätze |
| `silos` | Silobestände |
| `vermehrungen` | Saatgutvermehrungen |
| `shapes` | Geospatiale Feldgrenzen (GeoJSON-Polygone) |
| `gps_positionen` | Live-GPS-Positionen der Maschinen |
| `waage_live` | Live-Waagengewichte vom Bridge-Dienst |
| `artikel` | Warenkatalog (Getreide, Sorten etc.) |
| `kontakte` | Kontaktverwaltung (Lieferanten, Kunden) |
| `kontrakte` | Vertragsmanagement |
| `warenbewegungen` | Lagerzu-/abgänge (Eingang & Ausgang) |
| `nachrichten` | In-App-Benachrichtigungen |
| `jd_tokens` | John Deere Maschinenintegration (OAuth-Tokens) |

---

## Benutzerrollen & Dashboards

| Rolle | Farbe | Hauptfunktionen |
|---|---|---|
| **Drescher** | Bernstein `#c8962e` | Ernteaufträge annehmen, Felder abarbeiten, Abschluss melden |
| **Abfahrer** | Blau `#4a8ab0` | Fuhren mit Voll-/Leergewicht erfassen, Waagenwidget nutzen |
| **Silomeister** | — | Silobestände verwalten, Warenbewegungen buchen |
| **Admin** | Gold `#c8a84b` | Vollzugriff: Dashboard, Fortschritt, Fuhren, Schläge, Nutzer, Kontrakte, KDV |

### Haupt-Render-Funktionen

```
renderLogin()          — Anmeldebildschirm
renderMain()           — Router → rollenspezifisches Dashboard
renderDrescher()       — Drescher-Dashboard
renderAbfahrer()       — Abfahrer-Dashboard (offene Fuhren)
renderAbfahrerOffen()  — Offene Lieferungen
renderAbfahrerFertig() — Abgeschlossene Lieferungen
renderAdmin()          — Admin-Panel mit Sidebar-Navigation
renderSilomeister()    — Silomeister-Dashboard

renderAdminDash()      — Admin-Übersicht
renderAdminFuhren()    — Fuhrenverwaltung
renderAdminSchlaege()  — Schlagverwaltung
renderAdminKarte()     — Kartenansicht aller Felder
renderAdminNutzer()    — Nutzerverwaltung
renderAdminFortschritt() — Fortschrittsanalyse
```

---

## Code-Konventionen

### Supabase-Datenbankzugriff

Immer `try/catch` verwenden — **kein** `.catch()`:

```javascript
// RICHTIG
try {
  const { data, error } = await sb.from('fuhren').select('*').order('id');
  if (error) throw error;
} catch (err) {
  console.error(err);
}

// FALSCH — nicht verwenden
sb.from('fuhren').select('*').then(...).catch(...);
```

### UI-Rendering-Muster

Die App rendert die gesamte UI durch DOM-String-Injection in `#app`:

```javascript
function renderBeispiel() {
  document.getElementById('app').innerHTML = `
    <div class="card">...</div>
  `;
  // Event-Listener danach binden
  document.getElementById('btn-save').addEventListener('click', speichern);
}
```

### Supabase Echtzeit-Subscriptions

```javascript
sb.channel('tabelle-changes')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'fuhren' }, () => {
    renderAktuelleAnsicht();
  })
  .subscribe();
```

### Stil & CSS

- Inline-`<style>`-Block am Ende von `index.html` — hier CSS-Änderungen vornehmen
- Theme-Overrides in `themes/*.css` — für vollständige Theme-Wechsel
- Aktives Theme: **industrial.css** (Dunkel/Bernstein)
- CSS-Klassen direkt als Strings in Template-Literals — kein CSS-in-JS

---

## Waage-Bridge

Der `waage-bridge/`-Dienst verbindet eine **Schenck Disomat Opus** Waage via TCP (MinProz-Protokoll) mit Supabase.

**Konfiguration (`.env`):**
```
WAAGE_IP=192.168.1.50      # IP der Waage im Netzwerk
WAAGE_PORT=8000             # TCP-Port
POLL_CMD=SI                 # MinProz-Befehl (Stable/Instable)
POLL_MS=2000                # Abfrageintervall in ms
SUPABASE_URL=...
SUPABASE_KEY=...            # Service-Role-Key (nicht Anon-Key!)
```

**Antwortformat der Waage:** `+014500.000 kg ST` (Wert, Einheit, Status ST/US/OL/ER)

**Starten:** `cd waage-bridge && npm install && node index.js`

---

## Entwicklungsworkflow

### Lokale Entwicklung starten
```bash
npx serve . -p 3000
# Anwendung öffnen: http://localhost:3000
```

### Änderungen an `index.html` vornehmen
1. Direkt in `index.html` editieren — die gesamte Anwendungslogik ist hier
2. Browser-Tab neu laden (kein Build nötig)
3. Für Datenbankänderungen: Supabase-Dashboard nutzen

### Theme wechseln
Methode A (Inline): CSS-Block am Ende des `<style>`-Tags in `index.html` durch den Inhalt eines `themes/*.css` ersetzen.
Methode B (Extern): `<link rel="stylesheet" href="themes/industrial.css">` ans Ende des `<head>` anfügen.

### Git-Branches
- `main` — Produktionszweig
- Feature-Branches nach Muster `claude/beschreibung-XXXXX`

---

## Schlüsselkennzahlen

| Kennzahl | Wert |
|---|---|
| Hauptdatei | `index.html` (~6.500 Zeilen) |
| JavaScript-Funktionen | 166+ |
| CSS-Klassen | 507 |
| Supabase-Tabellen | 15 |
| Datenbankoperationen | 132+ |
| Benutzerrollen | 4 |
| Externe Abhängigkeiten | 4 (Supabase JS, Leaflet, PDF.js, Google Fonts) |
| Build-Prozess | Keiner |
