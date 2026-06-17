---
description: >-
  Server-authoritative FiveM door system with an in-game Valora administration
  panel — single/double and sliding doors, job/gang/character/item/passcode and
  ACE access, biometric Face ID & fingerprint, a placeable metal detector with a
  laser checkpoint, lockpicking, persistence and a security audit history. Built
  on ox_lib + oxmysql with a qbox/qb/esx access bridge.
---

# vlr_doorlock — Door Lock & Security System

`vlr_doorlock` is a server-authoritative door manager. Every door is created and edited
in-game through a Valora administration panel, with a wide range of access rules
(jobs, gangs, characters, items, passcodes, public, ACE), optional biometric Face ID
and fingerprint scanners, a placeable item detector that physically blocks flagged
players behind a laser checkpoint, lockpicking, persistent states, automatic relocking
and a security audit log. All authority lives on the server and the framework is
auto-detected.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | Qbox / QBCore / ESX — auto-detected (only needed for job/gang/admin checks); runs standalone otherwise |
| **Storage** | oxmysql (`vlr_doorlock` + `vlr_doorlock_audit` tables, self-creating) |
| **Built on** | ox_lib + oxmysql |

## Features

* **In-game door editor** — create and edit every door from the `/dooradmin` panel; aim
  at the exact door object to capture it. No coordinate hunting in config.
* **Single & double doors** — capture one or both leaves of a doorway.
* **Sliding / automatic doors** — flagged doors use a fixed-position Valora HUD with `E`
  (alongside `ox_target`), so a door object that has moved can always be closed again.
* **Rich access rules** — combine **jobs**, **gangs**, **character IDs**, **items**,
  **public access**, an **access code (keypad)** and **ACE/admin** on a single door.
  Jobs and gangs use a `name:minimumGrade` format.
* **Biometric verification** — per-door **Face ID** and/or **fingerprint** profiles,
  registered against the player's persistent character identifier and validated on the
  server; a live GTA character portrait renders inside the Face ID scanner. Biometrics
  can both lock and unlock an eligible door.
* **Placeable biometric scanner** — drop a scanner prop in the world; biometric actions
  then move from the door model onto that scanner's `ox_target`.
* **Item security detector** — a placeable metal-detector prop with server-side
  prohibited-item scanning. A laser-barrier checkpoint physically blocks a flagged
  player until the scan passes; clearance is short-lived and consumed on entry. One-way
  or two-way scanning.
* **Lockpicking** — optional ox_lib skill-check lockpicking, gated by configurable
  lockpick items with a per-item break chance.
* **Private-property management** — assign an online player as the property owner and
  add editors, each granted only the editing permissions you choose (PIN, Access,
  Biometrics, Settings).
* **Persistent states & auto-relock** — door state survives restarts (always-locked,
  always-unlocked, or restore-last-state per door); optional auto-lock timer and
  synchronized hold-open behavior.
* **Security audit history** — a searchable activity log of locks, unlocks, denials,
  lockpicks and editor changes, with retention limits; optional Discord webhook.
* **Server-authoritative** — distance, permission and cooldown are all re-validated on
  the server; passcodes and full access rules are never sent in normal player sync.
* **i18n** — config-driven locales (English + Bosnian shipped), independent of `ox:locale`.
* **Developer API** — server exports plus a `vlr_doorlock:stateChanged` event for
  alarms, dispatch, robberies or logging.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | Yes | UI, callbacks, skill-checks, commands, notifications. |
