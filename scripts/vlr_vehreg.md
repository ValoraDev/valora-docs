---
description: Realistic vehicle registration and technical inspection, with owned business
  centers and an in-game admin panel.
---

# vlr_vehreg — Vehicle Registration

A full registration & **technical inspection** loop: drivers register plates, pass a
periodic inspection performed by NPCs, and registrations expire over time. Optionally,
inspection centers become **owned passive businesses** with a vault and upkeep.

| | |
|---|---|
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | oxmysql |

## Features

* **Registration + technical inspection** with realistic, server-judged vehicle health.
* **NPC stations** (clerk → mechanic → police) via `lib.points` + `ox_target`.
* **Registration expiry** and physical registration document (optional inventory item).
* **Owned business centers** (Phase 2) — vault income, upkeep obligations, open/closed state.
* **Admin panel** `/regadmin` — ghost editor for placement, prices and owner assignment.
* 12+ exports (e.g. `IsPlateRegistered`, `GetMDTVehicleInfo`, `OrderInspection`).

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library + points. |
| [ox_target](https://github.com/overextended/ox_target) | ✅ | NPC interaction. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Stores registrations & centers. |
| nc-banking | ⬚ Optional | Business bank accounts for centers. |
| lb-tablet | ⬚ Optional | Writes registration data to police/mechanic MDT. |
| t1ger_mechanic | ⬚ Optional | Uses part-health for inspection (native fallback otherwise). |

{% hint style="info" %}
This resource is feature-rich; the full exports list, admin-panel walkthrough and the
owned-business economy are being documented. Commands today: `/regcheck`, `/ordreg`,
`/regadmin`.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). Import the SQL, then `ensure`
after `ox_lib`, `ox_target` and `oxmysql`.

## Configuration

_Detailed config and admin-panel reference coming soon._
