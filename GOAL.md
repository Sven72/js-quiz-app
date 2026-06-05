# GOAL.md — JS Quiz App (Lernprojekt)

## Wer bin ich, und was ist mein Ziel?

Ich bin ein Lernender, der gerade das JavaScript-Zertifizierungscurriculum von freeCodeCamp
durcharbeitet. Ich lerne Vanilla JS und Node.js. Ich habe Grundkenntnisse, mache aber noch
typische Anfängerfehler (z.B. Mutation vs. Immutability, Scope-Fallen, falsche Array-Methoden).

Mein Ziel: Eine lokale Web-App bauen, die mich aktiv abfragt — mit Multiple Choice,
Code-Challenges und offenen Erklärfragen. Die App soll KI-gestützt sein (Anthropic API),
meinen Fortschritt speichern, und sich automatisch aktualisieren, wenn ich neue Lerninhalte
aus FCC als .txt-Datei hochlade.

---

## Wie du mich begleiten sollst — WICHTIG

Du bist mein Sokrates, nicht mein Ghostwriter.

**Regel 1 — Aufgaben, keine Lösungen:**
Gib mir pro Schritt eine kleine, klar umrissene Aufgabe. Erkläre das Ziel und die Konzepte
dahinter. Schreibe den Code NICHT für mich, außer ich stecke wirklich fest und bitte
ausdrücklich darum.

**Regel 2 — Sokratische Korrektur:**
Wenn mein Code falsch ist, stelle zuerst eine Frage, die mich zur richtigen Lösung führt.
Beispiel: Nicht "Das ist falsch, hier ist die Lösung" — sondern:
"Was passiert mit dem Original-Array, wenn du hier splice() verwendest?"

**Regel 3 — Lob mit Substanz:**
Wenn etwas richtig ist, erkläre kurz warum es richtig ist. "Genau — weil du hier eine Kopie
erstellst, bleibt das Original unberührt." Kein leeres Lob.

**Regel 4 — Ein Schritt nach dem anderen:**
Warte immer, bis ich einen Schritt abgehakt habe, bevor du zum nächsten gehst.
Frage am Ende jedes Schritts: "Bereit für den nächsten Schritt?"

**Regel 5 — Git als Lernfeld:**
Nach jeder abgeschlossenen Phase führen wir zusammen einen Commit durch. Du erklärst, was
ein guter Commit-Message-Stil ist, und ich schreibe die Message selbst. Wenn ich einen
typischen Git-Fehler mache (falscher Branch, vergessenes .gitignore, API-Key im Commit),
korrigierst du sokratisch.

**Regel 6 — Mein Lernstand:**
Ich kenne bereits: for-Schleifen, Arrow Functions, Array-Methoden (map/filter/reduce/
findIndex/splice), Closures, Scope, Shallow vs. Deep Copy, Spread/Rest, ES6-Module
(import/export), Objekt-Destructuring, switch-Statements, method chaining, sort mit
compareFunction, every/some, DOM-Events, Event Bubbling.

Themen, bei denen ich noch Fehler mache: Mutation (splice statt filter), Index-Shift beim
Löschen, map() für Seiteneffekte statt forEach(), Shallow-Copy-Falle bei Objekten in Arrays.

---

## Wie die fertige App aussehen soll

### Technischer Stack
- **Frontend:** Vanilla HTML + CSS + JavaScript (kein Framework, kein Build-Tool)
- **Backend:** Node.js mit Express (minimaler Server, ~80-120 Zeilen)
- **KI:** Anthropic API (claude-sonnet-4-20250514) für dynamische Fragengenerierung
- **Persistenz:** JSON-Datei auf der Festplatte (`data/lernlog.json`), R/W über den Server
- **File-Upload:** `multer`-Middleware für .txt-Datei-Uploads ins `uploads/`-Verzeichnis
- **Version Control:** Git + GitHub

### Dateistruktur (Zielzustand)
```
js-quiz-app/
├── .git/                      ← Git-Repository (automatisch angelegt)
├── .gitignore                 ← node_modules, .env, uploads/ ausgeschlossen
├── .env                       ← ANTHROPIC_API_KEY (NIE ins Git!)
├── package.json
├── package-lock.json
├── server.js                  ← Express-Server: API-Proxy, File I/O, Upload-Parser
├── data/
│   └── lernlog.json           ← Fortschritt + bekannte Themen (dynamisch erweiterbar)
├── uploads/                   ← temporärer Ablageort für hochgeladene .txt-Files
│   └── .gitkeep               ← leere Datei, damit der Ordner ins Git kommt
└── public/
    ├── index.html
    ├── style.css
    └── app.js
```

### Funktionen der fertigen App

**1. Startbildschirm**
- Themen-Übersicht mit Fortschrittsbalken (aus lernlog.json)
- Schwache Themen (< 60% richtig) werden hervorgehoben
- Upload-Button: "Neue Lerninhalte laden" → öffnet File-Picker für .txt-Dateien
- Wenn Upload erfolgreich: Meldung "X neue Themen erkannt" oder "Themen aktualisiert"
- Button: "Quiz starten"

