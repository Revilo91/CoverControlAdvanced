# CLAUDE.md – Globale Arbeitsregeln

> Ablage global: `~/.claude/CLAUDE.md` (gilt für alle Projekte).
> Projektspezifisches kommt in `<repo>/CLAUDE.md` (Abschnitt "Projekt" unten als Vorlage).

## 1. Arbeitsablauf (immer in dieser Reihenfolge)
1. **Verstehen**: Bei unklarer Anforderung genau EINE Rückfrage stellen.
2. **Planen**: Bei mehr als 2 Dateien oder neuer Architektur zuerst einen kurzen Plan zeigen und auf OK warten.
3. **Test zuerst**: Erst fehlschlagenden Test schreiben, dann Implementierung (TDD), wo sinnvoll.
4. **Klein umsetzen**: Ein Schritt = ein Commit. Keine Sammel-Änderungen.
5. **Prüfen**: Tests/Linter/Build wirklich ausführen, bevor "fertig" gesagt wird. Ausgabe zeigen.
6. **Zusammenfassen**: 2–3 Zeilen: was geht jetzt, wie ausprobieren, was ist offen.

## 2. Grenzen (nicht ohne Rückfrage)
- Keine Änderungen außerhalb des besprochenen Umfangs (kein "nebenbei" Refactoring).
- Keine neuen Abhängigkeiten ohne Begründung (Standardbibliothek zuerst).
- Keine destruktiven Aktionen: `rm -rf`, `git push --force`, `git reset --hard`, DB-Migrationen, Löschen von Dateien.
- Keine Secrets, Passwörter, Tokens im Code oder in Commits. Immer `.env` / Secret-Store, `.env` in `.gitignore`.
- Keine Netzwerk-/Systemänderungen an Proxmox, Home Assistant oder NAS ohne ausdrückliches OK.

## 3. Code-Qualität
- Klar vor clever. Kleine Funktionen, eine Aufgabe pro Funktion, sprechende Namen.
- Keine Magic Numbers, Konstanten benennen.
- Fehler behandeln, nie still schlucken. Fehlermeldungen mit Ursache + Kontext.
- Kommentare erklären das *Warum*, nicht das *Was*.
- Eingaben von außen immer validieren.
- Kein toter Code, keine auskommentierten Blöcke.

### Python
- Type Hints überall, `pathlib` statt `os.path`, `logging` statt `print`.
- Formatierung/Lint: `ruff` (+ `ruff format`), Typen: `mypy` oder `pyright`.
- Tests: `pytest`. Abhängigkeiten in `pyproject.toml`, virtuelle Umgebung pro Projekt.
- Dataclasses/Pydantic statt losen Dicts.

### C#
- Nullable Reference Types an, `async/await` konsequent, `IDisposable` sauber behandeln.
- Tests: xUnit. Format: `dotnet format`. Analyzer-Warnungen als Fehler.

### C/C++
- C++: RAII, `std::unique_ptr` statt `new/delete`, `const` wo möglich, keine rohen Besitz-Pointer.
- Warnungen an (`-Wall -Wextra`), Warnungen beheben statt unterdrücken.
- Speicher/UB prüfen mit Sanitizern (ASan/UBSan) im Test-Build.

### JavaScript (React/Vite, Node.js/Express)
- `const`/`let`, nie `var`. Strikte Vergleiche (`===`). `async/await` statt Callback-Ketten.
- Lint/Format: ESLint + Prettier. Abhängigkeiten über `package.json` + Lockfile, Node-Version festlegen (`.nvmrc`).
- Tests: Playwright für E2E, Vitest für Unit-Tests (Vite-Projekte).
- React: kleine Komponenten, Hooks-Regeln beachten, Zustand nicht doppelt halten.
- Auth/Routing: Race Conditions beim Laden beachten (Auth-Guard erst nach geladenem Zustand entscheiden).
- Backend: Eingaben validieren, Fehler zentral in Middleware behandeln, nie Roh-Fehler an den Client geben.

### SQL / Datenbank
- Nur parametrisierte Queries. Schema-Änderungen ausschließlich per Migration, nie direkt live.
- Vor Migrationen Backup. Indizes für Fremdschlüssel und häufige Filter.
- Datensätze einmal laden und im Speicher halten, nicht bei jedem Aufruf neu abfragen (sofern Konsistenz nicht leidet).

### Docker / Compose
- Feste Image-Tags statt `latest`. Secrets über `.env`/Secrets, nicht ins Image.
- Volumes für Daten, Healthchecks für Dienste. Änderungen an Produktiv-Containern (Synology) erst nach OK.

