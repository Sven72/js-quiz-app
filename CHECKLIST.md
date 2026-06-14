# CHECKLIST.md — Schritt-für-Schritt-Plan

Jeder Schritt ist eine abgeschlossene Lerneinheit. Hake ab mit [x] wenn fertig.
Claude Code: Warte nach jedem Schritt auf Bestätigung, bevor du weitermachst.
Nach jeder Phase: gemeinsamer Git-Commit (du schreibst die Message, ich erkläre das Warum).

---

## PHASE 0 — Git & GitHub Setup

### Schritt 0.1 — Git initialisieren
**Aufgabe:** Im Projektordner:
```bash
git init
git status
```
Beantworte bevor du weitermachst:
- Was bedeutet die Ausgabe von `git status`?
- Was ist der Unterschied zwischen `git init` und `git clone`?
- Was ist ein "Working Tree"?
- [x] Fertig

---

### Schritt 0.2 — .gitignore schreiben (BEVOR erster Commit)
**Aufgabe:** Schreibe `.gitignore` mit mindestens:
```
node_modules/
.env
uploads/
```
**Warum das die ERSTE Datei ist, die du schreibst:**
Wenn du `.env` einmal committet hast, ist der API-Key in der Git-History —
selbst wenn du die Datei danach löschst. Das ist nicht rückgängig zu machen ohne
History-Rewrite. Frage die du beantworten sollst: Warum reicht es nicht, `.env`
einfach später zu löschen und neu zu committen?
- [x] Fertig

---

### Schritt 0.3 — Erster Commit
**Aufgabe:** Stage alle Dateien und mache deinen ersten Commit:
```bash
git add .
git status     # was siehst du jetzt?
git commit -m "???"
```
Schreibe selbst eine Commit-Message. Ich erkläre danach, was einen guten
Commit-Message-Stil ausmacht (Imperativ, max. 72 Zeichen, Warum nicht Was).
- [x] Fertig

---

### Schritt 0.4 — GitHub-Repository anlegen und verbinden
**Aufgabe:**
1. Lege auf github.com ein neues leeres Repository an (kein README, keine License)
2. Verbinde es mit deinem lokalen Repo:
```bash
git remote add origin https://github.com/DEIN-USERNAME/js-quiz-app.git
git branch -M main
git push -u origin main
```
**Lernziel:** Was bedeutet `origin`? Was macht `-u`? Was ist der Unterschied zwischen
`main` und `master`?
- [x] Fertig

---

## PHASE 1 — Projekt-Fundament

### Schritt 1.1 — Ordnerstruktur anlegen
**Aufgabe:** Erstelle die Ordnerstruktur manuell im Terminal (mkdir, touch).
```
js-quiz-app/
├── server.js
├── .env
├── data/
│   └── lernlog.json
├── uploads/
│   └── .gitkeep
└── public/
    ├── index.html
    ├── style.css
    └── app.js
```
Warum braucht `uploads/` eine `.gitkeep`-Datei?
- [ ] Fertig

---

### Schritt 1.2 — package.json
**Aufgabe:** `npm init -y`, dann öffne die Datei und beantworte:
- Was bedeutet `"main": "index.js"` — und warum ändern wir das nicht (wir nutzen `scripts`)?
- Was ist der Unterschied zwischen `dependencies` und `devDependencies`?

Füge ein: `"start": "node server.js"` und `"dev": "node --watch server.js"`
Was macht `--watch`? Warum ist das praktisch?
- [ ] Fertig

---

### Schritt 1.3 — Dependencies installieren
**Aufgabe:**
```bash
npm install express dotenv multer
```
Danach: Erkläre in einem Satz, was jedes der drei Pakete tut.
Was ist `node_modules/` — und warum ignorieren wir es im Git?
- [ ] Fertig

---

### Schritt 1.4 — lernlog.json Grundstruktur
**Aufgabe:** Schreibe die initiale Datenstruktur (alle 8 Themen aus GOAL.md).
Jedes Thema braucht: `richtig`, `falsch`, `zuletzt`, `kontext`.
Plus zwei Top-Level-Felder: `letzter_upload` und `upload_kontext`.

Denk nach: Warum ist ein Objekt `{ "schleifen": {...} }` besser als ein
Array `[{ "name": "schleifen", ... }]` für unseren Anwendungsfall?
- [ ] Fertig

---

### Phase-1-Commit
**Aufgabe:** Committe den aktuellen Stand.
```bash
git add .
git status    # prüfe was gestaged wird
git commit -m "???"
```
Schreibe selbst eine aussagekräftige Message. Dann pushen:
```bash
git push
```
- [ ] Fertig

---

## PHASE 2 — Der Server

### Schritt 2.1 — Express-Grundgerüst
**Aufgabe:** Minimaler Express-Server in `server.js`:
- `dotenv` konfigurieren (erste Zeile!)
- `express` importieren und App-Instanz erstellen
- `express.json()` als Middleware
- `express.static('public')` für das Frontend
- Server auf Port 3000 starten

