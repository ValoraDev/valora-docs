---
description: >-
  Premium standalone UI component pack for FiveM in the Valora industrial-luxe
  design system — notifications, TextUI, progress, skill checks, input & alert
  dialogs, context/list/radial menus. Fully standalone with an optional,
  non-blocking ox_lib compatibility loader. No framework, no database.
---

# vlr_uipack — Valora UI Pack

`vlr_uipack` is a self-contained set of premium interface components for FiveM, drawn in the Valora **industrial-luxe** design system (graphite surfaces, gold accent, condensed display type, mono numerals, clipped corners). It runs **standalone** through its own exports, and can optionally take over the `ox_lib` UI functions of your existing resources via a non-blocking compatibility loader — with stock `ox_lib` UI as the automatic fallback.

| | |
|---|---|
| **Version** | 1.0.6 |
| **Author** | Valora |
| **Framework** | None — fully standalone (no framework detection or dependency) |
| **Storage** | None — transient only (progress props synced via player state bags) |
| **Built on** | Vanilla NUI (no build step); optional `ox_lib` compatibility |

## Features

* **Notifications** — typed (info / success / warning / error), with IDs (replace-in-place), positions, durations and a configurable chime. Bounded queues protect the UI from burst traffic.
* **TextUI** — interaction prompts with a key glyph and heading.
* **Progress** — linear `ProgressBar` and circular `ProgressCircle` with animations, attached props, control locks and cancel/interruption handling.
* **Skill checks** — multi-stage reaction checks with configurable difficulty and key sets.
* **Input dialogs** — mixed forms: text, number, checkbox, select, multi-select, slider, textarea, date, date range, time and color fields (the `color` field honours `hex` / `rgb` / `rgba` formats).
* **Alert / confirmation dialogs** — header, content and labelled confirm/cancel.
* **Context menus** — titles, descriptions, icons, metadata rows, progress, submenus and `onSelect` args.
* **List menus** — keyboard-first navigation with values (cycle), toggles and per-row progress.
* **Radial menu** — a global radial with nested submenus, key-mapped open.
* **One accent recolors the pack** — change `Config.Theme.accent` and `opacity`; no UI files touched, no build step.
* **CEF-safe** — no `backdrop-filter`; depth comes from gradients/shadows.
* **Optional in-game showcase** — `/vlrui` demo command (off by default, ACE-gated).
* **Localization** — ships in **English**, fully translatable via the open `locales/` files. More languages are planned over time. Config-driven and independent of `ox:locale`.
* **Server & client notification exports** — push a notification to a specific player from the server.

## Dependencies

`vlr_uipack` declares **no `dependencies` in its manifest** — it is standalone. `ox_lib` is only relevant if you use the optional compatibility loader.

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ⬚ Optional | Only when using the compatibility loader so `lib.notify`, `lib.progressBar`, `lib.inputDialog`, `lib.registerContext`, `lib.addRadialItem`, etc. route through Valora UI. Standalone exports need none of it. |
| OneSync | ⬚ Optional | Required only for synchronized progress **props** (attached objects replicate via state bags). |

{% hint style="info" %}
There is **no framework and no database dependency**. The pack works on a bare server with just its own exports. See [Dependencies](../getting-started/dependencies.md) for general setup of the optional pieces.
{% endhint %}

## Installation

### Standalone (recommended baseline)

1. Copy `vlr_uipack` into `resources/` (we recommend `resources/[valora]/vlr_uipack`).
2. Start it **before** any resource that calls its exports:

```cfg
ensure vlr_uipack
```

