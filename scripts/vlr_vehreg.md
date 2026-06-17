---
description: >-
  Realistic vehicle registration and technical inspection for FiveM — office
  clerk, mechanic walk-around inspection and a global police DMV NPC, with
  player-owned center businesses, a live admin panel, and optional t1ger_mechanic
  and lb-tablet integration. Server-authoritative and framework-agnostic.
---

# vlr_vehreg — Vehicle Registration & Technical Inspection

`vlr_vehreg` turns vehicle paperwork into a full roleplay loop: a player takes a car to a
registration **center**, opens a case with the **office clerk**, has the **mechanic** run a
real technical inspection (the car can fail and must be repaired), and ends up with a
registration that is **valid for a configurable period and then expires**. A single fixed
**police DMV NPC** handles ownership transfers and plate look-ups, police can order
extraordinary inspections, and every center can be a **player-owned business** managed live
from an in-game admin panel. Standalone, with automatic `t1ger_mechanic` and `lb-tablet`
integration when present.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | qbox / qb / esx — auto-detected (`qbox > qb > esx > standalone`); force via `Config.Framework` |
| **Storage** | oxmysql (4 tables, self-creating) |
| **Built on** | ox_lib + ox_target |

## Features

* **Registration centers** — each center bundles two NPCs (office clerk + mechanic) and an
  inspection bay. Add as many centers as you like.
* **Office clerk** — the player reports here first: pick one of their vehicles, open a
  registration or renewal case, review existing registrations, or reprint a lost document.
* **Technical inspection (walk-around)** — the mechanic NPC circles the parked vehicle while
  a focus-less overlay reveals each check live; the player stays free to watch, never frozen.
  With `t1ger_mechanic` present the read is authoritative (core-part health, failured parts,
  mileage); otherwise it uses native engine + body health plus **measured** glass, tyres and
  lights — a wrecked car hard-fails. The mechanic's walk-around is **synced** to nearby players.
* **Registration lifecycle** — configurable validity period, renew grace window, expiry
  warnings, and a server sweep that flips expired registrations automatically.
* **Police DMV NPC** — one fixed global NPC handles **ownership transfer** as a contract
  (owner offers a vehicle to a nearby buyer at a price; the buyer accepts, pays the seller
  plus a small admin fee, and framework ownership + the registration record move over) and
  **plate look-ups**.
* **Police tools** — on-duty police get a vehicle `ox_target` option, `/regcheck` to look up
  any plate, and `/ordreg` to order an extraordinary inspection that suspends the
  registration until the car passes again.
* **Physical document item** *(optional)* — issues a `vehicle_registration` ox_inventory item
  alongside the DB record; using it opens the full registration document. The DB is always the
  source of truth.
* **In-game admin panel** (`/regadmin`) — create / delete / rename centers, place every NPC
  and the inspection bay with a ghost editor, set default fees and all business parameters,
  reposition the police NPC, and assign or clear center owners. Everything is DB-backed and
  re-syncs to all players live (no restart).
* **Player-owned center business** *(optional)* — fees flow into a center **treasury** (minus
  a state tax); the owner withdraws profit, sets per-center prices within admin caps, and
  keeps the center alive through **supplies** (forms per document, diagnostic kits per
  inspection), **equipment integrity** (wears with use and over time) and an **operating
  license**. A center closes when any of these runs out. **State-run** centers never close,
  never consume supplies/equipment, can't be bought, and charge a configurable fee multiple.
* **Vehicle preview images** — office cards show a picture of each vehicle by spawn code,
  resolved from a local folder, optional `uz_AutoShot` shots, or a CDN fallback.
* **Optional integrations** — `t1ger_mechanic` (authoritative condition, owned-plate claim,
  service history, plate updates), `lb-tablet` (MDT vehicle profiles + status tags), and
  `nc-banking` (route center income into a society account).
* **Server-authoritative** — identity from `source`, ownership and fees validated server-side,
  the server decides pass/fail; dates are formatted server-side and the NUI only renders
  ready-made strings (vanilla JS, `textContent`, code-drawn SVG, no emoji, no `backdrop-filter`).
