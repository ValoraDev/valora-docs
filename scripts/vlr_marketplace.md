---
description: A player-to-player marketplace with a full tax lifecycle.
---

# vlr_marketplace — Player Marketplace

A player-driven marketplace where players list, browse and buy items from each other,
with a configurable **tax lifecycle** feeding the server economy.

| | |
|---|---|
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | oxmysql |

## Features

* Player listings with search and categories.
* Server-authoritative purchases (money & items via framework APIs).
* Configurable listing fees / sales tax.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Stores listings & transactions. |

{% hint style="info" %}
Config, exports and the tax-lifecycle reference are being written.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). Import the SQL, then `ensure`
after `ox_lib` and `oxmysql`.

## Configuration

_Detailed config reference coming soon._