3. Configure theme, locale and keys in `config.lua`.
4. Call the exports documented in [Developer API](#developer-api).

There is **no SQL to import** and **no items to register** — the pack stores nothing.

### Optional: ox_lib / QBox compatibility

The compatibility loader replaces only the **UI functions** on each resource's `lib` table; every other `ox_lib` feature is untouched. It is non-blocking and falls back to stock `ox_lib` UI if `vlr_uipack` is stopped or ordered wrong.

**Global setup (recommended):** append the contents of `install/ox_lib_hook.lua` to the end of `ox_lib/init.lua` (remove any old UI override hook first), then use this start order:

```cfg
ensure ox_lib
ensure vlr_uipack
ensure qbx_core
ensure ox_target
ensure ox_inventory
```

Resources started after `vlr_uipack` that already import `@ox_lib/init.lua` automatically receive the Valora UI functions.

**Per-resource setup:** instead of the global hook, a single resource can load both files:

```lua
shared_scripts {
    '@ox_lib/init.lua',
    '@vlr_uipack/init.lua',
}
```

For complete notification routing (catching direct `ox_lib:notify` events used by QBox server exports), also copy `install/ox_lib_notify.lua` over `ox_lib/resource/interface/client/notify.lua`. Individual compatibility groups can be toggled in `Config.Compatibility.components`.

{% hint style="warning" %}
If you are migrating from a previous ox_lib UI override: remove any old UI-pack `config_init.lua` references and the old loader block from the end of `ox_lib/init.lua`. Some older blocks use a `repeat … until <ui_pack> is started` loop that blocks every resource importing ox_lib whenever that pack is stopped — the Valora hook never waits and never throws.
{% endhint %}

### Optional: ox_target config bridge

If your customized `ox_target` previously relied on a separate UI pack only for `getConfigValue`, copy `install/ox_target_valora_config.lua` to `ox_target/client/valora_config.lua` and load it before `client/main.lua`. It ships Valora defaults and does **not** create a hard dependency.

## Database

`vlr_uipack` **uses no database**. It does not call oxmysql and writes no KVP. The only persisted-per-session data is the **progress prop list**, replicated through a player **state bag** (`vlr:progressProps`) so other clients can render attached props during synchronized progress — this is transient and cleared on player drop.

## Configuration

Everything lives in the open `config.lua`.

### General

| Key | Meaning |
|---|---|
| `Config.Locale` | Active locale code (default `'en'`); add more `locales/<code>.json` files to translate. Independent of `ox:locale`. |
| `Config.Debug` | Gate for debug prints. |

### Theme — `Config.Theme`

`accent` (gold hex, default `#d4af37`), `brand` (kicker text, default `VALORA`), `audio` (enable/disable UI sound), `reduceMotion`, `opacity` (panel transparency, default `0.90`). One accent recolors the whole pack.

### Notifications — `Config.Notify`

`position`, `forcePosition` (force ox_lib/QBox notifications into the configured Valora position), `duration`, `maxVisible`, `maxQueued`, `burstPerFrame`, `soundCooldown`, and `sound` (`{ name, set }` frontend chime; alternatives are listed inline in the config).

### Interaction & components

| Key | Meaning |
|---|---|
| `Config.TextUI` | `position`, `defaultKey`, `heading`. |
| `Config.Progress` | `position`, `cancelKey`, `maxProps` (max attached props). |
| `Config.Radial` | `key` (key mapping), `openMode` (`'press'` or `'hold'`). |
| `Config.Menu` | `position` of side menus. |

### Demo showcase — `Config.Demo`

`enabled` (off by default), `command` (default `vlrui`), `ace` (ACE permission, default `vlr.uipack.demo`).

### Compatibility — `Config.Compatibility`

`oxLib` master switch plus a per-component map (`components.notify`, `textUI`, `progress`, `skillCheck`, `inputDialog`, `alertDialog`, `contextMenu`, `listMenu`, `radial`) so any single override can be left to another resource.

## Commands & key bindings

| Command / binding | Default key | Description |
|---|---|---|
| `/vlrui` | — | Opens the in-game component showcase. Only registered when `Config.Demo.enabled = true`; ACE-gated by `Config.Demo.ace`. |
| Cancel progress | `Config.Progress.cancelKey` (`X`) | Key-mapped (`+vlr_cancel_progress`) — cancels an active cancelable progress action. |
| Open radial | `Config.Radial.key` (`Z`) | Key-mapped (`+vlr_radial`) — opens the global radial menu. |

The cancel and radial bindings are registered through `RegisterKeyMapping`, so players can rebind them in the FiveM settings. `/vlrui` cannot be run from the server console.

## Developer API

Every method is exported in both **PascalCase** and a matching **lowerCamelCase** name (e.g. `Notify` and `notify`). Unless noted, exports are **client-side**.

### Notifications

```lua
-- client
exports.vlr_uipack:Notify({
    id = 'garage_status',   -- optional: replaces an existing toast with the same id
    title = 'Garage',
    description = 'Vehicle stored.',
    type = 'success',       -- info | success | warning | error
    duration = 4500,
    position = 'top-right',
    icon = 'car'
})

-- server: push to a specific player
exports.vlr_uipack:Notify(playerId, { title = 'Admin', description = 'Report accepted.', type = 'success' })
```

The server export is `Notify(target, data)` / `notify(target, data)`; it forwards to the client over `vlr_uipack:client:notify`.

### TextUI

```lua
exports.vlr_uipack:ShowTextUI('Open garage', { key = 'E', heading = 'PILLBOX GARAGE', position = 'left-center' })
exports.vlr_uipack:HideTextUI()
local open, text = exports.vlr_uipack:IsTextUIOpen()
```

### Progress

```lua
local complete = exports.vlr_uipack:ProgressBar({ label = 'Repairing engine', duration = 5000, canCancel = true,
    disable = { move = true, combat = true }, anim = {...}, prop = {...} })
local complete = exports.vlr_uipack:ProgressCircle(data)
exports.vlr_uipack:CancelProgress()
local active = exports.vlr_uipack:ProgressActive()
```

### Skill check

```lua
local ok = exports.vlr_uipack:SkillCheck({ 'easy', 'medium', { areaSize = 12, speedMultiplier = 1.5 } }, { 'e', 'q', 'r' }, { label = 'Ignition bypass' })
local active = exports.vlr_uipack:SkillCheckActive()
exports.vlr_uipack:CancelSkillCheck()
```

### Dialogs

```lua
local result = exports.vlr_uipack:InputDialog('Create profile', fields, options)
exports.vlr_uipack:CloseInputDialog()

local result = exports.vlr_uipack:AlertDialog({ header = 'Delete', content = 'Cannot be undone.', cancel = true,
    labels = { cancel = 'Keep', confirm = 'Delete' } }) -- returns 'confirm', 'cancel' or nil
exports.vlr_uipack:CloseAlertDialog()
```

### Context menus

```lua
exports.vlr_uipack:RegisterContext({ id = 'vehicle_actions', title = 'Vehicle actions', options = {...} })
exports.vlr_uipack:ShowContext('vehicle_actions')
exports.vlr_uipack:HideContext()
local id = exports.vlr_uipack:GetOpenContextMenu()
```

### List menus

```lua
exports.vlr_uipack:RegisterMenu({ id = 'workshop', title = 'Workshop', options = {...} }, function(selected, scrollIndex, args, checked) end)
exports.vlr_uipack:ShowMenu('workshop')
exports.vlr_uipack:HideMenu()
exports.vlr_uipack:SetMenuOptions(id, options)
local id = exports.vlr_uipack:GetOpenMenu()
```

### Radial menu

```lua
exports.vlr_uipack:AddRadialItem({ {...}, {...} })
exports.vlr_uipack:RegisterRadial({ id = 'business_menu', items = {...} })
exports.vlr_uipack:RemoveRadialItem(id)
exports.vlr_uipack:ClearRadialItems()
exports.vlr_uipack:HideRadial()
exports.vlr_uipack:DisableRadial(state)
local id = exports.vlr_uipack:GetCurrentRadialId()
```

### Events

| Event | Direction | Purpose |
|---|---|---|
| `vlr_uipack:client:notify` | server → client | Deliver a notification (used by the server `Notify` export). |
| `vlr_uipack:client:alert` | → client | Open an alert dialog from an event. |
| `vlr_uipack:client:openDemo` | server → client | Open the showcase context (demo flow). |
| `vlr_uipack:server:setProgressProps` | client → server | Set the player's progress-prop list (capped at `Config.Progress.maxProps`); replicated via the `vlr:progressProps` state bag. |

{% hint style="info" %}
Most integrations should call the **exports**, not these events. The events are listed for awareness and for the few server-driven flows (server notify, demo).
{% endhint %}

## Troubleshooting

| Symptom | Fix |
|---|---|
| Exports return `nil` / errors | `vlr_uipack` is started **after** the calling resource. Ensure it earlier in `server.cfg`. |
| `No context menu registered with id` after a restart | Fixed in 1.0.4 — the ox_lib bridge re-pushes cached menu definitions before each show. Make sure you are on ≥ 1.0.4 and have not edited the bridge. |
| ox_lib resources still show stock UI | The global hook was not appended to `ox_lib/init.lua`, or the resource starts **before** `vlr_uipack`. Check the start order and that the relevant `Config.Compatibility.components` flag is `true`. |
| Notifications appear in the wrong position | Set `Config.Notify.forcePosition = true` to pin ox_lib/QBox notifications to the configured Valora position. |
| Progress props not visible to others | Props replicate via OneSync state bags — enable OneSync and keep `Config.Progress.maxProps` ≥ 1. |
| `/vlrui` does nothing | The demo is off by default. Set `Config.Demo.enabled = true` and grant the ACE (`add_ace group.admin vlr.uipack.demo allow`). |
| A ghost panel stays on screen | Fixed in 1.0.5 (Backspace close path) — update to the latest version. |
| Migrating from another UI pack: resources hang on start | Remove the old `repeat … until` loader block from `ox_lib/init.lua` and any old UI-pack `config_init.lua` references. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
