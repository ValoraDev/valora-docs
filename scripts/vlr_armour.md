---
description: >-
  Advanced ballistic vest, helmet and armour-plate system built on ox_inventory —
  server-authoritative, framework-agnostic, fully themeable.
---

# vlr_armour — Armour System

`vlr_armour` replaces the flat vanilla armour bar with a realistic, item-driven
protection system: **ballistic vests**, **helmets** and consumable **armour plates**
that degrade as they take damage, can be **repaired**, and persist per player. Logic is
fully server-authoritative and the framework is auto-detected.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | oxmysql |

## Features

* **Vests, helmets and plates** as real `ox_inventory` items with their own durability.
* **Damage model** — armour absorbs hits and wears down; plates are consumed; gear can break.
* **Repair** via a repair kit item.
* **Persistence** — equipped armour and its condition survive relog and death rules.
* **Custom NUI** — industrial-luxe interface (open `html/` to re-theme).
* **Server-authoritative** — every equip / use / repair is validated server-side.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library. |
| [ox_inventory](https://github.com/overextended/ox_inventory) | ✅ | Armour pieces are inventory items. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Persists armour state. |

See [Dependencies](../getting-started/dependencies.md) for install links.

## Installation

1. Install the dependencies above and ensure they start first.
2. Place `vlr_armour` in `resources/[valora]/`.
3. **Register the items** in `ox_inventory` — add the vest, helmet, plate and repair-kit
   item definitions (and their images) as described in the resource's `install/` folder.
4. Import the SQL from the `install/` folder if present.
5. Add to `server.cfg`:

```cfg
ensure ox_lib
ensure ox_inventory
ensure oxmysql
ensure vlr_armour
```

6. Configure in `config.lua` (armour values, items, behaviour) — see below.

{% hint style="warning" %}
Armour pieces are `ox_inventory` items. If the items aren't registered in `ox_inventory`,
players won't be able to receive or use them. Do step 3 before going live.
{% endhint %}

## Configuration

All settings live in the open `config.lua`. Translations are in `locales/*.json`
(English `en.json` ships by default). Use these to tune armour point values per piece,
the items used, repair behaviour and death/break rules. Because `config.lua` and
`locales/*.json` are escrow-ignored, you can edit them freely.

## Exports

### Server

#### `GetArmour(source)`

Returns the current armour state for a player.

```lua
local armour = exports.vlr_armour:GetArmour(source)
```

Use this from other resources (e.g. a HUD or a medic script) to read a player's current
protection without touching the client.

## Events

`vlr_armour` exposes net events for the armour lifecycle. The useful ones to integrate
against are:

| Event | Side | Purpose |
|---|---|---|
| `vlr_armour:sync` | server → client | Armour state changed; clients update. |
| `vlr_armour:break` | server | An armour piece broke. |
| `vlr_armour:onDeath` | server | Death handling for armour rules. |

Item-use is wired through `ox_inventory` (the vest / helmet / plate / repair-kit items
trigger their handlers internally), so you normally interact with armour through items and
the `GetArmour` export rather than by raising events yourself.

## Framework integration

The bridge listens for the right player-loaded event on each framework
(`qbx_core:client:onPlayerLoaded`, `QBCore:Client:OnPlayerLoaded`, `ox:playerLoaded`,
`esx:playerLoaded`), so no manual wiring is needed.

## FAQ

**Players don't get the armour item.**
The items must be registered in `ox_inventory` (step 3) and the item names must match
`config.lua`.

**Armour doesn't persist after relog.**
Confirm `oxmysql` is running and the install SQL was imported.

**Can another script read a player's armour?**
Yes — call the server export `GetArmour(source)`.

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
