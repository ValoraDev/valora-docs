---
description: Vinewood Sign customizer with live in-world preview and a ghost layout editor.
---

# vlr_vwcustom — Vinewood Sign Customizer

Rewrite the iconic Vinewood Sign in real time: type your text and watch a **live in-world
preview**, then fine-tune each letter with an **in-world ghost layout editor**. Admin-gated
and persisted with KVP (no database).

| | |
|---|---|
| **Framework** | Qbox / QBCore / ESX (auto-detected) |
| **Storage** | KVP (no database) |

## Features

* Live WYSIWYG in-world preview while typing.
* Per-letter ghost editor — position, height, yaw, pitch/roll, group move.
* Raycast "add letter" placement.
* Single shared colour (white = emissive at night).
* Admin gate (ACE / group / identifier); state broadcast to all players incl. late join.
* Exports: `getSign`, `setSign`, `resetSign`.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Core library. Only dependency. |

{% hint style="info" %}
Letters are limited to a–z (streamed assets) and a single shared colour. Full config and
exports reference is being written. Command: `/vinewood`.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). No SQL needed. `ensure` after
`ox_lib`.

## Configuration

_Detailed config reference coming soon._
