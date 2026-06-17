---
description: >-
  Server-authoritative sit system for FiveM — per-model seat profiles, multi-seat
  synced benches, an in-world ghost editor for pixel-perfect seat placement, and a
  catalog of custom props you can place and sync to every player. Configured entirely
  in game through an industrial-luxe Valora panel.
---

# vlr_sitsys — Sit System

`vlr_sitsys` lets players sit on chairs and benches through `ox_target`, while admins
configure every model **in game** with a full Valora panel — including a live in-world
**ghost editor** that shows exactly where a player will land on each seat. Multi-seat
benches are **server-synchronized**, so two players can never claim the same seat, and a
custom-prop catalog lets you place streamed props in the world that are synced to everyone.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | qbox / qb / esx — auto-detected (only for the admin permission check); standalone via ACE |
| **Storage** | oxmysql (self-creating tables) |
| **Built on** | ox_lib + ox_target |

## Features

* **Per-model seat profiles** — every chair/bench model carries its own seat layout, sit
  style (scenario) and interaction distance.
* **Multi-seat benches** — server-authoritative occupancy means no two players can share a
  seat; seats free up automatically when a player stands, dies, or drops.
* **Ghost editor** — spawns a translucent seated ghost on every seat so you can nudge each
  one live in the world (WASD to move, height/rotate with the arrows, SPACE to switch
  seats) and save. What you see is exactly where players land.
* **Custom props (placed + synced)** — keep a catalog of your custom prop models, place
  them in the world from the panel, and they are saved to the database and spawned for
  **every player**, sittable via their seat profile.
* **In-game prop capture** — look at any chair or bench and capture its model with a
  camera raycast, then build its profile.
* **NPC check** — refuses a seat a local NPC has already taken, with a themed message.
* **Broad out-of-the-box coverage** — a generic single-seat list covers dozens of common
  GTA chair models; edit any of them in game to save a tuned override.
* **Auto stand-up** — the player stands automatically on damage/ragdoll, death, or when
  entering a vehicle (each configurable).
