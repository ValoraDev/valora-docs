---
description: >-
  Advanced ballistic vest, helmet and armour-plate system built on ox_inventory —
  durability, hit-zones, weapon penetration, crafting & repair stations,
  persistence and a live HUD. Server-authoritative and framework-agnostic.
---

# vlr_armour — Armour System

`vlr_armour` replaces the standard *"use item → instant 100 armour"* with a realistic,
item-driven protection system: **ballistic vests** and **helmets** with their own
durability, **armour plates** and **repair kits**, optional **crafting / repair
stations**, full persistence and a live helmet HUD. All authority lives on the server
and the framework is auto-detected.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | ESX / QBCore / Qbox / ox_core — auto-detected (only needed for job restrictions) |
| **Storage** | oxmysql (`vlr_player_armour` table, self-creating) |
| **Built on** | ox_inventory + ox_lib |

## Features

* **Durability vests** — each tier grants a configurable armour value; the armour you
  lose to bullets is converted back into the item's durability, stored as `ox_inventory`
  metadata so it travels with the item.
* **Realistic hit-zones** — only **torso** hits drain the vest; **arm/leg** hits bypass
  it and go to health; **head** hits are handled by the helmet.
* **Weapon penetration** — per weapon-group `penetration` + `wear`: a pistol barely
  dents a vest, a sniper / AP round bleeds through and tears it up.
* **Helmets** — block lethal **headshots** while they have durability
  (`SetPedSuffersCriticalHits`), reduce residual headshot damage by tier, shake the
  camera on impact, and lose durability per hit until they break.
* **Plates & repair kits** — plates top up your worn vest; repair kits restore the
  first damaged worn piece by priority.
* **Crafting & repair stations** *(optional)* — ox_lib points + context menus with
  skill-checks, ingredient checks and distance/job validation.
* **Job whitelist** — restrict any vest/helmet to jobs/grades, enforced server-side.
* **Realistic visuals** — vest on ped component 9 (overlay over clothes), helmet on head
  prop 0; gender-aware, freemode-safe; the outfit/hat underneath is saved & restored.
* **Persistence** — worn pieces and durability survive relogs; on death they return to
  inventory or are lost, per config.
* **Anti-cheat aware** — every equip/plate/repair is validated server-side; the
  durability sync path only ever accepts a *decrease*.
* **Live HUD** — circular "pool" showing the helmet's condition only (the native HUD
  already shows the armour bar); 8 positions, no `backdrop-filter`.