| [oxmysql](https://github.com/overextended/oxmysql) | Yes | Persists doors and the audit log. |
| [ox_target](https://github.com/overextended/ox_target) | Optional | Used for door / scanner / detector interactions when present (a compact HUD fallback exists). |
| [ox_inventory](https://github.com/overextended/ox_inventory) | Optional | Item access rules, lockpick consumption and detector item scanning. `qb-inventory` and ESX inventories are also supported by the bridge. |
| Framework (Qbox / QBCore / ESX) | Optional | Only for job/gang resolution and group-based admin checks. Auto-detected; works standalone with ACE admin. |

{% hint style="info" %}
Only **ox_lib** and **oxmysql** are hard dependencies (`fxmanifest.lua`). Everything else
— `ox_target`, an inventory, and a framework — is auto-detected at runtime and used only
if it is running.
{% endhint %}

## Installation

### 1. Start order

Ensure the hard dependencies start **before** `vlr_doorlock`:

```cfg
ensure ox_lib
ensure oxmysql
ensure vlr_doorlock
```

### 2. Drop in the resource

Copy `vlr_doorlock` into `resources/` (we recommend `resources/[valora]/vlr_doorlock`).

### 3. Database

The `vlr_doorlock` and `vlr_doorlock_audit` tables **self-create on first start**.
Importing `install/vlr_doorlock.sql` is only needed for manual setups or schema review.

### 4. Grant admin access

Give your administrators the ACE permission:

```cfg
add_ace group.admin vlr.doorlock.admin allow
```

Framework groups in `Config.Admin.groups` (`admin`, `god` by default) are also accepted.

### 5. Create doors

In-game, run `/dooradmin`, create a door, and aim at the door object when the editor
asks for a capture.

## Database

Two self-creating tables:

```sql
CREATE TABLE IF NOT EXISTS `vlr_doorlock` (
    `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(80) NOT NULL,
    `data` LONGTEXT NOT NULL,
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uq_vlr_doorlock_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE IF NOT EXISTS `vlr_doorlock_audit` (
    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    `door_id` INT UNSIGNED NULL,
    `door_name` VARCHAR(80) NOT NULL,
    `actor_identifier` VARCHAR(80) NULL,
    `actor_name` VARCHAR(80) NOT NULL,
    `action` VARCHAR(24) NOT NULL,
    `details` VARCHAR(255) NULL,
    `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    KEY `idx_vlr_doorlock_audit_door` (`door_id`),
    KEY `idx_vlr_doorlock_audit_created` (`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

Each door's full definition (parts, access rules, biometrics, detector, owner/editors)
is stored as JSON in the `data` column. Database door names are unique.

## Configuration

Everything lives in the open `config.lua`.

### General

| Key | Meaning |
|---|---|
| `Config.Locale` | Active locale (`'ba'` / `'en'`), independent of `ox:locale`. |
| `Config.Debug` | Enables debug prints. |
| `Config.UI` | NUI accent colour, panel `position`, and `showDistance`. |
| `Config.Notify` | Notification function (defaults to `lib.notify`). |

### Commands & keys

* `Config.Commands` — command names: `admin` (`dooradmin`), `toggle` (`doorlock`),
  `lockpick` (`lockpickdoor`).
* `Config.Keys` — interaction keys: `toggle` (`E`), `lockpick` (`G`).

### Interaction

`Config.Interaction` — `scanDistance`, `defaultDistance`, `serverTolerance`, `cooldown`,
`drawMarker`, `useTarget` (use `ox_target` when available) and `showHudFallback`.

### Security & lockpicking

* `Config.Security` — keypad brute-force protection: `keypadAttempts`, `keypadLockout`
  (seconds) and `lockpickTokenLifetime`.
* `Config.Lockpick` — `enabled`, allowed `items` (`lockpick`, `advancedlockpick`),
  per-item `breakChance`, skill-check `difficulty` and `inputs`.

### Biometrics & detectors

* `Config.BiometricScanner` — scanner prop `model` and interaction `distance`.
* `Config.Detector` — the item-detector / laser-checkpoint behaviour: detector prop
  `model`, `distance`, `drawRange`, `scanRange`, `wallStop`, `scanTime`, the invisible
  collision `wallModel` (with `wallHeadingOffset` / `wallZOffset`), `clearanceSeconds`,
  `scanCooldown`, and laser-beam geometry (`barrierWidth`, `laserWidth`, `laserCount`,
  `laserBottom`, `laserSpacing`).

### Admin & persistence

* `Config.Admin` — `ace` permission (`vlr.doorlock.admin`), framework `groups`, and
  `bypassLocks` (admins open any door, ignoring access rules — disable to test
  job-locked doors on an admin account).
* `Config.Persistence` — `persistStates`, `auditLimit`, `auditRetentionDays`.

### Presentation

* `Config.Audio` — lock/unlock beep sound set.
* `Config.Animation` — keycard-style use animation (`dictionary`, `clip`, `duration`).
* `Config.Webhook` — optional Discord audit webhook (`enabled`, `url`, `username`,
  `color`).

### Static doors

`Config.StaticDoors` — optional config-defined doors (same shape the in-game editor
produces). They use **negative runtime IDs** and are **read-only** in the panel; database
doors are recommended.

{% hint style="info" %}
The bridge also honours optional override hooks if you define them in config:
`Config.GetIdentifier`, `Config.GetJob`, `Config.GetGang`, and `Config.IsAdmin`. These let
you bolt the access logic onto a custom framework. They are not present by default.
{% endhint %}

## Commands

| Command | Default | Description |
|---|---|---|
| `/dooradmin` | `Config.Commands.admin` | Open the Valora Doorlock administration panel. Visible to admins, property owners and assigned editors; non-admins only see doors they may manage. |
| `/doorlock` | `Config.Commands.toggle` | Lock or unlock the closest door. |
| `/lockpickdoor` | `Config.Commands.lockpick` | Attempt to pick the closest eligible lock. |

By default `E` toggles the closest door and `G` attempts a lockpick (configurable via
`Config.Keys`).

## Developer API

### Server exports

```lua
-- Read a single door by its runtime id (full data)
local door = exports.vlr_doorlock:getDoor(1)

-- Read every door
local allDoors = exports.vlr_doorlock:getDoors()

-- Find a door by its unique internal name
local mrpd = exports.vlr_doorlock:getDoorByName('mrpd_reception')

-- Set a door's state: 0 = unlocked, 1 = locked. Returns true on success.
exports.vlr_doorlock:setState(1, 1)
```

| Export | Side | Description |
|---|---|---|
| `getDoor(id)` | server | Returns the door object for a runtime id, or `nil`. |
| `getDoors()` | server | Returns the list of all doors. |
| `getDoorByName(name)` | server | Returns the door whose unique internal name matches, or `nil`. |
| `setState(id, state)` | server | Sets a door locked (`1`) / unlocked (`0`); returns `true`/`false`. |

### Client exports

| Export | Side | Description |
|---|---|---|
| `useClosestDoor()` | client | Toggle the closest eligible door (the `E` action). |
| `pickClosestDoor()` | client | Attempt to lockpick the closest eligible door (the `G` action). |
| `getClosestDoor()` | client | Returns the currently tracked closest-door object. |

### Events

```lua
AddEventHandler('vlr_doorlock:stateChanged', function(source, doorId, locked, reason)
    -- source : the player who triggered it (0 for system/external)
    -- doorId : the door's runtime id
    -- locked : boolean (true = now locked)
    -- reason : string describing the trigger (e.g. 'external')
    -- Integrate alarms, dispatch, robberies or logging here.
end)
```

| Event | Direction | Purpose |
|---|---|---|
| `vlr_doorlock:stateChanged` | server (emit) | Fired whenever a door's lock state changes. |
| `vlr_doorlock:server:setState` | server (net) | Internal — accepts state changes only from the server console (`source == 0`). |

Internal client net events (`vlr_doorlock:client:setState`, `:sync`, `:upsert`,
`:remove`, `:notify`, `:openAdmin`) and server `lib.callback` handlers
(`vlr_doorlock:toggle`, `:admin`, `:beginLockpick`, `:breakLockpick`, `:canUse`,
`:scanDetector`, `:getDoors`) drive the UI and are listed for integration awareness
only — they are not a public API.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `/dooradmin` says access denied | Grant the ACE (`add_ace group.admin vlr.doorlock.admin allow`) or add your group to `Config.Admin.groups`. Property owners/editors only see their own doors. |
| A job/gang-locked door opens for an admin anyway | `Config.Admin.bypassLocks` is `true` (admins open any door). Set it to `false` to test access rules on an admin/god account. |
| Doors don't persist after restart | Ensure `oxmysql` is started before `vlr_doorlock` and the tables exist; check the door's "State after restart" setting (always-locked / always-unlocked / restore-last-state). |
| Sliding / garage door won't close again | It must be flagged **Automatic / sliding** — those use the fixed Valora HUD + `E` because the door object may have moved out of `ox_target` range. |
| Lockpicking does nothing | `Config.Lockpick.enabled` must be on, the door must allow lockpicking, and the player must carry a configured lockpick item. |
| Item not recognised in access rules / detector | Confirm the item name; the bridge falls back across ox / qb / esx inventories and tries weapon name variants automatically. |
| Detector never grants clearance | The player must physically pass through the laser checkpoint, must not carry any prohibited item, and clearance is consumed on entry (re-scan if it expires). |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
