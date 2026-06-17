---
description: Garage system with persistent vehicle deformation and a Valora-themed interface.
---

# vlr_garages — Garages

Store and retrieve vehicles from themed garages, with **vehicle deformation / condition**
persisted between sessions.

| | |
|---|---|
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | oxmysql |

## Features

* Public and job/owned garages.
* Persistent vehicle state, including a deformation system.
* Industrial-luxe NUI.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Stores vehicles & state. |

{% hint style="info" %}
Config and exports reference are being written.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). Import the SQL, then `ensure`
after `ox_lib` and `oxmysql`.

## Configuration

_Detailed config reference coming soon._
