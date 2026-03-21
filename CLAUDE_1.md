# CLAUDE.md – Ernte 2026 PWA

Dieses Dokument beschreibt das Projekt vollständig für Claude Code.

---

## Projektübersicht

**Ernte 2026** ist eine Progressive Web App (PWA) zur digitalen Ernteerfassung für einen landwirtschaftlichen Lohnunternehmer in Sachsen-Anhalt. Die App koordiniert Drescherfahrer, Abfahrer (LKW-Fahrer) und den Betriebsleiter in Echtzeit während der Getreideernte.

**Technologie:** Single-File HTML/CSS/JS (534 KB), kein Build-System, kein Framework  
**Hosting:** GitHub Pages → `https://sakulun.github.io/ernteneu/`  
**Repository:** `Sakulun/ernteneu`  
**Datei:** `ernte2026.html` (wird als `index.html` deployed)

---

## Backend: Supabase

```
URL:  https://fijfxmjtoexpuxxjqqbf.supabase.co
KEY:  sb_publishable_HPUpEywyC27cmVfthrqY4A_aWeuYth1
```

**Realtime** ist aktiv für: `fuhren`, `gps_positionen`, `nachrichten`  
**RLS** ist aktiviert auf allen Tabellen mit `allow_all_anon` Policy.

---

## Datenbankschema

```sql
nutzer (
  id SERIAL PK,
  name TEXT,
  rolle TEXT,          -- 'admin' | 'drescher' | 'abfahrer' | 'silomeister'
  pw TEXT,             -- SHA-256(username:password)
  label TEXT
)

felder (
  id SERIAL PK,
  name TEXT,
  flaeche NUMERIC,
  fruchtart TEXT,
  status TEXT,         -- 'aktiv' | 'inaktiv' | 'abgeerntet'
  betrieb TEXT         -- Betriebsname (wichtig für Bio-Trennung!)
)

fuhren (
  id SERIAL PK,
  nr TEXT,             -- z.B. 'F-001'
  status TEXT,         -- 'offen' | 'fertig'
  drescher_id INT,
  abfahrer_id INT,
  feld_id INT,
  fruchtart TEXT,
  sorte TEXT,          -- NULL = Konsum, gesetzt = Vermehrungssorte (z.B. 'Toras')
  vollgewicht INT,     -- kg
  leergewicht INT,     -- kg
  feuchte NUMERIC,
  fallzahl INT,
  protein NUMERIC,
  hl_gewicht NUMERIC,
  gluten NUMERIC,
  oelgehalt NUMERIC,
  zeit TIMESTAMPTZ,
  verifiziert BOOLEAN,
  verifiziert_von INT,
  silo_id TEXT,
  feld_id_korr INT,
  drescher_id_korr INT,
  abfahrer_id_korr INT,
  fruchtart_korr TEXT
)

silos (
  id TEXT PK,          -- z.B. 'A1', 'B1', 'HOF'
  kapazitaet_t NUMERIC,
  fruchtart TEXT,
  notiz TEXT
)

shapes (
  feld_id INT PK,
  betrieb TEXT,
  outer_coords JSONB,  -- [[lat,lng], ...]
  holes JSONB
)

vermehrungen (
  id SERIAL PK,
  feld_id INT,
  sorte TEXT,          -- z.B. 'Toras', 'Tobak', 'Souleyka', 'SU Ahuriri'
  flaeche NUMERIC,
  fruchtart TEXT,
  bio BOOLEAN
)

gps_positionen (
  nutzer_id INT PK,
  lat NUMERIC,
  lon NUMERIC,
  aktualisiert_am TIMESTAMPTZ
)

nachrichten (
  id SERIAL PK,
  text TEXT,
  empfaenger TEXT,     -- 'alle' | 'drescher' | 'abfahrer' | user_id
  von INT,
  von_name TEXT,
  gelesen BOOLEAN,
  zeit TIMESTAMPTZ
)

lieferungen (
  id SERIAL PK,
  nr TEXT,
  datum DATE,
  kaeufer_name TEXT,
  kaeufer_adresse TEXT,
  kennzeichen TEXT,
  spedition TEXT,
  kontrakt TEXT,
  fruchtart TEXT,
  leergewicht INT,
  vollgewicht INT,
  notiz TEXT,
  status TEXT          -- 'offen' | 'abgeschlossen'
)

jd_tokens (
  user_id INT PK,
  access_token TEXT,
  refresh_token TEXT,
  expires_at TIMESTAMPTZ
)
```

---

## Nutzer & Rollen

| Name | Rolle | Passwort |
|------|-------|----------|
| Betriebsleiter | admin | admin123 |
| Silomeister | silomeister | silo123 |
| Hans Müller | drescher | drescher1 |
| Karl Bauer | drescher | drescher2 |
| Peter Schmidt | abfahrer | abfahrer1 |
| Thomas Wagner | abfahrer | abfahrer2 |
| Fritz Lange | abfahrer | abfahrer3 |