Teste: `npm run dev` → `localhost:3000` im Browser.
Frage: Warum muss `dotenv.config()` als ERSTE Zeile stehen?
- [ ] Fertig

---

### Schritt 2.2 — GET /api/progress
**Aufgabe:** Endpunkt, der `lernlog.json` liest und als JSON zurückgibt.
Nutze `fs.promises.readFile` (nicht die synchrone Version).

Teste mit curl:
```bash
curl http://localhost:3000/api/progress
```
Frage: Was ist der Unterschied zwischen `fs.readFileSync` und `fs.promises.readFile`?
Wann würdest du welche verwenden?
- [ ] Fertig

---

### Schritt 2.3 — POST /api/progress
**Aufgabe:** Endpunkt, der einen aktualisierten Fortschritt entgegennimmt und schreibt.
Wichtig: Erst lesen, dann mergen, dann schreiben — niemals blind überschreiben.

Schreibe eine Hilfsfunktion `mergeLernlog(alt, neu)` — die sollte pure sein (kein Side Effect).
Frage: Was bedeutet "pure function"? Warum ist das hier wichtig?
- [ ] Fertig

---

### Schritt 2.4 — POST /api/upload (File-Upload + Parsing)
Das ist der komplexeste Schritt. Teile ihn in drei Teile:

**Teil A — multer einrichten:**
- `multer` importieren
- Storage-Konfiguration: Dateien landen in `uploads/`, Dateiname bleibt erhalten
- Middleware nur für diesen Endpunkt (nicht global!)
- Teste: Datei hochladen, prüfen ob sie in `uploads/` landet

**Teil B — Datei lesen und an Claude schicken:**
- Hochgeladene Datei mit `fs.promises.readFile` einlesen
- An Claude schicken mit diesem System-Prompt-Rahmen:
```
Analysiere diesen JavaScript-Lernlog. Extrahiere alle behandelten Themen
und gib sie als JSON zurück. Format:
{
  "neue_themen": ["thema1", "thema2"],
  "aktualisierungen": { "thema": "kurze Beschreibung was gelernt wurde" },
  "upload_kontext": "2-3 Satz Zusammenfassung für zukünftige Fragen"
}
Antworte NUR mit dem JSON, keine Backticks.
```
Frage: Warum schicken wir die Datei an Claude statt sie selbst zu parsen?

**Teil C — lernlog.json aktualisieren:**
- Neue Themen hinzufügen (mit Startwerten richtig:0, falsch:0)
- Bestehende Themen mit neuem Kontext anreichern
- `letzter_upload` und `upload_kontext` updaten
- Datei in `uploads/` nach dem Verarbeiten löschen (`fs.promises.unlink`)
- [ ] Fertig

---

### Schritt 2.5 — POST /api/question
**Aufgabe:** Claude generiert eine Frage basierend auf Thema, Fragetyp und Lernkontext.

Der System-Prompt soll den `upload_kontext` aus lernlog.json enthalten:
```
Du bist ein JS-Lernassistent. Kontext aus dem letzten Lernlog: {upload_kontext}

Generiere eine {fragetyp}-Frage zum Thema {thema}.
Antwort NUR als JSON:
{
  "frage": "...",
  "optionen": ["A", "B", "C", "D"],   // nur bei multiple_choice
  "antwort": "...",
  "erklaerung": "...",
  "sokrates_nachfrage": "..."          // Frage für den Fall einer falschen Antwort
}
```
- [ ] Fertig

---

### Phase-2-Commit
```bash
git add .
git commit -m "???"
git push
```
Diesmal: Erkläre in der Commit-Message was du gebaut hast UND warum du `multer`
statt eines manuellen Parsers verwendet hast.
- [ ] Fertig

---

## PHASE 3 — Das Frontend

### Schritt 3.1 — HTML-Grundgerüst
**Aufgabe:** `index.html` mit vier Sektionen:
- `#startscreen` — Themen-Liste + Upload-Bereich + Start-Button
- `#uploadscreen` — Feedback nach dem Upload (kurz sichtbar)
- `#quizscreen` — Frage + Antwort-Bereich
- `#resultscreen` — Ergebnis-Zusammenfassung

**Für den Upload-Bereich:** Ein `<input type="file" accept=".txt">` und ein Button.
Frage: Was macht das `accept`-Attribut? Ist es eine echte Sicherheitsmaßnahme?
- [ ] Fertig

---

### Schritt 3.2 — CSS: Layout und Theme
**Aufgabe:** `style.css` mit:
- Mindestens 6 CSS Custom Properties (Farben, Spacing, Border-Radius)
- Dunkles Theme
- `.hidden`-Klasse zum Ein-/Ausblenden von Screens
- Fortschrittsbalken-Struktur: `<div class="bar"><div class="bar__fill"></div></div>`
- Upload-Zone: visuell klar, mit Hover-State
- Code-Snippets: `<pre><code>` in Monospace mit anderem Hintergrund

