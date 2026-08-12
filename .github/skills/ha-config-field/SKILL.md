---
name: ha-config-field
description: 'Use when adding, renaming, or removing a config entry / config flow field in the Cover Control Advanced Home Assistant integration (const.py, config_flow.py, controller.py, sensor.py, select.py, translations). Ensures the config key stays consistent across the whole contract and both translation files, and that validation passes.'
---

# Config-Flow-Feld hinzufügen/ändern/entfernen

## Wann verwenden
- Ein neues Feld (Selector, Optional/Required-Wert) wird zum Config Flow einer Room- oder Cover-Konfiguration hinzugefügt.
- Ein bestehender `CONF_*`-Schlüssel wird umbenannt oder entfernt.
- Fachlogik in `controller.py` benötigt einen neuen gespeicherten Wert aus dem Config Entry.

## Kernvertrag (immer alle Stellen prüfen)
1. **`const.py`** – `CONF_*`-Konstante definieren/umbenennen/entfernen. Bei Auswahllisten (wie `ROOM_MODES`) auch die Liste pflegen.
2. **`config_flow.py`** – Schema-Eintrag (`vol.Required`/`vol.Optional`) in `_ROOM_SCHEMA` bzw. Cover-Schema hinzufügen/anpassen, inkl. passendem `selector.*`. Import der Konstante aus `const.py` nicht vergessen.
3. **`controller.py`** – Lesen/Verwenden des neuen Werts aus dem Config Entry (`entry.data`/`entry.options`). Keine blockierenden Aufrufe, weiterhin async.
4. **`sensor.py`** / **`select.py`** – Falls der Wert Auswirkung auf Attribute, `last_reason` oder Entity-State hat, hier nachziehen.
5. **Übersetzungen** – `custom_components/covercontroladvanced/translations/de.json` **und** `en.json` sowie `strings.json` um den neuen Feldnamen/Beschreibung/Abbruchmeldung ergänzen. Beide Sprachdateien müssen inhaltlich synchron bleiben (siehe [translations.instructions.md](../../instructions/translations.instructions.md)).
6. **README.md** – Konfigurationstabelle aktualisieren, falls das Feld nutzersichtbar ist.

## Schritt-für-Schritt
1. Suche alle Vorkommen des betroffenen `CONF_*`-Schlüssels im Repo (`grep`), um keine Stelle zu übersehen.
2. Ändere `const.py` zuerst.
3. Ziehe die Änderung durch `config_flow.py`, `controller.py`, `sensor.py`/`select.py` in dieser Reihenfolge nach.
4. Aktualisiere `de.json`, `en.json`, `strings.json` identisch strukturiert.
5. Aktualisiere ggf. die README-Tabelle.
6. Validiere: `./scripts/lint.sh` ausführen (Ruff format + lint). Bei Bedarf `./scripts/lint.sh --fix`.
7. Optional für einen echten Funktionstest: `./scripts/start-ha-dev.sh` starten und den Config Flow in der UI durchklicken, danach `./scripts/stop-ha-dev.sh`.

## Stolperfallen
- Neue Felder ohne Übersetzung führen zu rohen Schlüsseln in der HA-UI statt lesbarem Text.
- `last_reason`-Texte müssen kurz und für Endnutzer verständlich bleiben, auch wenn neue Entscheidungsgründe hinzukommen.
- Home Assistant erwartet in beiden Übersetzungsdateien exakt dieselbe JSON-Struktur (gleiche Schlüssel-Hierarchie).