**Passwort-Hashing:** `SHA-256(name.toLowerCase() + ':' + passwort)`  
Format in DB: `s:[64-char-hex]`

---

## Betriebe & Bio-Trennung

```javascript
const BIO_BETRIEBE = ['Niko von Reiche', 'Bernhard von Reiche', 'von Reiche GbR', 'ABB'];
// ABB = Ackerbau Beesenstedt GbR
```

**Konventionelle Betriebe:** AGH, J&S, LW Höhnstedt, Landgut, Viehmast

**Silo-Sperre:** Bio-Ware und konventionelle Ware dürfen NICHT in dasselbe Silo. Hart gesperrt, keine Ausnahmen.  
**Vermehrungen:** Jede Sorte bekommt ihr eigenes Silo. Toras ≠ Tobak ≠ Konsum, auch bei gleicher Fruchtart.

---

## Vermehrungen (Demo-Daten)

| Schlag | Sorte | Fläche | Fruchtart |
|--------|-------|--------|-----------|
| Pfingstwiese 1 (ID 3) | Toras | 40 ha | Winterweichweizen |
| Pfingstwiese 1 (ID 3) | Tobak | 2 ha | Winterweichweizen |
| Zuckerfabrik (ID 7) | Souleyka | 16 ha | Wintergerste |
| Zins (ID 8) | SU Ahuriri | 16 ha | Winterweichweizen |

Flächenangaben sind **Teilflächen** der Gesamtfläche (Konsum + Vermehrung = Gesamtfläche).

---

## Silos

- **B-Silos:** B1–B5, je 1.000 t, 2+2+1 Layout
- **A-Silos:** A1–A21, je 300 t, 3 Spalten
- **HOF:** Gemischtes Zwischenlager, keine Fruchtart-Sperre

---

## John Deere API

```
Client ID:     0oatm7a63xrKg50rk5d7
Client Secret: in Supabase Edge Function 'jd-token' als Secret gespeichert
Redirect URI:  https://sakulun.github.io/ernteneu/
Status:        Sandbox (Production beantragt)
```

**Edge Function URL:** `https://fijfxmjtoexpuxxjqqbf.supabase.co/functions/v1/jd-token`  
Token-Austausch läuft über diese Edge Function, Secret nie im Browser.

---

## Firmendaten (für Lieferschein PDF)

```
Saatgut & Agrarservice Beesenstedt GmbH
Bahnhofstraße 11
06198 Salzatal OT Beesenstedt

Bank 1: Saalkreissparkasse
Bank 2: Raiffeisenbank Pfaffenwinkel

Bio-Zertifikat: DE-ÖKO-006
```

---

## App-Architektur

Die gesamte App ist eine **einzige HTML-Datei** ohne Build-System:

```
ernte2026.html
├── <head>
│   ├── Leaflet.js (Karte)
│   ├── Supabase JS Client
│   └── CSS (alle Styles inline)
├── <body>
│   ├── #login-screen
│   ├── #app
│   │   ├── .topbar
│   │   ├── #admin-sidebar (nur Admin)
│   │   └── #admin-main / Drescher / Abfahrer Views
│   └── Modals (Silo-Overlay, Nachrichten, Onboarding)
└── <script>
    ├── State Management (state = {...})
    ├── DB Layer (db.getFuhren(), db.insertFuhre(), ...)
    ├── Render Functions (renderAdmin(), renderDrescher(), ...)
    ├── Event Handlers
    └── Boot (bootApp())
```

**State-Objekt:**
```javascript
state = {
  currentUser: null,
  users: [],
  felder: [],
  fuhren: [],
  silos: [],
  vermehrungen: [],
  nachrichten: [],
  lieferungen: [],
  nextNr: 1,
  nextId: 1,
  lastFeldId: null
}
```

---

## Rollen & Views

### Admin (Betriebsleiter)
- Dashboard: Statistiken, Fortschrittsbalken, Kulturtabelle
- Schläge: Aktivieren/Deaktivieren, Status setzen
- Fuhren: Alle Fuhren, Verifizierung, Korrekturen
- Silos: Drag & Drop Zuordnung, Kapazitätsanzeige
- Karte: Live GPS aller Fahrer + Schlagpolygone + JD-Maschinen
- Warenausgang: Lieferscheine erstellen, PDF Export
- Nachrichten: Push an alle/Drescher/Abfahrer/Einzelperson
- Nutzer: Anlegen, bearbeiten
- Neues Erntejahr: CSV/Excel/KML Import
- Tagesbericht: PDF A4 Querformat

### Drescher
- Tab Zuweisung: Schlag wählen → Sorte (Konsum/Vermehrung) → Abfahrer → Starten
- Tab Meine Fuhren: Übersicht eigener Fuhren
- Tab Schlag: Karte mit Navigation zu aktivem Schlag
- Onboarding beim ersten Login

