---
description: >-
  Admin-managed, server-synchronized custom map blips for FiveM. An in-game
  admin panel lets authorized staff create, edit, hide and delete blips that
  are stored in MySQL and pushed live to every connected player — no restart,
  no rejoin. Built on ox_lib, server-authoritative, framework auto-detected.
---

# vlr_blips — Custom Map Blips

`vlr_blips` gives your staff an in-game panel (default `/blips`) to place, style and
manage custom map blips. Every blip is saved to the database and broadcast **live to
all players**, so the map stays in sync without a restart. Authority lives on the
server, the framework is auto-detected, and access is gated by ACE permission and/or
an identifier allow-list.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora Development |
| **Framework** | Qbox / QBCore / ESX — auto-detected (only used for the identifier allow-list and notifications) |
| **Storage** | oxmysql (`vlr_blips` table, self-creating) |
| **Built on** | ox_lib |

## Features

* **In-game admin panel** (`/blips`) — create, edit, hide/show, delete and teleport to
  blips from a single panel in the Valora "industrial-luxe" UI.
* **Live sync** — creates, edits, hide/show and deletes appear instantly on every
  connected player's map. No restart, no rejoin.
* **Persistent** — blips are stored in MySQL and survive restarts; the table is created
  automatically on first start.
* **In-world placement** — aim with the camera and drop the blip exactly where you
  want (scroll to nudge height, Enter to confirm, Backspace/Esc to cancel), or snap to
  your current position.
* **Live preview** — while editing, a real flashing preview blip is dropped on the map
  so you see the actual sprite/colour before saving.
* **Curated pickers + numeric override** — a searchable sprite catalog and GTA colour
  palette, plus numeric fields that accept any sprite id (1–826) or colour id (0–85).
* **Categories** — group blips (general, shops, jobs, government, medical, police,
  leisure, transport, events, illegal) for the admin list and filtering.
* **Hide vs. show** — players only ever receive **enabled** blips; hidden ones stay
  admin-only and are removed from clients live when toggled off.
* **Access control** — ACE permission and/or an identifier allow-list; a player passes
  if **either** check matches. Every mutation is re-checked server-side.
