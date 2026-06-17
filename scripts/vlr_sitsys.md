---
description: Advanced sit system with per-model seating profiles and synced multi-seat benches.
---

# vlr_sitsys — Sit System

An advanced, server-authoritative sit system: per-model seating **profiles**, synced
**multi-seat benches** (no two players on the same spot), placeable custom props, and an
in-game ghost editor.

| | |
|---|---|
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | oxmysql |

## Features

* Per-model seating profiles (config defaults + DB overrides).
* Server-authoritative seat occupancy on multi-seat props.
* NPC-aware (won't seat you on top of a world ped).
* In-game `/sitadmin` panel: raycast capture + ghost editor.
* Placeable, synced custom props with a catalog.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library. |
| [ox_target](https://github.com/overextended/ox_target) | ✅ | Sit / stand interaction. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Profiles, props & occupancy. |

{% hint style="info" %}
Config, the admin/ghost-editor walkthrough and exports are being written. Command:
`/sitadmin`.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). Import the SQL, then `ensure`
after `ox_lib`, `ox_target` and `oxmysql`.

## Configuration

_Detailed config reference coming soon._
