---
description: >-
  Live, in-game customizer for the Vinewood sign — change its text and colour and
  watch the world update as you type, fine-tune every letter with an in-world ghost
  layout editor, then commit it for everyone. Server-authoritative, admin-gated,
  KVP-persisted, framework-agnostic, with no database dependency.
---

# vlr_vwcustom — Vinewood Sign Customizer

`vlr_vwcustom` turns the iconic **Vinewood sign** into something you can edit live in
the world: type new text, pick a colour and brightness, and the sign updates **as you
type** before you commit. An in-world **ghost layout editor** lets you nudge every
letter's position, rotation and tilt, and drop new letter slots exactly where you aim.
Saving broadcasts the result to every player. All authority lives on the server, the
admin gate is re-checked there, and state persists to resource **KVP** — no database
required.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | Qbox / QBCore / ESX — auto-detected (only used for the admin gate); works standalone |
| **Storage** | Resource KVP (no database, no oxmysql) |
| **Built on** | ox_lib |

## Features

* **Live WYSIWYG preview** — the sign in the world updates locally as you type or pick a
  colour, before you commit. **Save** broadcasts to everyone; closing reverts your local
  preview.
* **In-world ghost layout editor** — every letter is shown as a translucent ghost you can
  nudge: WASD to move (camera-relative), arrows for height/turn, `Ctrl`+arrows to tilt
  (pitch/roll), `SPACE` for the next letter, `G` to move the **whole sign** as a group,
  `Shift` for fast mode, `E` to save, `Backspace` to cancel.
* **Aim-to-add anchor** — *Add letter slot* drops a new letter anchor exactly where you
  look (raycast capture).
* **Alignment** — left / center / right, with spaces, case-insensitive input, and a
  configurable glyph map.
* **Colour** — preset palette + custom hex / colour picker + brightness. Pure white keeps
  the original **emissive** texture (it glows at night); any other colour is a flat swatch,
  visibly dimmer after dark. Optional night-brightness boost.
* **Server-authoritative** — admin-gated (ACE / framework group / identifier allow-list);
  every value is clamped and validated server-side.
* **KVP persistence** — the committed sign (text, colour, brightness, alignment, and any
  layout override) survives restarts without a database, and is synced to all players with
  late-join hydration.
* **Framework-agnostic** — an auto-detecting bridge (qbox > qb > esx) is used **only** for
  the admin permission gate; the sign itself runs standalone.
* **Industrial-luxe NUI** — vanilla JS, no build step, no CDN, CEF-safe, no emoji.
* **Localization** — ships in **English**, fully translatable via the open `locales/` files.
  More languages are planned over time.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | UI, callbacks, notifications, commands. |
| Framework (Qbox / QBCore / ESX) | ⬚ Optional | Only used to resolve `Config.Admin.groups`. Auto-detected; the ACE and identifier gates work standalone. |

{% hint style="info" %}
There is **no database dependency**. State is stored in the resource's own KVP, so there
is no SQL to import and no `oxmysql` requirement.
{% endhint %}

## Installation

### 1. Install the resource

1. Copy `vlr_vwcustom` into `resources/` (we recommend `resources/[valora]/vlr_vwcustom`).
2. Start it **after** `ox_lib` in `server.cfg`:

```cfg
ensure ox_lib
ensure vlr_vwcustom
```

The streamed letter assets (the a–z letter models and the default-map replacement that
removes the original baked letters) ship inside the resource's `stream/` folder and load
automatically — no manual streaming setup is needed.

### 2. Grant admin access

