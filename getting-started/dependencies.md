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

## Framework

Valora auto-detects your framework — you do **not** configure it. The bridge resolves in
this order:

1. **qbx_core** (Qbox)
2. **qb-core** (QBCore)
3. **es_extended** (ESX)

If none is found, scripts fall back to standalone behaviour where possible.

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
