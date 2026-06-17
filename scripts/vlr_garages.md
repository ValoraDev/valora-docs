---
description: >-
  Modern multi-framework garage system for FiveM — owned & public garages, a
  cross-city fleet view, police impound, premium interiors, realpark, lost/orphan
  vehicle recovery and an in-game admin builder, with exact body-damage
  persistence. Server-authoritative, ESX / QBCore / QBox auto-detected.
---

# vlr_garages — Garage System

`vlr_garages` is a complete vehicle storage suite: park and retrieve owned vehicles
across standard, premium and realpark garages, run a police impound, recover lost or
orphaned cars, and build every garage live in-game with no manual config editing. It
auto-detects your framework, target, inventory and notify stack, and persists exact
body damage so a car comes back exactly as it was parked.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora Development |
| **Framework** | ESX / QBCore / QBox — auto-detected (`Config.Framework`) |
| **Storage** | oxmysql (own `vlr_garages` table + columns added to your vehicles table) |
| **Built on** | ox_lib + oxmysql + PolyZone |

{% hint style="info" %}
A framework (ESX / QBCore / QBox) is **required** — `vlr_garages` is not standalone. All
three are genuinely supported through a runtime bridge. **ESX note:** an impounded vehicle
is marked with `stored = 2` on the `owned_vehicles` table; that is correct for `vlr_garages`
itself, but another ESX garage script reading that column may treat the car as "stored".
Run one garage system at a time.
{% endhint %}

## Features

* **Standard garages** — park & retrieve owned vehicles, with per-garage slot limits
  and ownership filtering.
* **Buy & own garages** — players can purchase a garage, set an hourly price, manage an
  access list and rename it. Owner debt is tracked and paid through the UI.
* **My Fleet (cross-city view)** — see every vehicle you own across the map and pay a
  transfer fee to bring one to the garage you're standing in.
* **Police impound** — authorized jobs seize any vehicle with a reason and a lock
  timer; the owner pays a fee (and waits out the lock) to retrieve it. Officers can
  release without payment.
* **Lost & orphan recovery** — vehicles stranded by a deleted garage are flagged as
  *orphans* and can be transferred; a background sweep flips destroyed/lost cars into
  impound so they can be recovered; impounded cars can also be pulled remotely for a
  surcharge.
* **Premium interiors** — teleport-based garages using configurable interior presets
  and spawn slots.
* **Realpark** — retrieve a vehicle at fixed display slots, with slot-reservation
  protection that warns or impounds vehicles blocking a slot.
* **Exact body-damage persistence** — dirt, body health, deformation, broken
  doors/tyres/windows/bumpers are saved and restored on next spawn (`Config.SaveExactDeformation`).
* **In-game admin builder** — create, edit, move vehicles between, and delete garages
  live, captured from a camera raycast.
* **NPC garages & blips** — an optional ped + target at each garage's access point, plus
  per-type map blips.
* **Themeable NUI** — clean vanilla NUI with a configurable accent color and per-vehicle
  preview images from local files, uz_AutoShot, or a CDN.
* **Localization** — ships in **English**, fully translatable via the open `locales/` files.
  More languages are planned over time.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/communityox/ox_lib) | Required | UI, callbacks, notifications, locale. |