* **Industrial-luxe Valora NUI** — graphite + gold, vanilla JS, no build step, no
  `backdrop-filter`; locale-driven (`en` / `ba`).

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | Yes | UI, callbacks, commands, keymapping, notifications. |
| [ox_target](https://github.com/overextended/ox_target) | Yes | The sit / stand interaction on chair and bench models. |
| [oxmysql](https://github.com/overextended/oxmysql) | Yes | Persists seat profiles, the custom-prop catalog, and placed props. |
| Framework (qbox / qb / esx) | Optional | Only for the admin permission check. Auto-detected; works standalone via the ACE permission `vlr.sitsys.admin`. |

## Installation

{% hint style="info" %}
**Plug & play.** No core edits and no `ox_inventory` items — this is a drop-in resource.
The only setup is starting it after its dependencies and granting your admins access.
{% endhint %}

### 1. Install the resource

1. Copy `vlr_sitsys` into `resources/` (we recommend `resources/[valora]/vlr_sitsys`).
2. Start it **after** its dependencies in `server.cfg`:

```cfg
ensure ox_lib
ensure ox_target
ensure oxmysql
ensure vlr_sitsys
```

### 2. Database

The tables **self-create on first start**. Importing `install/vlr_sitsys.sql` is only
needed for manual setups — see [Database](#database).

### 3. Grant admin access

Configuration is admin-only. Grant access either through a framework admin group listed in
`Config.Admin.groups`, or with the ACE permission:

```cfg
add_ace group.admin vlr.sitsys.admin allow
```

### 4. Test it

* Look at a chair or bench → `ox_target` → **Sit down** / **Stand up** (or press **X** /
  `/standup` to get up).
* Run `/sitadmin` (as an admin) to open the configuration panel.

## Database

`vlr_sitsys` uses **oxmysql** with three self-creating tables:

* `vlr_sitsys` — one row per model; the JSON profile produced by the in-game editor (slots,
  scenario, distance, label) is stored in `data`.
* `vlr_sitsys_catalog` — the custom-prop catalog admins build in the panel (`model` + `label`).
* `vlr_sitsys_props` — the props placed in the world (model + coords + heading), spawned for
  every player.

All three are created on first start; `install/vlr_sitsys.sql` ships the primary profile
table for manual imports.

```sql
CREATE TABLE IF NOT EXISTS `vlr_sitsys` (
    `id`         INT UNSIGNED NOT NULL AUTO_INCREMENT,
    `model`      VARCHAR(80)  NOT NULL,
    `data`       LONGTEXT     NOT NULL,
    `created_at` TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uq_vlr_sitsys_model` (`model`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

{% hint style="info" %}
Seat **occupancy** (who is sitting where right now) is held **in memory** and resets on a
resource restart — nobody is seated after a restart. Only the seat *profiles*, the *catalog*
and *placed props* are persisted.
{% endhint %}

## Configuration

Everything lives in the open `config.lua`. Most day-to-day tuning happens **in game** via
`/sitadmin`; the config file holds defaults and global behaviour.

### General

| Key | Meaning |
|---|---|
| `Config.Locale` | UI / notification language — `'en'` or `'ba'` (add more `locales/<code>.json`). |
| `Config.Debug` | Developer logging. |
| `Config.UI.accent` | Panel accent colour (default Valora gold `#d4af37`). |
| `Config.Commands` | Command names — `admin` (default `sitadmin`) and `stand` (default `standup`). |
| `Config.Keys.stand` | Key bound to standing up while seated (default `X`). |

### Interaction

`Config.Interaction` controls the `ox_target` reach and the server-side range check:
`defaultDistance` for a new profile, `minDistance` / `maxDistance` editor bounds,
`serverTolerance` added to the distance for the seat-claim check, and `keyHint` to show a
small "[X] Stand up" text UI while seated.

### Sit behaviour

`Config.Sit` defines the default scenario and the auto-stand rules: `standOnDamage`,
`standOnDeath`, `standOnVehicle`, plus the NPC guard `checkNpc` and `npcBlockRadius` (how
close a local must be to count as occupying a seat).

### Seats & scenarios

* `Config.GenericSeat` — the default single seat (scenario, prop-local `offset`, `heading`)
  applied to an unconfigured chair and used as the seed when you first open the ghost editor.
* `Config.Scenarios` — the sit styles offered in the editor dropdown (Bench/Chair,
  Chair upright, Armchair, Deckchair, Sunlounger, Steps). Only verified, reliably-playing
  scenarios are listed.
* `Config.DefaultProfiles` — curated multi-seat profiles ready out of the box (park benches,
  etc.). The shape matches the in-game editor output, so anything tuned with the ghost editor
  can be pasted back here to ship as a default.
* `Config.GenericChairs` — a long list of common GTA chair models that get an automatic
  single seat. Editing any of them in game saves a tuned override.
* `Config.CustomPropCatalog` — seed entries for the custom-prop catalog (`model` + `label`);
  admins add more live in the panel.

### Admin access

`Config.Admin` sets who may configure seats: the ACE permission `ace` (default
`vlr.sitsys.admin`) and framework `groups` (default `admin`, `god`).

### Notifications

`Config.Notify` is a function wrapping `lib.notify`, so you can re-route all of the
script's notifications to your own system in one place.

## Commands

| Command | Access | Description |
|---|---|---|
| `/sitadmin` | Admin | Open the Sit System configuration panel (capture, profiles, ghost editor, custom props). |
| `/standup` | Player | Stand up from a seat (also bound to the **X** key). |

Both names are configurable in `config.lua` → `Config.Commands`, and the stand-up key in
`Config.Keys.stand`.

## Usage

* **Sit / stand** — look at a chair or bench, use `ox_target` → *Sit down* / *Stand up*.
  Press **X** (configurable) or `/standup` to get up.
* **Configure** — `/sitadmin` opens the panel:
  * **Capture prop** → look at a chair/bench and press `E` to capture its model.
  * Set a label, sit style, interaction distance, and toggle the profile on/off.
  * **Open ghost** → in-world editor: `WASD` move, `↑/↓` height, `←/→` rotate, `SHIFT`
    faster, `SPACE` switch seat, `E` save, `ESC` cancel. Add/remove seats in the panel.
  * **Save** → the profile is stored in the database and synced to all players instantly.
* **Custom props** — `/sitadmin` → **Custom props** tab: add a model to the catalog, place
  it in the world, then create its seat profile so players can sit on it. Placed props are
  listed with a delete button.

## Developer API

All exports are **server-side**.

### `exports.vlr_sitsys:getProfiles()`

Returns the list of enabled seat profiles.

```lua
local profiles = exports.vlr_sitsys:getProfiles()
```

### `exports.vlr_sitsys:isSeatTaken(propKey, slot)`

Returns `true` if the given seat slot on a prop instance is currently occupied.

```lua
local taken = exports.vlr_sitsys:isSeatTaken(propKey, slot)
```

### `exports.vlr_sitsys:getSeat(playerId)`

Returns the seat a player currently occupies as `{ key, slot }`, or `nil` if they are not
seated.

```lua
local seat = exports.vlr_sitsys:getSeat(playerId)
```

### Server callbacks (`lib.callback`)

Driven internally by the client; listed for integration awareness:
`vlr_sitsys:admin`, `vlr_sitsys:claimSeat`, `vlr_sitsys:releaseSeat`,
`vlr_sitsys:getData`.

### Net events

| Event | Direction | Purpose |
|---|---|---|
| `vlr_sitsys:client:sync` | server → client | Push the current profile / placed-prop set to clients. |
| `vlr_sitsys:client:seatUpdate` | server → client | Broadcast an occupancy change for a seat. |
| `vlr_sitsys:client:openAdmin` | server → client | Open the admin panel for an authorized player. |
| `vlr_sitsys:client:notify` | server → client | Show a localized notification. |
| `vlr_sitsys:client:propAdded` | server → client | A custom prop was placed — spawn it. |
| `vlr_sitsys:client:propRemoved` | server → client | A placed custom prop was deleted — remove it. |
| `vlr_sitsys:server:release` | client → server | Release the seat the player is leaving. |

## Troubleshooting

| Symptom | Fix |
|---|---|
| `/sitadmin` says access denied | Add your admin group to `Config.Admin.groups`, or grant the ACE `add_ace group.admin vlr.sitsys.admin allow`. |
| No *Sit down* option on a chair | The model has no enabled profile — it must be in `Config.GenericChairs`/`Config.DefaultProfiles` or captured and saved via `/sitadmin`. |
| "Save did not persist" | oxmysql is not started or not reachable — ensure `oxmysql` starts before `vlr_sitsys`. |
| Seats reset after a restart | Expected — occupancy is in-memory; profiles, catalog and placed props persist, who is seated does not. |
| "Shoo the local off…" | An NPC is occupying that seat. Lower `Config.Sit.npcBlockRadius` or set `Config.Sit.checkNpc = false` if you don't want this guard. |
| Custom prop doesn't appear for others | It must be placed through the panel (which saves + syncs it); the model also has to be streamed/available on every client. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