* **Localised** — `en` + `ba` locale files, switchable via `Config.Locale`.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | Required | Callbacks, NUI helpers, notifications. |
| [oxmysql](https://github.com/overextended/oxmysql) | Required | Persistence (registrations, log, settings, centers). |
| [ox_target](https://github.com/overextended/ox_target) | Required | NPC and vehicle interaction targets. |
| Framework (qbox / qb / esx) | Optional | Auto-detected; runs standalone otherwise. |
| `t1ger_mechanic` | Optional | Authoritative inspection condition + service history + plate sync. |
| `lb-tablet` | Optional | Writes MDT vehicle profiles and registration status tags. |
| `nc-banking` | Optional | Deposit center income into a society account. |
| [ox_inventory](https://github.com/overextended/ox_inventory) | Optional | Physical `vehicle_registration` document item. |

{% hint style="info" %}
Only `ox_lib`, `oxmysql` and `ox_target` are hard dependencies (declared in
`fxmanifest.lua`). Everything else is auto-detected and degrades gracefully when absent.
{% endhint %}

## Installation

### 1. Install the resource

1. Copy `vlr_vehreg` into `resources/` (we recommend `resources/[valora]/vlr_vehreg`).
2. Start it **after** its dependencies — and after your framework / `t1ger_mechanic` /
   `lb-tablet` if you use them:

```cfg
ensure ox_lib
ensure oxmysql
ensure ox_target
ensure vlr_vehreg
```

### 2. Database

The four tables **self-create on first start** (`CREATE TABLE IF NOT EXISTS`). Importing
`install/vlr_vehreg.sql` is optional but recommended for a clean install. See
[Database](#database).

### 3. (Optional) Register the document item

To issue the physical "saobraćajna" item, register it in `ox_inventory/data/items.lua`:

```lua
['vehicle_registration'] = {
    label = 'Saobraćajna',
    weight = 10,
    stack = false,
    close = true,
    description = 'Potvrda o registraciji vozila',
    client = { export = 'vlr_vehreg.useDocument' },
},
```

Then keep `Config.Document.asItem = true` (the default). On qb/esx (non-ox inventories) the
item is registered usable automatically — no `client.export` line is needed.

### 4. Place NPCs and tune fees in-game

`config.lua` only **seeds** the database on first boot. After that the DB is the source of
truth: open the admin panel with `/regadmin` to create centers, place NPCs and the inspection
bay with the ghost editor, set fees, and configure the business economy. Editing `config.lua`
afterwards does nothing unless the DB row is missing.

## Database

`vlr_vehreg` uses **oxmysql**. Four self-creating tables:

```sql
CREATE TABLE IF NOT EXISTS `vlr_vehicle_registrations` (
    `plate` VARCHAR(12) NOT NULL,
    `vin` VARCHAR(24) DEFAULT NULL,
    `owner_identifier` VARCHAR(80) NOT NULL,
    `owner_name` VARCHAR(120) DEFAULT NULL,
    `model` VARCHAR(60) DEFAULT NULL,
    `model_label` VARCHAR(80) DEFAULT NULL,
    `color` INT DEFAULT NULL,
    `color_name` VARCHAR(40) DEFAULT NULL,
    `status` VARCHAR(20) NOT NULL DEFAULT 'unregistered',
    `paid_admin` TINYINT(1) NOT NULL DEFAULT 0,
    `inspection_passed` TINYINT(1) NOT NULL DEFAULT 0,
    `inspection_at` BIGINT DEFAULT NULL,
    `inspection_result` LONGTEXT DEFAULT NULL,
    `police_hold` TINYINT(1) NOT NULL DEFAULT 0,
    `issued_at` BIGINT DEFAULT NULL,
    `expires_at` BIGINT DEFAULT NULL,
    `center` VARCHAR(40) DEFAULT NULL,
    `created_at` BIGINT DEFAULT NULL,
    `updated_at` BIGINT DEFAULT NULL,
    PRIMARY KEY (`plate`),
    UNIQUE KEY `uq_vin` (`vin`),
    KEY `idx_owner` (`owner_identifier`),
    KEY `idx_status` (`status`)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

| Table | Purpose |
|---|---|
| `vlr_vehicle_registrations` | One row per plate — owner, model, status, inspection result, expiry, police hold. |
| `vlr_registration_log` | Audit log of registration actions (`action`, `actor`, `detail`). |
| `vlr_reg_settings` | Runtime config (`k`/`v`) seeded from `config.lua`, edited live via `/regadmin`. |
| `vlr_reg_centers` | Per-center state — placement JSON, owner, treasury, supplies, equipment integrity, license, buy price. |

{% hint style="warning" %}
After first boot, `vlr_reg_settings` and `vlr_reg_centers` are the source of truth — edit them
through `/regadmin`, not `config.lua`.
{% endhint %}

## Configuration

Everything lives in the open `config.lua`. Top-level sections:

| Section | What it controls |
|---|---|
| `Config.Locale` / `Config.Debug` / `Config.Framework` | Active locale (`'ba'`/`'en'`), diagnostics gate, framework override (`auto`/`qbox`/`qb`/`esx`/`standalone`). |
| `Config.Payment` | Charge account (`cash`/`bank`), cash fallback, currency symbol, optional nc-banking `society` deposit. |
| `Config.Prices` | Default fees: `newRegistration`, `renewal`, `inspection`, `titleChange`, `transfer`, `duplicate`. |
| `Config.Registration` | `validDays`, `renewGraceDays`, `expireWarnDays`, `inspectionValidHours`, re-inspection rules, expiry sweep interval, `vinPrefix`, `plateMaxLength`. |
| `Config.Inspection` | t1ger toggle, reveal/linger timings, watchdog cap, bay radius, owner/engine requirements, pass `thresholds` (engine/body/core part), t1ger core parts per drivetrain, cosmetic checks, `maxCosmeticDefects`. |
| `Config.Document` | Physical item on/off (`asItem`), `itemName`, `oneItemPerPlate`. |
| `Config.VehicleImages` | Office card images — local folder, `uz_AutoShot`, CDN fallback, placeholder. |
| `Config.Police` | Police `jobs`/`minGrade`, `/regcheck` & `/ordreg` command names, vehicle target, admin ACE/groups, `/regadmin` panel name. |
| `Config.Integrations` | Per-integration toggles for `t1ger` and `lbTablet` (incl. MDT tag styling and backfill). |
| `Config.Target` / `Config.Ped` | Interaction distance and per-NPC icons; NPC freeze/invincible/blockEvents flags. |
| `Config.Centers` | **Seed** centers (id, label, blip, office + inspector NPC + bay). |
| `Config.PoliceNpc` | The single fixed global police DMV NPC (label, model, coords, blip). |
| `Config.Business` | Ownership economy: `enabled`, `stateTaxPct`, `buyPrice`, `sellBackPct`, `stateFeeMultiplier`, per-fee `priceCaps`, `supplies`, `equipment` wear, `license`. |
| `Config.Dev` | `/reg_devpos` position-printer command name (Debug only). |
| `Config.Notify` | Notification helper (wraps `lib.notify`). |

{% hint style="info" %}
`Config.Centers`, `Config.PoliceNpc`, `Config.Prices` and `Config.Business` only **seed** the
database on the first boot. Manage them live with `/regadmin` afterwards.
{% endhint %}

## Commands

| Command | Access | Description |
|---|---|---|
| `/regadmin` | Admin (ACE `vlr.registration.admin` or `Config.Police.adminGroups`) | Open the admin panel — centers, NPC placement, fees, business params, owners. Rename/disable via `Config.Police.adminPanel`. |
| `/regcheck [plate]` | Police / admin | Look up a plate's registration. Rename/disable via `Config.Police.checkCommand`. |
| `/ordreg [plate]` | Police | Order an extraordinary inspection (suspends the registration until it passes). Rename/disable via `Config.Police.orderCommand`. |
| `/reg_devpos` | Dev (`Config.Debug`) | Print the player's coords/heading (and nearest vehicle coords) for placing NPCs/bays. Rename via `Config.Dev.posCommand`. |

## Developer API

### Exports (server)

Bind these from any other resource (MDT, dispatch, insurance, courts, …):

```lua
exports.vlr_vehreg:IsPlateRegistered(plate)            --> boolean (resolves expiry live)
exports.vlr_vehreg:GetRegistrationStatus(plate)        --> 'unregistered'|'pending'|'registered'|'expired'|'suspended'
exports.vlr_vehreg:GetRegistration(plate)              --> document table | nil
exports.vlr_vehreg:GetRegistrationExpiry(plate)        --> epoch seconds | 0
exports.vlr_vehreg:SetRegistrationStatus(plate, status)--> boolean
exports.vlr_vehreg:SuspendRegistration(plate)          --> boolean
exports.vlr_vehreg:RestoreRegistration(plate)          --> boolean
exports.vlr_vehreg:ChangeRegistrationPlate(old, new)   --> boolean (also updates t1ger)
exports.vlr_vehreg:GetMDTVehicleInfo(plate)            --> { plate, status, statusLabel, registered, owner, expires, expiresText } | nil
exports.vlr_vehreg:OrderInspection(plate)              --> boolean (order an extraordinary inspection, suspend until passed)
exports.vlr_vehreg:LiftInspectionHold(plate)           --> boolean (clear a police inspection hold)
exports.vlr_vehreg:HasInspectionHold(plate)            --> boolean
```

### Export (client)

```lua
exports.vlr_vehreg:useDocument(data, slot)  -- opens the full registration document
```

Wired from the `vehicle_registration` ox_inventory item via `client = { export = 'vlr_vehreg.useDocument' }`.

### Event — status changes

Other resources can listen for registration status changes:

```lua
AddEventHandler('vlr_vehreg:server:statusChanged', function(plate, status, info)
    -- status: 'unregistered'|'pending'|'registered'|'expired'|'suspended'
end)
```

This is the supported public event surface. The resource's other network events
(`vlr_vehreg:inspect:*`, `vlr_vehreg:transfer:*`, `vlr_vehreg:admin:*`, `vlr_vehreg:notify`,
etc.) and `lib.callback` callbacks are internal plumbing between its own client and server and
are not part of the integration contract.

## Integrations

### t1ger_mechanic (auto)

* On registration: `AddPlateToOwnedPlates(plate)` so t1ger stops randomizing the vehicle's
  mileage/condition.
* On a passed inspection: `AddServiceHistory(plate, 'Tehnički pregled', mileage, …)`.
* `ChangeRegistrationPlate` also calls `UpdateVehicleDataPlate(old, new)`.
* The inspection reads `GetCorePartHealth`, `DoesVehicleHaveFailuredParts` and
  `GetVehicleMileage` for an authoritative result. Toggle via `Config.Integrations.t1ger`.

### lb-tablet (auto, plug-and-play — no lb-tablet edits)

When lb-tablet is running, the resource writes the registration status straight into its MDT
tables — it ensures a vehicle MDT profile and applies a status **tag** (`REGISTROVANO` /
`REG. ISTEKLA` / `NEREGISTROVANO` / `SUSPENDOVANO`). Tuneable via
`Config.Integrations.lbTablet` (`department`, `tagType`, `appliedType`, `tags`, and the
on-start backfill). The exports and `/regcheck` remain a backend-agnostic path for any other
MDT/dispatch.

### nc-banking (optional)

Set `Config.Payment.society = '<societyId>'` to deposit center income into a society account
(`AddBusinessMoney`). Off by default.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Centers / NPCs don't appear after editing `config.lua` | After first boot the DB is the source of truth — edit centers and NPC positions with `/regadmin`, not `config.lua`. |
| `/regadmin` says you can't open it | Grant the ACE `vlr.registration.admin` or add your group to `Config.Police.adminGroups`. |
| Mechanic refuses to inspect | The office must open a case first (`Config.Inspection.requireCaseFirst`), the car must be parked within the bay radius, the engine off (`requireEngineOff`), and the registering owner must be present (`requireOwnerPresent`). |
| A clean car still fails | The inspection measures real glass/tyres/lights and (with t1ger) core-part health — repair the car. Tune pass `thresholds` and `maxCosmeticDefects` in `Config.Inspection`. |
| Document item does nothing / isn't given | Register `vehicle_registration` in `ox_inventory/data/items.lua` with the `client.export` line and keep `Config.Document.asItem = true`. |
| lb-tablet tags don't match a vehicle | The MDT plate must match the plate lb-tablet reads from your framework's vehicle table; normalize trailing spaces consistently. |
| `/regcheck` not recognised | Ensure your job is in `Config.Police.jobs` (or you're admin); the command can be renamed/disabled via `Config.Police.checkCommand`. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