| [oxmysql](https://github.com/overextended/oxmysql) | Required | Garage and vehicle persistence. |
| [PolyZone](https://github.com/mkafrin/PolyZone) | Required | Loaded directly by the manifest's client scripts; used by the qb-target path. |
| A target resource — [ox_target](https://github.com/overextended/ox_target) **or** qb-target | Required | `Config.Target` (`auto` / `ox_target` / `qb-target`). |
| Framework — ESX / QBCore / QBox | Required | Auto-detected via `Config.Framework`. |
| [ox_inventory](https://github.com/overextended/ox_inventory) / qb-inventory / qs-inventory | Optional | `Config.Inventory` — only used where inventory access is needed. |
| Fuel resource (LegacyFuel, ox_fuel, ps-fuel, lc_fuel, …) | Optional | `Config.FuelResource` — set to `nil` to use native fuel. |
| Vehicle-keys resource (qbx/qb/wasabi or any export) | Optional | `Config.KeysResource` — set to `nil` to disable key handling. |
| uz_AutoShot | Optional | Source for vehicle preview images (`Config.VehicleImages.autoShot`). |

{% hint style="info" %}
`vlr_garages` declares only `oxmysql` and `ox_lib` in its `fxmanifest` dependencies block.
PolyZone is pulled in directly through the manifest's client scripts, and the framework,
target and the optional resources above are detected/consumed at runtime — make sure the
ones you use are started **before** `vlr_garages`.
{% endhint %}

## Installation

### 1. Install the resource

1. Copy `vlr_garages` into `resources/` (we recommend `resources/[valora]/vlr_garages`).
2. Start it **after** its dependencies in `server.cfg`:

```cfg
ensure oxmysql
ensure ox_lib
ensure PolyZone
ensure ox_target        # or qb-target
ensure vlr_garages
```

### 2. Import the database

Import `sql/install.sql` into your database. It is safe to re-run — it uses
`IF NOT EXISTS` for the `vlr_garages` table and column-existence checks before each
`ALTER TABLE`. See [Database](#database) below.

### 3. Configure

Edit `config/config.lua` — set `Config.Framework`, `Config.Target`, `Config.Inventory`,
`Config.Notify`, your `Config.FuelResource` / `Config.KeysResource`, the
`Config.AdminIdentifiers` list, and money/locale options. Premium interior presets live
in `config/garages.lua`.

### 4. Build your garages

Add an admin's citizenid/identifier to `Config.AdminIdentifiers`, then in-game run the
admin command (default `/vlrgaraze`) to create, edit and place garages live — no manual
config editing required.

## Database

`vlr_garages` uses **oxmysql**. Importing `sql/install.sql` does two things:

**1. Creates the resource's own garage table:**

```sql
CREATE TABLE IF NOT EXISTS `vlr_garages` (
    `id`         VARCHAR(64)  NOT NULL PRIMARY KEY,
    `label`      VARCHAR(120) NOT NULL,
    `type`       ENUM('standard','premium','realpark','impound') NOT NULL DEFAULT 'standard',
    `owner`      VARCHAR(64)  NULL,
    `owner_name` VARCHAR(120) NULL,
    `coords`     LONGTEXT     NOT NULL,
    `vehicle`    LONGTEXT     NOT NULL,
    `meta`       LONGTEXT     NOT NULL,
    `created_at` TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at` TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX `vlr_garages_owner_idx` (`owner`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```

**2. Adds two backwards-compatible columns to your existing vehicles table**
(`player_vehicles` for qbx/qbcore, `owned_vehicles` for ESX), each guarded by a
column-existence check so re-running is safe:

| Column | Type | Purpose |
|---|---|---|
| `vlr_orphan` | `TINYINT(1)` default `0` | Marks a vehicle stranded by a deleted garage. |
| `vlr_impound_meta` | `TEXT` NULL | JSON blob written on impound (destination garage, reason, locked-until, fee, officer id & name); cleared on release. |

The script reads/writes your framework's standard vehicle columns through `Config.DB`
(`table`, `identifier`, `plate`, `vehicle`, `state`, `garage`, `impoundFee`), which is
auto-filled per framework.

## Configuration

All normal configuration lives in `config/config.lua`; premium interior presets and
blip/vehicle-type defaults live in `config/garages.lua`. Key groups:

### Framework & integrations

| Key | Purpose |
|---|---|
| `Config.Framework` | `auto` / `qbox` / `qbcore` / `esx` (auto picks qbox > qbcore > esx). |
| `Config.Target` | `auto` / `ox_target` / `qb-target`. |
| `Config.Inventory` | `auto` / `ox_inventory` / `qb-inventory` / `qs-inventory` / `esx`. |
| `Config.Notify` | `auto` / `ox_lib` / `qb` / `esx` / `custom`. |
| `Config.SQL` | Database layer (only `oxmysql` is officially supported). |
| `Config.FuelResource` | Fuel export resource, or `nil` for native fuel. |
| `Config.KeysResource` | Vehicle-keys export resource, or `nil` to disable. |

### Localization

| Key | Purpose |
|---|---|
| `Config.DefaultLocale` | Active locale (default `en`). Add more `locales/<code>` files to translate. |
| `Config.FallbackLocale` | Used when a key is missing in the active locale. |

### General behavior

| Key | Purpose |
|---|---|
| `Config.Debug` | Extra console logging. |
| `Config.AdminIdentifiers` | List of citizenids/identifiers allowed to run the admin builder. |
| `Config.AdminCommand` | Admin builder command name (default `vlrgaraze`). |
| `Config.SpawnGarageNPC` | Spawn an NPC with a target at each garage access point. |
| `Config.GaragePedModel` / `Config.GaragePedZOffset` | NPC model and Z nudge. |
| `Config.FilterByOwner` | Show only the player's own vehicles in the list. |
| `Config.SharedRealparkPremium` | Make premium/realpark lots shared rather than ownership-filtered. |
| `Config.RepartCooldown` | Seconds before a freshly taken-out vehicle can be re-parked (anti-exploit). |
| `Config.SaveExactDeformation` | Persist exact dirt/body health/deformation/broken parts. |
| `Config.DefaultImpoundPrice` | Fallback impound fee when the vehicle row carries none. |
| `Config.DebtCheckInterval` | Minutes between hourly-debt accrual runs. |
| `Config.AllowGarageRename` | Let an owner rename their garage. |

### Recovery, transfer & impound

| Key | Purpose |
|---|---|
| `Config.Orphan` | Orphan-vehicle recovery (`Enabled`, `AllowAnyGarage`, `TransferFee`, and a nested `ImpoundRecovery` block for remote impound pickup: `Enabled`, `Multiplier`, `MinFee`, `Account`, `RespectLock`). |
| `Config.Transfer` | Paid transfer of an intact vehicle between garages (`Enabled`, `Fee`, `Account`, `AllowAnyGarage`). |
| `Config.Impound` | Impound-garage behavior (`ImpoundOnlyMode`, `PreferCash`). |
| `Config.RealparkSlotProtection` | Reserve realpark display slots (`Enabled`, `Radius`, `ImpoundFee`, `WarnCooldown`, `WarnDistance`). |
| `Config.LostVehicleRecovery` | Background sweep that flips destroyed/lost vehicles into impound (`Enabled`, `CheckInterval`, `Fee`, `Reason`, `DestinationGarage`, `GracePeriod`). |

### Police impound

`Config.Police` controls the officer flow: `Enabled`, `AuthorizedJobs`,
`DurationPresets` (a `seconds = 0` preset means permanent), `Reasons`, `DefaultFee` /
`MinFee` / `MaxFee`, `AllowOfficerRelease`, `Command` (default `impound`, set `nil` to
disable) and `UseTarget`.

### Money & UI

| Key | Purpose |
|---|---|
| `Config.PurchaseAccount` / `Config.ImpoundAccount` | `bank` or `cash` for purchase/impound costs. |
| `Config.DB` | Per-framework vehicle-column mapping (auto-filled; override only for custom schemas). |
| `Config.UI` | Accent/theme colors piped to CSS variables and the in-world marker (`accent`, `accentSoft`, `danger`, `warning`, `success`). |
| `Config.VehicleImageCDN` | Base URL for vehicle thumbnails. |
| `Config.VehicleImages` | Preview-image sources tried in order: `localFolder`, `autoShot` (uz_AutoShot), `cdn`, `placeholder`. |

### Premium presets (`config/garages.lua`)

| Key | Purpose |
|---|---|
| `Config.PremiumPresets` | Teleportable interiors (`rankStars`, `interiorCenter`, `spawnInside`, `spawnHeading`, `slots`). Ships with `lowgarage`, `midgarage`, `highgarage`. |
| `Config.VehicleTypes` | Vehicle-class options for the admin dropdown (`all`, `automobile`, `bike`, `plane`, `heli`, `boat`). |
| `Config.BlipDefaults` | Default blip sprite/color/size per garage type. |

## Commands

| Command | Description |
|---|---|
| `/vlrgaraze` | Open the in-game admin garage builder. Restricted to `Config.AdminIdentifiers`. Name is set by `Config.AdminCommand`. |
| `/impound` | Open the police impound UI on a nearby vehicle. For `Config.Police.AuthorizedJobs` only; name is set by `Config.Police.Command` (set to `nil` to disable and use ox_target instead). |

## Developer API

`vlr_garages` exposes **no public exports**. Integration happens through ox_lib
callbacks and net events.

### Server callbacks (`lib.callback`)

| Callback | Purpose |
|---|---|
| `vlr_garages:requestLostScan` | Run a self-service scan for the caller's lost/destroyed vehicles and flip eligible ones into impound. |
| `vlr_garages:vehicleImage` | Resolve and return a vehicle preview image for a given spawn code (used by the UI's image pipeline). |

### Net events

The resource drives its UI and flows through a large set of internal net events (server-
and client-side), for example: `vlr_garages:requestBootstrap`, `:requestGarage`,
`:park`, `:takeOutStandard`, `:takeOutPremium`, `:takeOutRealpark`, `:buyGarage`,
`:sellGarage`, `:setHourlyPrice`, `:payDebt`, `:addAccess` / `:removeAccess`,
`:renameGarage`, `:transferVehicle`, `:transferOrphan`, `:policeImpound`,
`:policeRelease`, `:releaseImpound`, `:recoverImpoundRemote`, and the admin events
`:adminCreate` / `:adminUpdateGarage` / `:adminDelete` / `:adminMoveVehicle` /
`:adminReleaseImpound`.

{% hint style="warning" %}
These events are **internal** — they are server-authoritative and validate identity and
permissions on the server. They are listed for integration awareness; trigger them
directly only if you know exactly what you are doing.
{% endhint %}

## Troubleshooting

| Symptom | Fix |
|---|---|
| No garages appear | None created yet — add an admin to `Config.AdminIdentifiers` and build one with `/vlrgaraze`. |
| `/vlrgaraze` does nothing | Your citizenid/identifier isn't in `Config.AdminIdentifiers`, or the command was renamed via `Config.AdminCommand`. |
| Officers can't impound | Job not in `Config.Police.AuthorizedJobs`, `Config.Police.Enabled = false`, or the command was renamed/disabled (`Config.Police.Command`). |
| Vehicles don't restore damage | `Config.SaveExactDeformation` is `false`. |
| Player re-parks a damaged car as clean | That's why `Config.RepartCooldown` exists — raise it if needed. |
| No vehicle thumbnails | Check `Config.VehicleImages` sources (drop `<spawncode>.png` into `html/img/vehicles/`, enable uz_AutoShot, or keep the CDN on). |
| Fuel/keys not applied on take-out | Set the correct `Config.FuelResource` / `Config.KeysResource` exports, or `nil` to use defaults. |
| SQL errors on start | Import `sql/install.sql` manually; it is safe to re-run. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