Only admins can open the customizer and commit changes. A player is an admin if they match
any one of the gates in `config.lua` → `Config.Admin` (see [Permissions](#permissions)).
The simplest is the ACE:

```cfg
add_ace group.admin vlr.vwcustom.admin allow
```

### 3. Configure and restart

Open `config.lua` to set the admin gate, the command name, the locale and the colour
presets, then restart the resource.

## Storage

`vlr_vwcustom` does **not** use a database. The committed sign — text, colour, brightness,
alignment, and an optional per-letter layout override from the ghost editor — is written to
the resource's **KVP** under a single key and re-broadcast to every client on change and on
join. There is no SQL table to create and no `oxmysql` dependency.

## Configuration

Everything lives in the open `config.lua`. The key sections:

### General

| Key | Meaning |
|---|---|
| `Config.Locale` | UI / notification language — default `'en'`; add more `locales/<code>.json` files to translate. |
| `Config.Debug` | Gates every diagnostic print. Leave `false` in production. |
| `Config.UI.accent` | Accent colour for the NUI (industrial-luxe gold by default). |

### Admin gate

`Config.Admin` defines who may open the editor and commit changes (re-checked on the
server):

| Key | Meaning |
|---|---|
| `ace` | An ACE permission checked with `IsPlayerAceAllowed` (default `vlr.vwcustom.admin`). |
| `groups` | Framework permission groups (default `{ 'admin', 'god' }`), resolved via the qbox/qb/esx bridge. |
| `identifiers` | A standalone allow-list of raw identifiers (any kind), matched against the player's identifiers. |

### Commands

* `Config.Commands.open` — the chat command that opens the customizer (default
  `vinewood` → `/vinewood`).

### Letters & glyphs

* `Config.Glyphs` — maps an input character to its streamed letter model. The shipped
  assets cover **a–z** (built automatically as `a..z`); add new entries here after
  streaming the matching model to support digits / symbols. Spaces leave a gap in the sign.
* `Config.Letters.object` — `lodDistance` (how far the letters keep drawing — the sign is
  read from across the map) and `freeze` (letters never move once placed).

### Layout & anchors

* `Config.Layout` — `align` (one of `'left' | 'center' | 'right'`, default `'center'`),
  `defaultText` (default `'vinewood'`), and `maxAnchors` (server cap on letter slots when
  an admin adds anchors in the editor, default `16`).
* `Config.Anchors` — the per-letter world transforms (`x, y, z, pitch, roll, yaw`). The
  eight defaults are the real Vinewood positions; the in-world ghost editor writes back to
  these as a KVP override, so you tune them live rather than guessing numbers.

### Colour

`Config.Color` controls the sign's single shared colour (the letters sample one shared
texture, so per-letter colour is not possible with these assets):

| Key | Meaning |
|---|---|
| `default` | Default hex (`#ffffff`). Pure white keeps the original **emissive** texture (glows at night). |
| `defaultBrightness` / `minBrightness` | Brightness multiplier (clamped between `minBrightness` (default `0.20`) and `1.0`); ignored on the emissive-white path. |
| `nightBoost` / `nightHours` / `nightBrightness` | Optionally lift a coloured sign's brightness during night hours (it still will not truly emit light — asset limitation). |
| `presets` | The swatches offered in the UI (`{ name, hex }`). |

### Texture

* `Config.Texture` — `origTxd` / `origTxn`, the texture dictionary/name that the colour
  replace overrides. Do not change unless you re-author the assets.

### Notifications

* `Config.Notify` — the function used for in-game notifications (defaults to `lib.notify`).

## Permissions

A player is treated as an admin if they match **any** of the following (all re-checked
server-side):

* the ACE in `Config.Admin.ace` (default `vlr.vwcustom.admin`), **or**
* a framework permission group listed in `Config.Admin.groups` (resolved via qbox / qb /
  esx), **or**
* an identifier listed in `Config.Admin.identifiers` (standalone allow-list).

The server console and scripted (export) calls are always allowed.

## Commands

| Command | Description |
|---|---|
| `/vinewood` | Open the Vinewood sign customizer (admins only). The name is configurable via `Config.Commands.open`. |

## Usage

1. Run `/vinewood` (admins only) to open the customizer.
2. Type the sign text, pick a colour / brightness / alignment → it previews **live** in the
   world.
3. **Open layout editor** to tune each letter in the world, and **Add letter slot** to aim
   and drop a new anchor. The layout is saved together with the text/colour.
4. **Save & apply** commits everything for every player; closing without saving reverts your
   local preview.

## Developer API

All exports are **server-side**.

### `getSign()` → table

Read the current committed sign:

```lua
local sign = exports.vlr_vwcustom:getSign()
-- sign = { text = 'vinewood', color = '#ffffff', brightness = 1.0, align = 'center' }
```

### `setSign(input)` → boolean

Apply a **partial**, server-validated change and sync it to everyone. Returns `false` if
`input` is not a table.

```lua
exports.vlr_vwcustom:setSign({ text = 'valora', color = '#d4af37' })
-- accepted fields: text, color, brightness, align (each clamped/validated server-side)
```

### `resetSign()` → boolean

Reset the sign to its configured defaults, persist, and sync.

```lua
exports.vlr_vwcustom:resetSign()
```

### Server callbacks (`lib.callback`)

Driven by the client/NUI; listed for integration awareness:

| Callback | Purpose |
|---|---|
| `vlr_vwcustom:getState` | Read-only — the current public sign state (used to render on join). |
| `vlr_vwcustom:admin:open` | Admin-gated — returns the editor payload (state, presets, accent, limits). |
| `vlr_vwcustom:admin:save` | Admin-gated — validate and commit a change. |
| `vlr_vwcustom:admin:reset` | Admin-gated — reset to defaults. |

### Net events

| Event | Direction | Purpose |
|---|---|---|
| `vlr_vwcustom:client:sync` | server → client | Broadcast the current sign state to render it. |
| `vlr_vwcustom:client:open` | server → client | Tell an authorised client to open the editor. |
| `vlr_vwcustom:client:notify` | server → client | Show a localised notification. |

## Troubleshooting

| Symptom | Fix |
|---|---|
| `/vinewood` says you are not allowed | You don't match any admin gate — grant the ACE (`add_ace group.admin vlr.vwcustom.admin allow`) or add your group/identifier in `Config.Admin`. |
| Letters don't appear in the world | Ensure `vlr_vwcustom` started cleanly after `ox_lib` so its streamed assets loaded; set `Config.Debug = true` for load diagnostics. |
| Sign is one colour only / a letter won't recolour | By design — the letters share one texture, so the whole sign is a single colour. |
| Coloured sign looks dim at night | Non-white colours are not emissive. Keep white for the night glow, or enable `Config.Color.nightBoost`. |
| Anchors look slightly off | Tune them once with the in-world ghost editor; the override is saved to KVP permanently. |
| Unsupported characters ignored | Only mapped glyphs (a–z by default) and spaces are kept; add more via `Config.Glyphs` after streaming the model. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