* **i18n** — config-driven locales (English + Serbian included), independent of `ox:locale`.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | ✅ | UI, callbacks, skill-checks, points. |
| [ox_inventory](https://github.com/overextended/ox_inventory) | ✅ | Armour pieces are inventory items with metadata-durability. |
| [oxmysql](https://github.com/overextended/oxmysql) | ✅ | Persists the worn vest/helmet. |
| Framework (ESX/QBCore/Qbox/ox_core) | ⬚ Optional | Only for `jobs` / station `allowedJobs`. Auto-detected; works standalone otherwise. |

## Installation

{% hint style="info" %}
**Plug & play.** The only change in `ox_inventory` is **adding the new items** to
`data/items.lua` — the standard step for any item-based script. No core edits, no
`data/utility.lua` changes.
{% endhint %}

### 1. Install the resource

1. Copy `vlr_armour` into `resources/` (we recommend `resources/[valora]/vlr_armour`).
2. Start it **after** its dependencies in `server.cfg`:

```cfg
ensure ox_lib
ensure oxmysql
ensure ox_inventory
ensure vlr_armour
```

### 2. Add the items → `ox_inventory/data/items.lua`

Open `install/items.lua` and paste every block into the items table in
`ox_inventory/data/items.lua`. All items are new `vlr_*` names plus two crafting
materials (`steel`, `rubber`) — see [Items](#items) below.

### 3. Add inventory images → `ox_inventory/web/images/`

Add a PNG (recommended 100×100) for each new item. The PNGs ship in the `install/`
folder:

```
vlr_vest_light.png      vlr_helmet_light.png      vlr_plate.png
vlr_vest_tactical.png   vlr_helmet_tactical.png   vlr_repair_kit.png
vlr_vest_heavy.png      vlr_helmet_heavy.png      steel.png   rubber.png
```

### 4. Database (optional)

The table `vlr_player_armour` **self-creates on first start**. Importing
`install/vlr_armour.sql` is only needed for manual setups.

### 5. Test it

```
/giveitem <id> vlr_vest_heavy 1
/giveitem <id> vlr_helmet_tactical 1
/giveitem <id> vlr_plate 5
/giveitem <id> vlr_repair_kit 2
```

## Items

Paste these from `install/items.lua`. The `client.event` is what wires each item to the
script — keep the item names matching `config.lua`.

| Item | Label | Stack | Trigger (`client.event`) |
|---|---|---|---|
| `vlr_vest_light` | Light Vest | ✗ | `vlr_armour:useVestItem` |
| `vlr_vest_tactical` | Tactical Vest | ✗ | `vlr_armour:useVestItem` |
| `vlr_vest_heavy` | Heavy Vest | ✗ | `vlr_armour:useVestItem` |
| `vlr_helmet_light` | Light Helmet | ✗ | `vlr_armour:useHelmetItem` |
| `vlr_helmet_tactical` | Tactical Helmet | ✗ | `vlr_armour:useHelmetItem` |
| `vlr_helmet_heavy` | Heavy Helmet | ✗ | `vlr_armour:useHelmetItem` |
| `vlr_plate` | Armor Plate | ✓ | `vlr_armour:usePlateItem` |
| `vlr_repair_kit` | Armor Repair Kit | ✓ | `vlr_armour:useRepairKitItem` |
| `steel`, `rubber` | Crafting materials | ✓ | — |

Optional **broken variants** (`vlr_vest_light_broken`, …) are only needed if you enable
`Config.BrokenItems` — see section C of `install/items.lua`.

## Configuration

Everything lives in the open `config.lua`. The key sections:

### Vests & helmets

Each tier in `Config.Vests` / `Config.Helmets` defines:

| Field | Meaning |
|---|---|
| `armor` | Max body-armour value (0–100) shown on the GTA armour bar. |
| `damageReduction` | Fraction of a torso (vest) / headshot (helmet) hit's damage stopped (0–1), before weapon penetration. |
| `useTime` | Equip animation duration (ms). |
| `returnOnDeath` | Return the piece to inventory on death instead of losing it. |
| `removeOnBreak` | On break: `true` = destroy, `false` = keep at 0% for station repair. |
| `skillCheck` | `false`, or an ox_lib skill-check definition. |
| `component` / `prop` | The visual (see [Visual armour](#visual-armour)). |
| `jobs` | *(optional)* whitelist — `{ 'police' }` or `{ police = 2 }` for a min grade. |
| `blockHeadshotKills`, `durabilityLossPerHit`, `shakeOnHit` | Helmet-only headshot behaviour. |

Shipped tiers: **Light / Tactical / Heavy** vests (armour 50 / 75 / 100) and helmets.

### Damage model

`Config.Damage` is what makes the system realistic:

* `zones` — GTA bone ids classified as **head** / **limb**; everything else is treated
  as **torso** (the vest absorbs it).
* `groups` / `weaponOverrides` — per weapon-group (and per-weapon) `penetration`
  (fraction that pierces the vest even while it has durability) and `wear` (how fast
  durability drops). Pistol → low; sniper/AP → high.
* `combatWindow` — how long (ms) the per-frame detector runs after a hit (performance).

Set `Config.Damage.enabled = false` to fall back to the simple model (armour value ⇄
durability, GTA handles the split; no zone/penetration logic).

### Plates, repair kits, crafting & stations

* `Config.Plates` — durability restored to the **worn vest**.
* `Config.RepairKits` — `restore` + `targets` priority (`{ 'vest', 'helmet' }`).
* `Config.Recipes` — craft/repair recipes (`ingredients`, `repairIngredients`, skill-check).
* `Config.Stations` — optional craft/repair benches with coords, marker, blip, and
  `allowedJobs`. **Set `Config.Stations = {}` to disable stations entirely.**

### Other

* `Config.BrokenItems` — hand a `<item>_broken` variant on break (must exist in
  ox_inventory; safely falls back otherwise).
* `Config.HUD` — `enabled`, `position` (8 options), `hideWhenEmpty`, `hideInVehicle`.
  Shows the **helmet** condition only.
* `Config.AutoRemoveHelmetIfTakenOff` — auto-unequip the helmet if a clothing menu
  strips the prop (no invisible head protection).
* `Config.Commands` — names for the remove-vest / remove-helmet commands.
* `Config.Locale` — `'en'` or `'sr'` (add more `locales/<code>.json`); independent of
  `ox:locale`.

### Visual armour

Visuals are **on by default**: the vest is drawn on ped **component 9** (overlay over
clothes; set `id = 11` for a full torso garment), the helmet on **head prop 0**.
Male/female drawables are separate and only freemode peds get the visual. To tune
drawable ids for your clothing pack:

1. set `Config.Debug = true`,
2. in-game: `/vlrvest 9 4` and `/vlrhelmet 0 123` (each prints the valid range),
3. copy the good numbers into config,
4. `/vlrresetprops` to clear the preview, then set `Config.Debug = false`.

## How the mechanics work

* **Torso hit** → the vest reduces the damage by its tier's `damageReduction`, lowered
  by the weapon's `penetration`; `wear` drains durability; at 0 the vest breaks.
* **Arm/leg hit** → the vest is bypassed; the hit goes to health, durability untouched.
* **Head hit** → while the helmet has durability the ped suffers no *critical* (lethal)
  headshot; residual damage is reduced by the helmet tier; each hit drains durability;
  at 0 headshots are lethal again.
* **Anti-desync guard** → if a clothing menu strips the helmet prop, the helmet
  auto-unequips and is returned (no invisible protection); the vest overlay is
  re-asserted if wiped. Debounced and paused while a menu is focused.

## Commands

| Command | Description |
|---|---|
| `/remvest` | Take off the worn vest (returned with remaining durability). |
| `/remhelmet` | Take off the worn helmet (returned with remaining durability). |
| `/vlrvest <component> <drawable> [texture]` | **Dev** (Config.Debug) — preview vest drawable ids. |
| `/vlrhelmet <propId> <drawable> [texture]` | **Dev** (Config.Debug) — preview helmet prop ids. |
| `/vlrresetprops` | **Dev** (Config.Debug) — clear the visual preview. |

Player command names are configurable in `config.lua` → `Config.Commands`.

## Developer API

### Export — `GetArmour(source)` *(server)*

Read a player's currently worn armour (or `false`):

```lua
local data = exports.vlr_armour:GetArmour(playerId)
-- data = {
--   vest   = { item = 'vlr_vest_heavy',  durability = 87.5 } | false,
--   helmet = { item = 'vlr_helmet_light', durability = 40.0 } | false,
-- }
```

Use it from a HUD, medic script or anticheat to read protection without touching the client.

### Server callbacks (`lib.callback`)

Driven internally by the client; listed for integration awareness:
`vlr_armour:getState`, `:equipVest`, `:equipHelmet`, `:remove`, `:usePlate`,
`:useRepairKit`, `:craft`, `:repairItem`.

### Net events

| Event | Direction | Purpose |
|---|---|---|
| `vlr_armour:sync` | client → server | Report a durability **decrease** (increases are rejected). |
| `vlr_armour:break` | client → server | A piece broke; server applies the break rule. |
| `vlr_armour:onDeath` | client → server | Apply death handling (return/lose per config). |

## Database

Single self-creating table:

```sql
CREATE TABLE IF NOT EXISTS `vlr_player_armour` (
    `identifier`        VARCHAR(64) NOT NULL,
    `vest_item`         VARCHAR(50) DEFAULT NULL,
    `vest_durability`   FLOAT NOT NULL DEFAULT 0,
    `helmet_item`       VARCHAR(50) DEFAULT NULL,
    `helmet_durability` FLOAT NOT NULL DEFAULT 0,
    `updated_at`        TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`identifier`)
);
```

## Troubleshooting

| Symptom | Fix |
|---|---|
| Using a vest does nothing | Items not pasted (step 2), or `vlr_armour` started before `ox_inventory`. |
| No item icons | Add the PNGs (step 3) to `ox_inventory/web/images`. |
| Armour jumps to 100 / resets | Another armour source is active (old `armour` item or framework armour) — stop handing it out / disable framework armour persistence. |
| Vest/helmet not visible | Non-freemode ped, or wrong drawable id for your pack — tune with the dev commands. |
| "You cannot do that right now" | You're dead, ragdolled, shooting, or already mid-action — by design. |
| Anti-cheat flags health/armour | The damage model adjusts health/armour client-side — whitelist `vlr_armour`, or set `Config.Damage.enabled = false`. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
