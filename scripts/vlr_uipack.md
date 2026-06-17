---
description: An ox_lib UI drop-in re-skin in the Valora industrial-luxe style.
---

# vlr_uipack — UI Pack

A drop-in re-skin of the **ox_lib** interface (notifications, context menus, dialogs,
progress bars and more) in the Valora industrial-luxe style. No code changes to your other
resources — they keep calling `ox_lib`, they just look like Valora.

| | |
|---|---|
| **Type** | ox_lib UI override (drop-in) |
| **Storage** | none |

## Features

* Restyles ox_lib UI elements to match the Valora design language.
* Drop-in — start it after `ox_lib` and you're done.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | The library it re-skins. |

{% hint style="warning" %}
**Start order matters:** `ensure vlr_uipack` **after** `ox_lib`. The pack re-registers the
UI on show, so it stays applied across resource restarts.
{% endhint %}

## Installation

See [Installation](../getting-started/installation.md). No SQL. `ensure` after `ox_lib`.

## Configuration

_Detailed theming notes coming soon._
