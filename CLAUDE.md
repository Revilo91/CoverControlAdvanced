# CLAUDE.md – CoverControlAdvanced

Arbeitsregeln für Claude Code in diesem Repository.

## Projekt

- **Zweck**: Home-Assistant-Custom-Integration für automatische Rollladen-/Coversteuerung (Beschattung, Tag/Nacht, Fensterkontakte, Raummodi), einrichtbar per UI.
- **Stack**: Python 3.12, Home Assistant Custom Component (`custom_components/covercontroladvanced`), HACS.
- **Start**: `bash scripts/start-ha-dev.sh` (Home Assistant auf :8123, `.devcontainer/`)
- **Tests**: keine Unit-Tests; CI prüft HACS-Validierung, Hassfest und Ruff (`.github/workflows/validate.yml`).
- **Lint/Format**: `bash scripts/lint.sh` (`--fix` zum Korrigieren; Ruff, Konfiguration in `pyproject.toml`)
- **Ordnerstruktur**: `custom_components/covercontroladvanced/` (`controller.py` Entscheidungslogik, `config_flow.py`, `const.py`, `sensor.py`, `select.py`, `translations/`), `docs/ANFORDERUNGEN.md`, `.devcontainer/`, `scripts/`.
- **Besonderheiten / Stolperfallen**: Fachlich maßgeblich ist `docs/ANFORDERUNGEN.md` (Entscheidungskaskade FA-24); bei Verhaltensänderung dort nachziehen. Neue Konfigurationsschlüssel in `const.py` immer auch in Config Flow, Laufzeitlogik und beiden Übersetzungen ergänzen. Domain `covercontroladvanced` muss in Ordnername, `manifest.json` und `DOMAIN` identisch bleiben. Weitere Konventionen: `.github/copilot-instructions.md`.

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
- Wiederkehrende Fehler der KI hier in die Datei eintragen (Abschnitt 10).
- Sessionende: Wurde ich korrigiert oder ist derselbe Fehler zweimal passiert, schlage genau EINE Zeile für Abschnitt 10 vor (Datum, Fehler → Regel). Nur nach meinem OK eintragen, als eigener Commit `docs:`.

## 10. Gelernte Korrekturen (laufend ergänzen)

- _(noch leer)_

## 11. Delegation & Modellwahl (nur Claude Code)

- Hauptagent: plant, delegiert, prüft, spricht mit mir. Umfangreiche oder parallele Arbeit macht ein Subagent.
- Selbst erledigen (kein Subagent): Rückfragen an mich, Pläne, Einzeiler, eine einzelne Datei lesen, Commit, Endkontrolle.
- Parallel nur bei unabhängigen Aufgaben (keine gemeinsamen Dateien).
- Auftrag an Subagenten immer vollständig: Ziel, betroffene Dateien, erwartetes Ergebnisformat, Grenzen aus Abschnitt 2.
- Ergebnis nie ungeprüft übernehmen: Diff lesen, Tests selbst ausführen (Abschnitt 1, Schritt 5).
- Haupt-KI hat immer das letzte Wort: Subagenten ändern nur Dateien und committen nie; den Commit für ihre Änderungen macht die Haupt-KI nach Diff-Prüfung und Tests.

Modellwahl (Aliase `haiku`, `sonnet`, `opus` nutzen, keine Versionsnummern):
- **haiku**: Dateien suchen/lesen, Logs zusammenfassen, Formatierung, Doku-Kleinkram.
- **sonnet** (Standard): Implementieren, Tests schreiben, Refactoring, Code-Review.
- **opus**: Architektur, schwieriges Debugging, Security-Review, oder wenn sonnet nach 3 Versuchen scheitert (Abschnitt 6).
- Im Zweifel eine Stufe niedriger starten, bei Misserfolg hochstufen.
