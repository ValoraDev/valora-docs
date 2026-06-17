---
description: Admin panel for server-synchronized map blips, with a live preview and sprite catalog.
---

# vlr_blips — Admin Blips

An admin NUI for creating and managing **server-synchronized** map blips: add, edit and
remove blips that every player sees, with a live preview and a sprite catalog.

| | |
|---|---|
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | oxmysql |

## Features

* Server-synced blips shared to all players.
* Admin panel with live preview and sprite/color catalog.
* Access gated by ACE + identifier.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Stores blips. |

{% hint style="info" %}
Config, access setup and exports reference are being written.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). Import the SQL, then `ensure`
after `ox_lib` and `oxmysql`.

## Configuration

_Detailed config reference coming soon._