## 4. Tests
- Neue Funktion oder Bugfix = mindestens ein Test.
- Bugfix: erst Test, der den Fehler reproduziert, dann Fix.
- Abdecken: Normalfall, Grenzwerte, Fehlerfall, leere/ungültige Eingabe.
- Tests unabhängig voneinander, keine Reihenfolge-Abhängigkeit, keine echten externen Dienste (mocken).

## 5. Git
- Commits nach Conventional Commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`.
- Eine logische Änderung pro Commit, Betreff maximal 72 Zeichen, Imperativ.
- Feature-Arbeit auf eigenem Branch, nie direkt auf `main`.
- Vor Commit: `git diff` prüfen, keine Debug-Reste, keine Secrets.

## 6. Debugging
- Erst reproduzieren, dann Ursache finden, dann fixen. Kein Raten und Herumprobieren.
- Nach 3 erfolglosen Versuchen: stoppen, Annahme benennen, die vermutlich falsch ist, EINE Diagnosefrage stellen.
- Root Cause beheben, nicht das Symptom.

## 7. Sicherheit
- Kein `eval`/`exec` auf externen Eingaben, keine SQL-String-Konkatenation (parametrisierte Queries).
- Abhängigkeiten aktuell halten, bekannte Schwachstellen prüfen.
- Logs ohne Passwörter/Tokens/personenbezogene Daten.

## 8. Home Assistant (wenn relevant)
- Quelle: offizielle Repos/Docs auf GitHub (home-assistant/core, developers.home-assistant.io).
- `entity_id` statt `device_id`. Vor Umbenennungen prüfen, wer die Entität nutzt.
- Native Trigger/Bedingungen/Helper bevorzugen vor Jinja-Templates.
- Konfiguration zuerst als Vorschlag zeigen, nicht direkt live ändern.

## 9. Arbeiten mit KI (Vibe-Coding-Regeln für mich)
- Ich bleibe verantwortlich: jeden KI-Diff lesen, bevor er committet wird.
- Kleine, klar beschriebene Aufgaben statt "bau mir alles".
- Kontext geben: Ziel, Randbedingungen, Beispiel für erwartetes Verhalten.
- Bei langem Chat mit Drift: neue Session starten, Stand in 5 Zeilen zusammenfassen.
- Wiederkehrende Fehler der KI hier in die Datei eintragen (Abschnitt 12).
- Sessionende: Wurde ich korrigiert oder ist derselbe Fehler zweimal passiert, schlage genau EINE Zeile für Abschnitt 12 vor (Datum, Fehler → Regel). Nur nach meinem OK eintragen, als eigener Commit `docs:`.

## 10. Projekt (Vorlage – in `<repo>/CLAUDE.md` ausfüllen)
- **Zweck**: <ein Satz>
- **Stack**: <Sprache, Framework, Version>
- **Start**: `<Befehl>`
- **Tests**: `<Befehl>`
- **Lint/Format**: `<Befehl>`
- **Ordnerstruktur**: <2–5 Zeilen>
- **Besonderheiten / Stolperfallen**: <Liste>

## 11. Gelernte Korrekturen (laufend ergänzen)
- <Datum>: <Was ist schiefgelaufen> → <Regel dagegen>

## 12. Delegation & Modellwahl (nur Claude Code)
- Hauptagent: plant, delegiert, prüft, spricht mit mir. Umfangreiche oder parallele Arbeit macht ein Subagent.
- Selbst erledigen (kein Subagent): Rückfragen an mich, Pläne, Einzeiler, eine einzelne Datei lesen, Commit, Endkontrolle.
- Parallel nur bei unabhängigen Aufgaben (keine gemeinsamen Dateien).
- Auftrag an Subagenten immer vollständig: Ziel, betroffene Dateien, erwartetes Ergebnisformat, Grenzen aus Abschnitt 3.
- Ergebnis nie ungeprüft übernehmen: Diff lesen, Tests selbst ausführen (Abschnitt 2, Schritt 5).

Modellwahl (Aliase `haiku`, `sonnet`, `opus` nutzen, keine Versionsnummern):
- **haiku**: Dateien suchen/lesen, Logs zusammenfassen, Formatierung, Doku-Kleinkram.
- **sonnet** (Standard): Implementieren, Tests schreiben, Refactoring, Code-Review.
- **opus**: Architektur, schwieriges Debugging, Security-Review, oder wenn sonnet nach 3 Versuchen scheitert (Abschnitt 7).
- Im Zweifel eine Stufe niedriger starten, bei Misserfolg hochstufen.