Frage: Warum ist `.hidden { display: none }` per CSS besser als
`element.style.display = 'none'` direkt in JS?
- [ ] Fertig

---

### Schritt 3.3 — app.js: Startscreen
**Aufgabe:** Funktion `ladeStartscreen()`, die:
1. `GET /api/progress` aufruft
2. Themen dynamisch ins DOM rendert (forEach über Object.entries())
3. Fortschrittsbalken-Breite berechnet und setzt

Frage: Warum `Object.entries()` statt einer for...in-Schleife hier?
- [ ] Fertig

---

### Schritt 3.4 — app.js: Upload-Logik
**Aufgabe:** Upload-Handler, der:
1. Die ausgewählte .txt-Datei als `FormData` packt
2. `POST /api/upload` schickt
3. Erfolgsmeldung zeigt: "X neue Themen erkannt"
4. Startscreen neu lädt (damit neue Themen erscheinen)

Frage: Was ist `FormData`? Warum kannst du hier nicht einfach `JSON.stringify` nutzen?
- [ ] Fertig

---

### Schritt 3.5 — app.js: Quiz-Flow
**Aufgabe:** Implementiere den kompletten Quiz-Ablauf:
1. "Quiz starten" → Screen wechseln → erste Frage laden
2. Multiple-Choice: Buttons rendern, Klick auswerten
3. Code-Challenge: `<textarea>` für Eingabe, Submit-Button
4. Erklär-Modus: Freitext, Submit → Claude bewertet
5. Nach Antwort: Feedback anzeigen, bei Falsch sokratische Nachfrage
6. Nach 5 Fragen: Resultscreen

Teile das in Unter-Aufgaben auf — fang mit Punkt 1 an und zeig mir, bevor du
zu Punkt 2 gehst.
- [ ] Fertig

---

### Schritt 3.6 — app.js: Fortschritt speichern
**Aufgabe:** Nach jeder beantworteten Frage:
- `POST /api/progress` mit dem aktualisierten Thema-Stand

Schreibe eine Funktion `aktualisiereThema(thema, richtig)` —
sie soll nur die Daten vorbereiten, der fetch-Call passiert separat.
Frage: Warum ist es besser, Datenvorbereitung und Netzwerk-Call zu trennen?
- [ ] Fertig

---

### Phase-3-Commit
```bash
git add .
git commit -m "???"
git push
```
- [ ] Fertig

---

## PHASE 4 — Fehlerbehandlung & Feinschliff

### Schritt 4.1 — Fehlerbehandlung Server
**Aufgabe:** Was passiert wenn:
- `lernlog.json` nicht existiert (erster Start)?
- Die hochgeladene Datei kein Text ist?
- Claude antwortet mit ungültigem JSON?

Schreibe für jeden Fall einen try/catch mit sinnvollem Fallback.
- [ ] Fertig

---

### Schritt 4.2 — Fehlerbehandlung Frontend
**Aufgabe:** Was passiert wenn:
- Der Server nicht läuft?
- Ein fetch-Call einen 500-Status zurückgibt?
- Der Upload zu groß ist?

Zeige dem User eine verständliche Fehlermeldung (kein "undefined" im UI).
- [ ] Fertig

---

### Schritt 4.3 — System-Prompt Qualität
**Aufgabe:** Teste alle drei Fragetypen × alle 8 Themen = 24 Kombinationen.
Dokumentiere welche Kombinationen schlechte Fragen produzieren.
Passe den System-Prompt an bis die Qualität konstant gut ist.
- [ ] Fertig

---

### Schritt 4.4 — Abschluss-Review
**Aufgabe:** Gehe durch die "Definition of Done" in GOAL.md.
Teste jeden Punkt manuell, inklusive:
- Lade eine neue .txt-Datei hoch → prüfe lernlog.json danach
- Prüfe `git log --oneline` — sind alle Commit-Messages aussagekräftig?
- Prüfe `git show HEAD:.gitignore` — ist .env wirklich drin?
- [ ] Fertig

---

### Abschluss-Commit & Tag
```bash
git add .
git commit -m "???"
git tag v1.0.0
git push --tags
```
Was ist ein Git-Tag? Wann und warum verwendet man ihn?
- [ ] Fertig

---

## Start-Befehl für Claude Code

Lege diese beiden Dateien (GOAL.md und CHECKLIST.md) in deinen Projektordner.
Dann starte Claude Code im Terminal:

```bash
cd js-quiz-app
claude
```

Erster Prompt an Claude Code:

> "Lies GOAL.md und CHECKLIST.md vollständig. Du kennst jetzt mein Projekt,
> meinen Lernstand und wie du mich begleiten sollst. Fang mit Schritt 0.1 an.
> Erkläre mir die Aufgabe — aber schreib keinen Code für mich.
> Stelle sokratische Fragen, wenn ich etwas falsch mache."