* **i18n** — config-driven locales; Bosnian (`ba`, default) and English (`en`) ship.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | Yes | NUI callbacks, server callback, notifications. |
| [oxmysql](https://github.com/overextended/oxmysql) | Yes | Persists the blip table; self-creates on first start. |
| Framework (Qbox / QBCore / ESX) | Optional | Auto-detected. Only used to resolve a player's identifier for the allow-list and for native notifications. Works standalone with ACE-only access. |

See [Dependencies](../getting-started/dependencies.md) for installing the shared
dependencies once for your whole Valora suite.

## Installation

{% hint style="info" %}
**Plug & play.** No core edits and no manual SQL import — the `vlr_blips` table is
created automatically on first start.
{% endhint %}

### 1. Install the resource

1. Copy `vlr_blips` into `resources/` (we recommend `resources/[valora]/vlr_blips`).
2. Start it **after** its dependencies in `server.cfg`:

```cfg
ensure ox_lib
ensure oxmysql
ensure vlr_blips
```

### 2. Grant access

Use either (or both) of these. A player is authorized if **either** matches.

* **ACE permission** (recommended) — add to your `server.cfg`:

  ```cfg
  add_ace group.admin vlr_blips.manage allow
  ```

* **Identifier allow-list** — list citizenids (Qbox/QBCore) or full identifiers (ESX)
  in `config/config.lua → Config.Access.Identifiers`.

### 3. Database (optional)

The `vlr_blips` table **self-creates on first start**. Importing
`sql/vlr_blips.sql` is only needed for manual / air-gapped setups.

## Database

`vlr_blips` uses **oxmysql** and creates a single table on boot. You do not normally
need to import anything.

```sql
CREATE TABLE IF NOT EXISTS `vlr_blips` (
    `id`          VARCHAR(16)  NOT NULL,
    `label`       VARCHAR(64)  NOT NULL DEFAULT 'Blip',
    `category`    VARCHAR(24)  NOT NULL DEFAULT 'general',
    `sprite`      INT          NOT NULL DEFAULT 1,
    `color`       INT          NOT NULL DEFAULT 3,
    `scale`       FLOAT        NOT NULL DEFAULT 0.8,
    `display`     INT          NOT NULL DEFAULT 4,
    `short_range` TINYINT(1)   NOT NULL DEFAULT 1,
    `enabled`     TINYINT(1)   NOT NULL DEFAULT 1,
    `coords`      LONGTEXT     NOT NULL,
    `created_by`  VARCHAR(64)  DEFAULT NULL,
    `created_at`  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

`coords` is stored as JSON (`{ x, y, z }`). All writes are parameterized.

## Configuration

Configuration lives in two open files: `config/config.lua` (behaviour) and
`config/blips.lua` (the pure-data catalogs that drive the panel pickers). Locale strings
live in `locales/`.

### `config/config.lua`

| Key | Default | Meaning |
|---|---|---|
| `Config.Framework` | `'auto'` | `'auto'` picks the first started framework (qbox > qbcore > esx); or force `'qbox'` / `'qbcore'` / `'esx'`. |
| `Config.Notify` | `'auto'` | Notification provider: `'auto'` uses ox_lib when present, else the framework native (`'ox_lib'` / `'qb'` / `'esx'`). |
| `Config.DefaultLocale` | `'ba'` | Active locale (`ba` or `en`). |
| `Config.FallbackLocale` | `'en'` | Used when a key is missing in the active locale. |
| `Config.Access.UseAce` | `true` | Enable ACE-permission access. |
| `Config.Access.AcePerm` | `'vlr_blips.manage'` | The ACE object checked with `IsPlayerAceAllowed`. |
| `Config.Access.Identifiers` | `{}` | Explicit allow-list of citizenids / identifiers. |
| `Config.Command` | `'blips'` | Chat command that opens the panel for authorized players. |
| `Config.OpenKey` | `false` | Optional keybind (e.g. `'F7'`) that requests the panel; `false` disables it. The server still authorizes before the panel opens. |
| `Config.Debug` | `false` | Print extra info to the console. |
| `Config.Defaults` | (table) | Field defaults applied to a new blip in the create form (`sprite`, `color`, `scale`, `display`, `shortRange`, `category`). |
| `Config.ShowDistanceTick` | `false` | Show a small distance label on blips when the player is far away (cosmetic). |
| `Config.NotifyTitle` | `'Blips'` | Title shown on this resource's notifications. |

### `config/blips.lua` — panel data

Pure data that drives the panel's pickers; safe to edit, never touches logic.

* `Config.BlipData.Categories` — the category list for grouping/filtering blips.
* `Config.BlipData.Sprites` — a curated sprite catalog for the picker. The panel also
  exposes a numeric field, so **any** sprite id (1–826) works even if not listed.
* `Config.BlipData.Colors` — the most-used GTA blip colours (swatches are hints; the
  numeric field accepts any colour id 0–85).
* `Config.BlipData.DisplayModes` — where the blip shows (minimap only, main map only,
  both, both + selectable).

{% hint style="warning" %}
A blip's real look is decided by GTA from the integer sprite/colour ids — the swatches
and icons in the panel are only hints. The **in-game live preview is authoritative**:
if a sprite or colour looks different than its tile, trust the preview and type the id
you want directly into the numeric field.
{% endhint %}

### `locales/`

`ba.lua` (default) and `en.lua` hold the notification and placement-prompt strings. Add
more locales by mirroring those files and pointing `Config.DefaultLocale` at the new code.

## Commands

| Command | Description |
|---|---|
| `/blips` | Open the admin panel. Authorized players only; non-admins are silently ignored. The name is configurable via `Config.Command`. |

An **optional keybind** can be enabled via `Config.OpenKey` (e.g. `'F7'`). It registers
a `vlr_blips_open` key mapping that requests the panel; the server still authorizes
before the panel opens, so binding it for everyone is safe.

Inside the panel:

* **List** (left) — every blip with search + category filter; hover a card for teleport
  / hide-show / delete, click to edit.
* **Editor** (right) — name, category, icon, colour, size, display mode, short-range and
  active toggles, and location.
* **Location** — *Mark on map* aims in the world (Enter confirms, Backspace/Esc cancels,
  scroll nudges height); *My position* snaps the blip to you.

## Developer API

{% hint style="info" %}
`vlr_blips` exposes **no public exports**. It is an admin tool, not an integration
library — there is no documented API to read or write blips from another resource.
{% endhint %}

### Server callback (`lib.callback`)

Used internally by the admin panel; listed for integration awareness only.

| Callback | Side | Purpose |
|---|---|---|
| `vlr_blips:admin:list` | server | Returns the **full** blip set (including hidden) to an authorized admin's panel; returns `{}` for everyone else. |

### Net events

Internal protocol between the panel, server cache and player clients. Every server-side
mutation event is guarded by the authorization check; the client form is never trusted.

| Event | Direction | Purpose |
|---|---|---|
| `vlr_blips:admin:requestOpen` | client → server | Authorize, then ask the server to open the panel for the caller. |
| `vlr_blips:admin:open` | server → client | Open the NUI panel. |
| `vlr_blips:admin:create` | client → server | Create a new blip from a sanitized payload. |
| `vlr_blips:admin:update` | client → server | Update an existing blip. |
| `vlr_blips:admin:toggle` | client → server | Hide/show (enable/disable) a blip. |
| `vlr_blips:admin:delete` | client → server | Delete a blip. |
| `vlr_blips:admin:setList` | server → client | Push the full list back to the acting admin after a change. |
| `vlr_blips:requestBootstrap` | client → server | A freshly joined / restarted client asks for the current blips. |
| `vlr_blips:bootstrapPending` | server → client | Cache not ready yet; client should retry. |
| `vlr_blips:sync` | server → client | Full snapshot of **enabled** blips to a client (or all). |
| `vlr_blips:upsert` | server → client | Add/update a single enabled blip live. |
| `vlr_blips:remove` | server → client | Remove a single blip live (also used when a blip is hidden). |

## Security

* **Server-authoritative.** Every create/update/toggle/delete is re-checked with
  `VLR.isAuthorized` (ACE permission or identifier allow-list) on the server — the
  panel being open client-side grants nothing.
* **Untrusted payloads** are sanitized server-side (`VLRUtil.sanitizeBlip`) before they
  reach the cache or the database.
* **Players only receive enabled blips.** The full set (including hidden blips) is only
  ever sent to an authorized admin's panel.
* **Parameterized SQL** throughout; identity is resolved from `source`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `/blips` does nothing | You are not authorized — grant the `vlr_blips.manage` ACE to your group, or add your citizenid/identifier to `Config.Access.Identifiers`. |
| "You are not allowed to manage blips." | Same as above; the command/keybind is authorized server-side. |
| "Blips are still loading, try again in a moment." | The server cache is loading from the database on start — wait a second and retry. |
| Blips don't appear for players | Make sure the blip is **enabled** (not hidden); players only receive enabled blips. |
| A sprite/colour looks different than its swatch | The panel swatches are hints; trust the in-game live preview and type the exact id into the numeric field. |
| Table not created | Ensure `oxmysql` starts before `vlr_blips` and your DB connection is valid; the table self-creates on first start. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
