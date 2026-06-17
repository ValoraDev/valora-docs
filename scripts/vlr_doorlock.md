---
description: Server-authoritative doorlock with admin, keypad, biometric and detector modes.
---

# vlr_doorlock — Doorlock

A server-authoritative doorlock system with a Valora NUI and multiple access modes:
**admin** management, **keypad** codes, **biometric** access and a door **detector**.

| | |
|---|---|
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | oxmysql |

## Features

* Server-authoritative lock state (the client is never trusted).
* Admin door placement & management UI.
* Keypad, biometric and detector access modes.
* Supports hinged and sliding doors.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Stores doors & permissions. |

{% hint style="info" %}
Config and admin-UI reference are being written.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). Import the SQL, then `ensure`
after `ox_lib` and `oxmysql`.

## Configuration

_Detailed config reference coming soon._
