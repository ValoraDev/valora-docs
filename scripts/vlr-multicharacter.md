---
description: >-
  Valora character-selection screen for QBox and QBCore — a cinematic single-ped
  preview, server-validated character creation, per-license slot overrides and a
  private routing bucket during selection. Built on ox_lib + oxmysql with an
  industrial-luxe NUI.
---

# vlr-multicharacter — Character Selection

`vlr-multicharacter` replaces the default (and escrowed) character selector with a
maintainable Valora resource. It keeps the familiar Prism-style flow — one cinematic
preview ped, animated camera, full character details — while moving every ownership,
validation and slot decision to the server. It is QBox-first with QBCore compatibility,
auto-detected at start.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | QBox (`qbx_core`) / QBCore (`qb-core`) — auto-detected, QBox preferred |
| **Storage** | oxmysql (`vlr_multicharacter_slots` table, self-creating) + your framework's `players` / `playerskins` tables (read-only) |
| **Built on** | ox_lib + oxmysql |

## Features

* **Multiple character slots** — a configurable default (`4`) up to a hard maximum
  (`8`), with per-license overrides from config or the admin command.
* **Cinematic preview scene** — a single preview ped is swapped between slots inside one
  stable interior, so every model is framed by the identical camera (configurable FOV
  and depth-of-field) without streaming multiple apartment shells.
* **Character details** — each slot shows first/last name, job, cash, bank, date of
  birth, nationality and phone, read live from the framework's `players` table.
* **Server-validated creation** — name length and character rules, age range
  (`minimumAge`/`maximumAge`) and an allow-list of nationalities are all enforced
  server-side; the client cannot bypass them.
* **Ownership-checked load & delete** — loading or deleting a character is verified
  against the player's Rockstar license before anything happens.
* **Private routing bucket** — players are isolated in a per-source bucket during
  selection and first-appearance creation, then returned to bucket `0` on spawn.
* **Appearance adapters** — `illenium-appearance`, `fivem-appearance` and `qb-clothing`
  are supported for preview and first-character customization.
* **Optional spawn-selector integration** — `prism-spawnselector`, `qbx_spawn` or
  `qb-spawn`, or `none` to spawn at the stored/last position.
* **Starter items** — configurable items handed to a brand-new character (via
  ox_inventory when present, otherwise the framework).
* **Loading hand-off** — keeps `vlr_loading` visible until the interior, collision,
  character data and NUI are ready, and can optionally wait for `ox_inventory` to finish
  building the player's inventory.
* **Commands** — `/relog` to return to selection and an ACE-gated `/setcharslots`.
* **i18n** — Bosnian and English locales, config-driven and independent of `ox:locale`.
* **Valora industrial-luxe NUI** — vanilla JS, no build step, sharing the
  `vlr_marketplace` design system.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | Callbacks, commands, notifications, locale. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Slot-overrides table plus reads of the framework `players` / `playerskins` tables. |
