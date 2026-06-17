---
description: The signature industrial-luxe Valora HUD — multi-skin, car control, custom
  waypoints and a CarPlay-style media panel.
---

# vlr_hud — HUD

The flagship Valora HUD: a vanilla-NUI heads-up display with a **multi-skin engine**, a
car-control panel, an in-world **custom waypoint** marker, and a CarPlay-style media
player.

| | |
|---|---|
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | KVP per player (+ oxmysql for shared media playlists) |

## Features

* **Vitals** — health, armour, hunger, thirst, stress, stamina, plus vehicle telemetry.
* **Multi-skin system** — switchable themes via a single `data-skin` token cascade.
* **Car control panel**, **custom 3D waypoint marker**, and **CarPlay** media (YouTube + playlists).
* Admin can switch and lock a server-wide skin via ACE.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library. |
| [oxmysql](https://github.com/overextended/oxmysql) | ⬚ Optional | Only for shared CarPlay playlists; falls back to KVP. |

{% hint style="info" %}
Full configuration, skin and exports reference is being written. For now, `config.lua`
covers skins, panels and keybinds, and is fully open for theming.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). Ensure after `ox_lib`.

## Configuration

_Detailed config reference coming soon._
