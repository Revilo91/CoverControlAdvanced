---
name: hacs-compliance
description: 'Use when preparing a release, bumping the integration version, changing manifest.json/hacs.json, or checking whether the Cover Control Advanced repository still satisfies HACS/hassfest requirements. Triggers: "HACS", "Release", "manifest.json", "hacs.json", "version bump", "hassfest", "HACS-Validierung".'
---

# HACS-Konformität sicherstellen

## Wann verwenden
- Vor jedem Release (neuer Git-Tag / GitHub Release).
- Wenn `manifest.json` oder `hacs.json` geändert werden.
- Wenn die CI-Validierung (`.github/workflows/validate.yml`) fehlschlägt oder proaktiv geprüft werden soll.
- Wenn sich die unterstützte Home-Assistant-Mindestversion ändert.

## Prüfliste

1. **`manifest.json`** (`custom_components/covercontroladvanced/manifest.json`)
   - `domain` bleibt `covercontroladvanced`.
   - `version` folgt SemVer und wird bei jedem Release erhöht — muss mit dem GitHub-Release-Tag übereinstimmen (ohne führendes `v`, HACS vergleicht `manifest.version` mit dem Tag).
   - `codeowners`, `documentation`, `issue_tracker` zeigen auf gültige, existierende URLs.
   - `config_flow: true` bleibt gesetzt, solange die Integration ausschließlich über UI konfiguriert wird.
   - `requirements`/`dependencies` nur ändern, wenn tatsächlich neue Python-Pakete oder HA-Integrationen als Abhängigkeit benötigt werden.

2. **`hacs.json`** (Repo-Root)
   - `name` bleibt `Cover Control Advanced` (sichtbarer Produktname, siehe [copilot-instructions.md](../../copilot-instructions.md)).
   - `homeassistant`-Mindestversion nur anheben, wenn tatsächlich neuere HA-APIs genutzt werden; sonst so niedrig wie möglich halten für Kompatibilität.
   - `render_readme: true` beibehalten, damit die HACS-UI die README anzeigt.

3. **Konsistenz Version ↔ Release**
   - Ein neues GitHub-Release-Tag muss exakt der `manifest.json`-Version entsprechen (z. B. Tag `1.1.0` ↔ `"version": "1.1.0"`).
   - Release-Notizen/Checkliste in `docs/RELEASE_CHECKLIST.md` (falls vorhanden) abarbeiten.

4. **Brand-Assets**
   - `custom_components/covercontroladvanced/brand/` muss vorhanden bleiben (Icon/Logo für die lokale Domain), sonst schlägt die HACS-Brand-Prüfung fehl.

5. **Lokale Validierung ausführen**
   - `./scripts/lint.sh` (Ruff format + lint).
   - Hassfest lokal ist optional; verlasse dich primär auf den CI-Job `validate-hassfest` in [.github/workflows/validate.yml](../../workflows/validate.yml).
   - Für einen echten HA-Start: `./scripts/start-ha-dev.sh`, danach `./scripts/stop-ha-dev.sh`.

6. **CI-Erwartung prüfen**
   - Drei Jobs müssen grün sein: `validate-hacs` (HACS Action), `validate-hassfest`, `lint` (Ruff).
   - Bei rotem HACS-Job: meist fehlende/inkonsistente Manifest-Felder, fehlende `codeowners`, oder ungültige `documentation`/`issue_tracker`-URL.
   - Bei rotem Hassfest-Job: meist Schema-Fehler in `manifest.json`, fehlende Übersetzungsschlüssel oder falsche Domain-Konsistenz.

## Stolperfallen
- Version in `manifest.json` vergessen zu erhöhen → HACS zeigt kein Update an, obwohl neuer Tag existiert.
- `domain` in `manifest.json`, Ordnername und `DOMAIN`-Konstante in `const.py` laufen auseinander → Hassfest schlägt fehl.
- README-Installationsanleitung und tatsächlicher Ordnername/HACS-Repo-URL weichen voneinander ab.