| `qbx_core` **or** `qb-core` | ✅ | The character framework — auto-detected (QBox preferred). One of the two must be present. |
| [ox_inventory](https://github.com/overextended/ox_inventory) | ⬚ Optional | Used for starter items and the optional "wait until inventory is ready" login gate. Falls back to the framework when absent. |
| `illenium-appearance` / `fivem-appearance` / `qb-clothing` | ⬚ Optional | Appearance adapter for preview and first-character customization (`illenium-appearance` by default). |
| `prism-spawnselector` / `qbx_spawn` / `qb-spawn` | ⬚ Optional | Spawn selector, only when `Config.SpawnSelector` is enabled. |
| `vlr_loading` | ⬚ Optional | The Valora loading overlay this resource hands off to. |

{% hint style="info" %}
**Do not run two character selectors at once.** Disable `prism-multicharacter`,
`qbx-multicharacter` or any other selector before starting `vlr-multicharacter`.
{% endhint %}

## Installation

### 1. Enable the external character interface (QBox)

In `qbx_core/config/client.lua`, set:

```lua
characters = {
    useExternalCharacters = true,
}
```

Leave the rest of that table untouched — only flip `useExternalCharacters` to `true`.

### 2. Drop in the resource and order it correctly

Copy `vlr-multicharacter` into `resources/` (we recommend
`resources/[valora]/vlr-multicharacter`) and start it **after** the core and its
dependencies:

```cfg
ensure ox_lib
ensure oxmysql
ensure qbx_core
ensure ox_inventory
ensure illenium-appearance
ensure vlr_loading
ensure vlr-multicharacter

add_ace group.admin vlr.multicharacter.admin allow
```

### 3. Disable the old selector

```cfg
# ensure prism-multicharacter
# ensure qbx-multicharacter
```

### 4. Database

The `vlr_multicharacter_slots` table **self-creates on first start**. Importing
`install/vlr_multicharacter.sql` is only needed for a manual setup. This resource also
**reads** your framework's existing `players` and `playerskins` tables — it does not
create or migrate them.

### 5. Test

Restart the server and connect with a test account — the Valora selection screen should
appear in place of the previous selector.

## Database

A single self-creating table stores per-identifier slot overrides:

```sql
CREATE TABLE IF NOT EXISTS `vlr_multicharacter_slots` (
    `identifier` VARCHAR(80) NOT NULL,
    `slots`      TINYINT UNSIGNED NOT NULL DEFAULT 4,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`identifier`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

Everything else (character rows, money, job, appearance) lives in your framework's own
`players` / `playerskins` tables; this resource only reads them.

## Configuration

Everything is in the open `config.lua`.

### General

| Key | Purpose |
|---|---|
| `Config.Debug` | Enable debug prints (otherwise silent). |
| `Config.Locale` | Active locale — `'ba'` or `'en'`. |
| `Config.Framework` | `'auto'`, `'qbox'` or `'qbcore'`. `auto` checks `qbx_core` then `qb-core`. |

### Interface & cursor

* `Config.UI` — `accent` colour, `logoUrl`, `serverName`, `currency` symbol.
* `Config.Cursor` — `nui`, `native`, `centerOnOpen` for the cursor rendered over the NUI.

### Characters & slots

`Config.Characters` controls slot counts and creation rules: `defaultSlots`, `maxSlots`,
`allowDelete`, `minimumAge`, `maximumAge`, `nameMinLength`, `nameMaxLength`, and a
`slotOverrides` table keyed by license. (The `/setcharslots` database value takes
priority over a static override.)

### Login & inventory gate

`Config.Login` — `waitForOxInventory` keeps the loading overlay up until `ox_inventory`
has built the player's inventory; `oxInventoryTimeout` (`0` = wait indefinitely) sets a
fallback.

### Spawning & scene

* `Config.SelectionBucket` — `enabled` and `base` for the private per-player routing bucket.
* `Config.DefaultSpawn` — `vec4` spawn for a freshly created character.
* `Config.SpawnSelector` — `'none'`, `'auto'`, `'prism-spawnselector'`, `'qbx_spawn'` or `'qb-spawn'`.
* `Config.Scene` — the preview interior: hidden-player and ped positions, camera/focus
  vectors, scenario, stream radius/timeout and settle timings.
* `Config.Camera` — `fov`, `nearDof`, `farDof`, `dofStrength`.

### Appearance, items & content

* `Config.Appearance` — `'illenium-appearance'`, `'fivem-appearance'`, `'qb-clothing'` or `'none'`.
* `Config.StarterItems` — list of `{ name, amount }` given to new characters.
* `Config.Nationalities` — the allow-list of selectable nationalities (`value`/`label`).

### Commands & notifications

* `Config.Commands` — per-command `enabled`, `name` and `ace` for `relog` and `setSlots`.
* `Config.Notify` — the notification function (defaults to `lib.notify`).

## Commands

| Command | Access | Description |
|---|---|---|
| `/relog` | Everyone by default (configurable ACE) | Log the player out and return them to character selection. |
| `/setcharslots [id] [slots]` | `vlr.multicharacter.admin` ACE | Set a player's number of character slots (1 … `maxSlots`), persisted to the database. |

Command names and ACE permissions are configurable in `config.lua` → `Config.Commands`.

## Developer API

`vlr-multicharacter` does **not** expose any `exports`. Integration is through ox_lib
callbacks and net events.

### Server callbacks (`lib.callback`)

Invoked by the resource's own client; listed for integration awareness.

| Callback | Purpose |
|---|---|
| `vlr_multicharacter:server:getData` | Return the player's characters, slot count, framework, UI config, nationalities and translations. |
| `vlr_multicharacter:server:loadCharacter` | Validate license ownership and log the chosen character in. |
| `vlr_multicharacter:server:createCharacter` | Sanitize and validate input, claim a free slot and create the character. |
| `vlr_multicharacter:server:deleteCharacter` | Validate ownership and delete the character (QBox delegates to its compatibility event). |

### Net events

| Event | Direction | Purpose |
|---|---|---|
| `vlr_multicharacter:client:open` | server → client | Open the selection NUI (also fired by `/relog`). |
| `vlr_multicharacter:server:enterSelection` | client → server | Put the player into the private selection bucket. |
| `vlr_multicharacter:server:finishCreation` | client → server | Leave the selection bucket once the player is loaded. |

## Troubleshooting

| Symptom | Fix |
|---|---|
| The old selector still shows | Disable `prism-multicharacter` / `qbx-multicharacter`, and set QBox `characters.useExternalCharacters = true`. |
| No characters appear | The player's `players` rows are matched by Rockstar `license`/`license2` — confirm the framework is writing those identifiers. |
| Stuck on the loading overlay | `Config.Login.waitForOxInventory` is on and `ox_inventory` never reported ready — start `ox_inventory` before this resource or set a positive `oxInventoryTimeout`. |
| `/setcharslots` says "not allowed" | Grant the ACE: `add_ace group.admin vlr.multicharacter.admin allow`. |
| Wrong framework detected | Force it with `Config.Framework = 'qbox'` or `'qbcore'` instead of `'auto'`. |
| Preview ped looks wrong / no clothes | Check `Config.Appearance` matches the appearance resource you actually run. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