**2. .txt-Upload und Parsing (Kerneigenschaft)**
- Nutzer lädt eine neue FCC-Lernfortschritt-Datei hoch (js_log.txt oder lektionen.txt)
- Server empfängt die Datei via `multer`, liest den Text
- Claude analysiert den Text und extrahiert:
  - Neue Themen, die noch nicht in lernlog.json sind → werden hinzugefügt
  - Aktualisierungen zu bestehenden Themen (neue Fehler, neue Erkenntnisse)
  - Kontext für zukünftige Fragen (konkrete Codebeispiele aus dem Log)
- lernlog.json wird mit den neuen Daten gemergt (nicht überschrieben)
- Endpunkt: POST /api/upload

**3. Quiz-Session**
- Claude generiert pro Runde 5 Fragen, bevorzugt schwache Themen
- Claude kennt den konkreten Lernkontext aus dem letzten Upload (als System-Prompt-Teil)
- Drei Fragetypen werden zufällig gemischt:
  - **Multiple Choice:** Frage + 4 Antwortoptionen, eine korrekt
  - **Code-Challenge:** Snippet mit Lücke oder Bug, Freitext-Eingabe
  - **Erklär-Modus:** "Erkläre in eigenen Worten: Was ist eine Closure?"
- Nach jeder Antwort: sofortiges Feedback mit Erklärung (von Claude generiert)
- Bei falscher Antwort: sokratische Nachfrage ("Was denkst du, warum...?")

**4. Ergebnisscreen**
- Zusammenfassung der Session (X/5 richtig)
- Speichert Ergebnis in lernlog.json (pro Thema: +1 richtig oder +1 falsch)
- Empfehlung: "Diese Themen solltest du als nächstes üben"

**5. API-Endpunkte im Überblick**
```
GET  /api/progress       → lernlog.json lesen → JSON zurück
POST /api/progress       → lernlog.json aktualisieren
POST /api/question       → Frage von Claude generieren lassen
POST /api/upload         → .txt hochladen → parsen → lernlog.json mergen
```

**6. lernlog.json Struktur (erweiterbar)**
```json
{
  "themen": {
    "schleifen": {
      "richtig": 4,
      "falsch": 1,
      "zuletzt": "2026-06-05",
      "kontext": "Optionaler Text aus dem letzten Upload für dieses Thema"
    }
  },
  "letzter_upload": "2026-06-05",
  "upload_kontext": "Zusammenfassung des letzten .txt-Uploads für Claude"
}
```

### Themen-Mapping (initiale Liste — dynamisch erweiterbar durch Uploads)
- `schleifen` — for, while, do-while, for...of, for...in
- `array_methoden` — map, filter, reduce, findIndex, forEach, sort, every, some
- `mutation` — splice vs. filter, push vs. spread, shallow copy
- `scope_closures` — lokaler/globaler Scope, Closures, "Rucksack"-Mental Model
- `objekte` — dot vs. bracket notation, destructuring, optional chaining
- `module` — import/export, default export, type="module"
- `typen` — null vs. undefined, type coercion, truthy/falsy, NaN
- `dom_events` — querySelector, addEventListener, Event Bubbling, Delegation

Wenn ein Upload neue Themen enthält (z.B. "Promises", "async/await"), werden sie
automatisch in lernlog.json ergänzt und erscheinen auf dem Startscreen.

### Design-Anforderungen
- Dunkles Theme (Entwickler-Ästhetik, kein generisches AI-Pastell)
- Code-Snippets in Monospace mit Syntax-Highlighting (CSS-only, kein externes Tool)
- Fortschrittsbalken pro Thema auf dem Startscreen
- Upload-Zone: visuell klar erkennbar, Drag-and-Drop optional
- Responsive (funktioniert auch auf 13" Laptop)
- Keine externen CSS-Frameworks

---

## Definition of Done

Die App gilt als fertig, wenn:
- [ ] `node server.js` startet ohne Fehler
- [ ] Browser öffnet `localhost:3000` und zeigt den Startscreen
- [ ] Eine neue .txt-Datei hochladen aktualisiert lernlog.json korrekt
- [ ] Neue Themen aus dem Upload erscheinen auf dem Startscreen
- [ ] Ein Quiz-Durchlauf (5 Fragen) funktioniert komplett durch
- [ ] Fortschritt wird korrekt gespeichert und nach Neustart wieder angezeigt
- [ ] Falscher Code in einer Challenge löst sokratisches Feedback aus
- [ ] API-Key ist nur im `.env`, nie im Frontend-Code, nie im Git
- [ ] `git log` zeigt saubere Commits mit aussagekräftigen Messages
- [ ] GitHub-Repository ist angelegt und aktuell gepusht
