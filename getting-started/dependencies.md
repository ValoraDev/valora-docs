# Dependencies

Valora is built on the modern **ox** stack. Install these once and they serve the whole
suite. Each script's page lists exactly which of these it needs.

## Required (almost everywhere)

| Dependency | What it does | Link |
|---|---|---|
| **ox_lib** | Core library — callbacks, notifications, context menus, cache, points. Required by every Valora resource. | [overextended/ox_lib](https://github.com/overextended/ox_lib) |
| **oxmysql** | Database layer. Required by any script that persists data (registrations, garages, marketplace, blips, …). | [overextended/oxmysql](https://github.com/overextended/oxmysql) |

## Common

| Dependency | Used by | Link |
|---|---|---|
| **ox_target** | Interaction targeting (e.g. `vlr_sitsys`, registration NPCs). | [overextended/ox_target](https://github.com/overextended/ox_target) |
| **ox_inventory** | Item handling (e.g. `vlr_armour` vests/plates/kits). | [overextended/ox_inventory](https://github.com/overextended/ox_inventory) |

## Framework compatibility

Valora auto-detects your framework — you do **not** configure it. Each script carries a
**bridge** with a real, separate code path per framework, resolved in this order:

1. **qbx_core** (QBox)
2. **qb-core** (QBCore)
3. **es_extended** (ESX)

Most of the suite runs on **all three**, and several scripts also run **standalone** (no
framework) where that makes sense. Each script's own page states exactly what it needs.

{% hint style="info" %}
**"Built on ox_lib" does not mean "QBox only."** `ox_lib`, `ox_target` and `ox_inventory`
are **standalone, framework-agnostic** libraries — they work identically on ESX, QBCore and
QBox. (QBox depends on `ox_lib`, not the other way around; `ox_inventory` selects your
framework with a single `setr inventory:framework "esx|qb|qbx"` convar.) So a script that
requires `ox_inventory` or `ox_target` is **not** locked to one framework.

QBox is a modern fork of QBCore and keeps a `qb-core` compatibility bridge, so QBCore and
QBox behave almost identically for our scripts — and every Valora script also ships a
**native QBox path**, not just the compatibility shim.
{% endhint %}

The one product that is **framework-specific** is **vlr-multicharacter** (QBox / QBCore
only — never ESX), because a character selector wires directly into the framework's own
character-creation flow and database schema. Its page says so plainly. Scripts that need no
framework at all (e.g. `vlr_uipack`, `vlr_loading`) list Framework as *None / standalone*.

## Optional integrations

Some scripts light up extra features when a partner resource is present, but never
require it:

| Integration | Enhances |
|---|---|
| **nc-banking** (NoxCore Banking) | Society / business bank accounts (registration centers, mechanic). |
| **lb-tablet** | MDT records (vehicle registration data in police/mechanic tablets). |
| **t1ger_mechanic** | Vehicle part-health used by the technical inspection in `vlr_vehreg`. |

{% hint style="warning" %}
Always start dependencies **before** the Valora resource in your `server.cfg`. A script
that can't find `ox_lib` on start will refuse to run.
{% endhint %}