### Abfahrer
- Tab Offen: Aktuelle Fuhre mit Bio/Vermehrungsbanner
- Tab Erledigt: Abgeschlossene Fuhren
- Tab Schlag: Karte mit Navigation
- Gewichtseingabe: Vollgewicht, Leergewicht, Qualitätsfelder je Fruchtart
- Onboarding beim ersten Login

---

## Qualitätsfelder je Kultur

| Kultur | Felder |
|--------|--------|
| Getreide | Feuchte, Protein, Gluten, HL-Gewicht, Fallzahl |
| Raps / Sonnenblumen | Feuchte, Ölgehalt, HL-Gewicht |
| Leguminosen | Feuchte, Protein, Ölgehalt |
| Mais / Rüben | Feuchte |

**Feuchte-Warnungen:** Getreide > 15%, Raps/Sonnenblumen > 9%, Leguminosen > 13%

---

## Design System

**Thema:** Dark Harvest – iOS-inspiriertes Dark Theme

```css
--bg:     #1c1c1e   /* iOS Systemschwarz */
--card:   #2c2c2e   /* iOS erhöhte Oberfläche */
--bg3:    #3a3a3c   /* iOS tertiär */
--text:   #ffffff
--text2:  #d1d1d6
--text3:  #636366
--green:  #30d158   /* iOS Systemgrün */
--blue:   #0a84ff   /* iOS Systemblau */
--amber:  #ff9f0a   /* iOS Systemorange */
--red:    #ff453a
--radius: 16-18px   /* Großzügige Rundungen */
Font: -apple-system, BlinkMacSystemFont, 'SF Pro Display'
```

**Kacheln/Cards:** `background:#2c2c2e`, `border-radius:18px`, kein Border, kein Schatten  
**Buttons:** `border-radius:14px`, Primär `#30d158` mit `color:#000`  
**Filter Pills:** Weiß/aktiv mit schwarzem Text, dunkelgrau/inaktiv  
**Inputs:** `background:#3a3a3c`, kein Border, `border-radius:12px`

---

## Sicherheit

- Passwörter: SHA-256(username.toLowerCase() + ':' + passwort) mit `s:` Prefix
- Rate Limiting: 5 Fehlversuche → 15 Minuten Sperre (clientseitig via `_loginAttempts`)
- RLS: aktiviert, `allow_all_anon` Policy
- JD Client Secret: nur in Supabase Edge Function, nie im HTML
- PWA: Service Worker + Web Manifest

---

## Wichtige Hinweise für Claude Code

1. **Keine separate Build-Pipeline** – alles in einer HTML-Datei
2. **JS-Rendering** – fast alle UI-Elemente werden per `innerHTML` in JS generiert, CSS-Klassen greifen per Override mit `!important`
3. **Inline Styles** – viele dynamische Elemente haben Inline-Styles im JS; bei Designänderungen beide Stellen anpassen
4. **Supabase Realtime** – Fuhren und GPS-Positionen werden live aktualisiert, nicht pollen
5. **Bio-Sperre** – `isBioBetrieb(betrieb)` prüft gegen `BIO_BETRIEBE` Array, `getSiloBioStatus(siloId)` liefert `'bio'|'konventionell'|null`
6. **Sortentrennung** – `getFuhreKulturKey(f)` gibt `'VERMEHRUNG:Toras'` oder `'Winterweichweizen'` zurück, wird für Silo-Sperre verwendet
7. **Passwort-Check** – immer `verifyPW(pw, stored, username)` verwenden, nicht direkten Hash-Vergleich
8. **GPS** – Abfahrer und Drescher teilen Position alle 30s via `shareUserGPS(userId)`
9. **Wake Lock** – aktiv für Drescher/Abfahrer damit Bildschirm anlässt

---

## SQL-Migrations-Dateien

| Datei | Inhalt |
|-------|--------|
| `ernte2026_supabase_setup.sql` | Initiales Schema + Demo-Daten |
| `ernte2026_shapes_migration.sql` | shapes-Tabelle + 156 Feldpolygone |
| `ernte2026_qualitaet_migration.sql` | gluten/oelgehalt Spalten |
| `ernte2026_vermehrungen.sql` | vermehrungen-Tabelle + sorte-Spalte in fuhren |
| `ernte2026_nachrichten.sql` | nachrichten-Tabelle |
| `ernte2026_lieferungen.sql` | lieferungen-Tabelle |
| `ernte2026_jdlink.sql` | jd_tokens-Tabelle |
| `ernte2026_rls.sql` | Row Level Security |
| `ernte2026_pw_reset.sql` | SHA-256 Passwort-Reset |
| `ernte2026_jd_edge_function.js` | Supabase Edge Function für JD Secret |
