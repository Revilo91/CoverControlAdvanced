# Cover Control Advanced

[![HACS Custom][hacs-badge]][hacs-url]
[![Validate][validate-badge]][validate-url]

A Home Assistant custom integration for automated cover/shutter control – configurable via the UI, no YAML automations required.

## Features

- One config entry per room
- Multiple covers per room
- Full shading logic in Python:
  - Day/night handling with configurable shading position
  - Window/door contact detection (multiple sensors per cover)
  - Sun-position based shading via a configured azimuth range, or via a dedicated sun light sensor per cover
  - Five room modes: automatic, always active, active, inactive, closed
  - Optional event switch with its own target position
  - Shading release via binary sensor (`on` = shading allowed, `off` = shading blocked) with a 4-minute off-delay
- Diagnostic sensors per cover: decision reason, live position, azimuth range, contact states

> **Requirements specification:** The authoritative functional spec lives in
> [`docs/ANFORDERUNGEN.md`](docs/ANFORDERUNGEN.md) (German). It documents every
> configuration field, the full decision cascade and the list of open questions.

## Installation via HACS

1. HACS → Integrations → ⋮ → Custom Repositories
2. URL: `https://github.com/revilo91/CoverControlAdvanced`
3. Category: `Integration`
4. Add repository, then install

## HACS Default Store Readiness

This repository is structured for HACS as a custom integration and includes local brand assets for the `covercontroladvanced` domain.

Before submitting it to the default HACS store, complete the remaining GitHub-side requirements:

1. Set a repository description on GitHub.
2. Add repository topics on GitHub, for example `home-assistant`, `hacs`, `integration`, `cover`, `roller-shutter`.
3. Ensure the validation workflow passes on the default branch without ignored HACS checks.
4. Publish a GitHub release after the checks pass.
5. Submit a pull request to `hacs/default` and add this repository alphabetically to the `integration` list.

The integration brand assets are stored in `custom_components/covercontroladvanced/brand/`.

## Manual Installation

```bash
cp -r custom_components/covercontroladvanced \
  /config/custom_components/covercontroladvanced
```

Restart HA.

## Configuration

**Settings → Integrations → + Add → Cover Control**

First pick the room, then configure the room-level settings, then add one or more covers to the room. Existing rooms can be extended later via the integration's configure dialog. Once the room is picked, the shading hysteresis, day/night, event switch, cover and window/door contact pickers are all pre-filtered to entities assigned to that room's area (falling back to the full list when the area has none), and a field with exactly one matching entity is pre-selected.

When at least one room is already configured, the setup wizard offers to copy an existing room's settings as a template — pick it and only the cover and window/door contacts need to be selected for the new room. All copied values remain editable afterwards via the configure dialog.

| Level | Field | Required | Description |
|---|---|---|---|
| Room | Room name | ✅ | Area-based room selection; the friendly area name is stored |
| Room | Shading hysteresis | ✅ | `binary_sensor` with `device_class: light` (`on` = shading allowed, `off` = shading blocked). `off` is delayed by 4 minutes before reevaluation. |
| Room | Day/night mode | ✅ | `input_boolean` (`on` = day) |
| Room | Shading height | ✅ | Shared target position for shading, `0..100 %`, default `20` |
| Room | Event switch | – | `switch.*` used as the shared trigger for shading events |
| Room | Event switch position | – | Target position used while the event switch is active, default `0` |
| Cover | Cover entity | ✅ | The `cover.*` entity to control |
| Cover | Window/door contacts | – | Multiple `binary_sensor.*` with `device_class` `window` or `door` |
| Cover | Sun light sensor | – | `binary_sensor` with `device_class: light`. If set, it replaces the azimuth range entirely. |
| Cover | Sun azimuth start | – | `0..359` degrees |
| Cover | Sun azimuth end | – | `0..359` degrees (`start > end` wraps over `0°`) |

## Decision Logic (Priority)

Evaluated per cover, first match wins:

```
1. Night + window contact configured + any contact open → Shading height
2. Only door contacts configured + any contact open     → Open
3. Room mode = closed                                   → Close
4. Room mode = inactive                                 → Open
5. Event switch active                                  → Event switch position
6. No contact open + night                              → Close
7. Day + (window contact or all closed) + shading       → Shading height
8. Default                                              → Day: Open / Night: Close

shading := (automatic     and hysteresis and sun on window)
        or (always_active and                sun on window)
        or (active)
```

Note that the event switch (5) is evaluated *after* the contacts and room
modes, so it has no effect while a door is open or the room mode is
`closed`/`inactive`.
# System Architecture:

This document describes the hierarchical structure and logical dependencies of the entities for automated roller shutter and blind control (cover control) at the room level.

## Visual Architecture
```mermaid
flowchart TD
    %% Room Level
    Room["🏠 Room"]

    subgraph RoomParams [Room Configuration]
        State["Room Mode Select<br/>(automatic / always_active /<br/>active / inactive / closed)"]
        BaseHeight["Global Shading Height"]
        Hysteresis["Binary Sensor (Shading ON/OFF)"]
    end

    subgraph EventSwitch [Event Control]
        Toggle["Event Switch (ON/OFF)"]
        EvHeight{{"Event-specific Height"}}
    end

    %% Links to Room
    Room --- State
    Room --- BaseHeight
    Room --- Hysteresis
    Room --- Toggle
    Toggle ---> EvHeight

    %% Stacked Covers using 'docs' shape
    CoverStack@{ shape: docs, label: "Covers 1..N" }

    Room ==> CoverStack

    subgraph CoverDetails [Cover Instance Specification]
        direction TB

        %% Separate Azimuth Nodes
        StartAz["Start Azimuth"]
        EndAz["End Azimuth"]

        %% Stacked Contacts with Device Class
        ContactStack@{ shape: docs, label: "Contacts 1..N" }
        DClass{{"device_class:<br/>door / window"}}

        ContactStack --- DClass
    end

    %% Connect the stack to its shared definition
    CoverStack --- StartAz
    CoverStack --- EndAz
    CoverStack --- ContactStack
 ```
## Technical Specification
- **Entity: Room**
    - **Room Mode (Select):** Defines the global operating mode. Valid values: `automatic` (shading when hysteresis *and* sun on window), `always_active` (shading whenever the sun is on the window, hysteresis ignored), `active` (shading regardless of hysteresis and sun), `inactive` (covers open), `closed` (covers closed). Defaults to `automatic` and is restored across restarts.
    - **Hysteresis (Binary Sensor):** Shading release input (`on` = shading , `off` = no shading). When it changes from `on` to `off`, reevaluation is delayed by 4 minutes to avoid rapid toggling.
    - **Shading Height (Value):** The default target position for all covers in the room.
    - **Event Switch:** A specialized toggle that activates a secondary set of height settings, overriding the default room height.
    - **1:N Relationship:** A single Room manages a collection of $N$ associated Covers.
- **Entity: Cover**
    - **Sun Light Sensor:** Optional binary sensor that decides directly whether the sun hits this cover. When set, the azimuth range below is ignored.
    - **Start Azimuth:** The sun's angle at which shading for this specific cover begins.
    - **End Azimuth:** The sun's angle at which shading for this specific cover ends.
    - **1:N Relationship (Contacts):** Every cover is linked to $N$ binary sensors.
    - **Contact Attribution** (`device_class`):
        - *Logic Rule:* Contacts primarily serve as safety or lockout conditions (e.g., "lock-out protection") specific to that individual cover.
        - `window`: Triggers ventilation or prevents closing if open.
        - `door`: Provides lock-out protection to prevent accidental closure while people are outside.

## Entities

Each config entry creates a single device with:

- **Room level:** room mode `select`, shading height sensor, event switch position sensor (only when an event switch is configured)
- **Per cover:** status sensor (the last decision reason as its `state`), sun azimuth start/end, live cover position, and the sun light sensor reference when configured
- **Per contact:** an `open`/`closed` diagnostic sensor

All sensors except the status sensor are marked as diagnostic.

## Developer Setup

This repository ships with a Home Assistant dev platform similar to the ComfoClime project.

### Dev Container (recommended)

1. Open this repository in VS Code.
2. Run `Dev Containers: Reopen in Container`.
3. Wait for setup scripts to finish.
4. Open Home Assistant at `http://localhost:8123`.

Detailed docs: `.devcontainer/README.md`

### Local Linux setup

```bash
bash .devcontainer/setup.sh
bash .devcontainer/start-ha.sh
```

Or with the wrapper:

```bash
bash scripts/start-ha-dev.sh
```

### Linting

```bash
bash scripts/lint.sh
bash scripts/lint.sh --fix
```

[hacs-badge]: https://img.shields.io/badge/HACS-Custom-orange.svg
[hacs-url]: https://hacs.xyz
[validate-badge]: https://github.com/revilo91/CoverControlAdvanced/actions/workflows/validate.yml/badge.svg
[validate-url]: https://github.com/revilo91/CoverControlAdvanced/actions/workflows/validate.yml
