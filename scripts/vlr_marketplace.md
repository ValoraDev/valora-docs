---
description: >-
  Player-owned market stands and street vendors for FiveM — admins place selling
  spots in game, players buy them, stock them from their own inventory, set prices
  and earn through an NPC vendor that sells even while they are offline, governed by
  a configurable state-tax lifecycle. Server-authoritative, framework-agnostic, with
  a themeable NUI.
---

# vlr_marketplace — Player Marketplace

`vlr_marketplace` turns market stands into a small-business loop: an admin drops a
"for sale" vendor in game, a player buys it, names it, stocks it with items from their
own inventory, sets per-unit prices, and an NPC sells to other players **24/7** — as
long as the owner keeps the **state tax** paid. Everything is validated on the server,
the framework is auto-detected, and the whole UI is driven by one locale file.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | QBox (primary) · QBCore · ESX — auto-detected |
| **Storage** | oxmysql (5 self-creating tables) |
| **Built on** | ox_lib + oxmysql + an inventory (ox_inventory or qb-inventory) |

## Features

### For players

* **Buy a selling spot** — walk up to a "for sale" vendor, name the stand, pay, and it
  is your business. Owned-stand count is capped by `Config.Economy.maxOwnedSpots`.
* **Stock from your own inventory** — choose items, quantities and a per-unit price;
  stock lives server-side, not in your pockets.
* **NPC sells for you 24/7** — customers buy through a shop UI even while you are
  offline; earnings (minus sales tax) land in the stand's register, and offline sales
  are summarized on your next login.
* **Register and withdrawals** — withdraw earnings any time from the in-shop finances tab.
* **State tax with real consequences** — pay per period, prepay several periods ahead,
  or enable **auto-pay from the register**. Miss it and the stand goes due → suspended
  → repossessed by the state.
* **Rename or give up the stand** — configurable rename price and refund percentage; goods
  on display return to you (or to your claim box if your inventory is full).

### For admins

* **In-game admin panel** (`/marketadmin`) — create, edit, delete spots with no config
  editing: capture your current position, pick the vendor NPC model, an optional stall
  prop, an idle scenario, the purchase price, tax amount/interval and slot count.
* **Manage owners** — evict owners, read each stand's live status and register balance,
  and jump to a spot's coordinates.
* **Per-stand item policy** — each stand is *sell anything* / *only these* (whitelist) /
  *all except these* (blacklist), set with a searchable item and weapon picker pulled
  live from the inventory registry.
* **Live global ban list** — ban or unban items server-wide from the panel (stored in the
  DB), overriding every stand on top of the hard-coded `Config.Items.alwaysBlock` list.

### Tech

* **Server-authoritative** — every purchase, stock move, withdrawal and tax payment is
  validated server-side; stock decrements are atomic (guarded UPDATEs) to avoid dupes;
  NUI inputs are sanitized and rate-limited.
* **Distance-driven world** — vendor NPCs, props and blips spawn within `Config.SpawnDistance`
  and despawn outside it.
* **ox_target support with automatic TextUI + [E] fallback.**
* **Item metadata transfers 1:1** — a worn item placed on display is exactly what the
  buyer receives (no "sell broken, get new" laundering); weapons (serial, ammo,
  components) carry over when `Config.Items.allowWeapons` is on.
* **Optional Discord webhook logging** — stand purchases, sales, tax suspensions,
  repossessions and admin actions.
* **Localization** — ships in **English**, fully translatable via the open `locales/` files.
  More languages are planned over time. One JSON file translates every string including the
  entire NUI, independent of the `ox:locale` convar.
* **Themeable NUI** — graphite design, one accent color (`Config.UI.accent`) recolors
  everything; no `backdrop-filter`.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | Yes | Callbacks, notifications, TextUI. |
| [oxmysql](https://github.com/overextended/oxmysql) | Yes | Persists spots, stock, claims, bans and sales. |
| [ox_inventory](https://github.com/overextended/ox_inventory) | Yes* | *Inventory backend — `qb-inventory` is also supported and auto-detected. One of the two is required at runtime. |
| [ox_target](https://github.com/overextended/ox_target) | Optional | Used when running; falls back to TextUI + `[E]` automatically. |
| Framework (QBox / QBCore / ESX) | Optional | Auto-detected. QBox is the primary target; only used for identity, money, character names and admin groups. |

{% hint style="info" %}
`fxmanifest.lua` formally declares only **ox_lib** and **oxmysql** as dependencies.
An inventory resource (`ox_inventory` or `qb-inventory`) is required at runtime but is
resolved dynamically, so it is not listed in the manifest. `ox_target` and the framework
are detected at runtime as well — with no framework present the resource still runs
standalone for identity.
{% endhint %}

## Installation

{% hint style="success" %}
**Plug & play.** No core edits and no item registration are required — stands sell items
that already exist in your inventory. Admins create every spot in game.
{% endhint %}

### 1. Install the resource

1. Copy `vlr_marketplace` into `resources/` (we recommend `resources/[valora]/vlr_marketplace`).
2. Start it **after** ox_lib, oxmysql, your framework and your inventory in `server.cfg`:

```cfg
ensure ox_lib
ensure oxmysql
ensure ox_inventory
ensure vlr_marketplace

add_ace group.admin vlr.marketplace.admin allow
```

### 2. Database (optional)

The five tables **self-create on first start**. Importing `install/vlr_marketplace.sql`
is only needed for manual setups that prefer explicit DDL.

### 3. Grant admin access

A player is treated as an admin when they hold the ACE permission **or** belong to one of
the framework groups. Pick either (or both) in `config.lua` → `Config.Admin`:

* **ACE** — `add_ace group.admin vlr.marketplace.admin allow` in `server.cfg`.
* **Framework groups** — `Config.Admin.groups = { 'admin', 'god' }` works with QBox/QBCore
  permission groups out of the box.

### 4. Create your first spot (in game)

1. Type `/marketadmin`.
2. Click **New spot**, stand where the vendor should stand (facing the customer) and
   capture your current position.
3. Pick the NPC, an optional stall prop, the purchase price, the tax amount and interval,
   then **Create**. The spot is live for everyone instantly.

## Database

`vlr_marketplace` uses **oxmysql**. All tables self-create on first start; the bundled
`install/vlr_marketplace.sql` is optional.

| Table | Purpose |
|---|---|
| `vlr_market_spots` | One row per selling spot: label, shop name, position, ped/prop/scenario, prices, tax state, owner, register ledger, item policy. |
| `vlr_market_stock` | Items currently on display per spot (item, label, price, amount, metadata). |
| `vlr_market_claims` | Goods and money owed to a player (from repossession or full-inventory returns), collected with `/marketclaim`. |
| `vlr_market_banned_items` | Live global ban list managed from the admin panel. |
| `vlr_market_sales` | Sales ledger; unseen rows back the offline-sales summary shown on login. |

## Configuration

Everything lives in the open `config.lua`. The key groups:

### General

| Key | Meaning |
|---|---|
| `Config.Locale` | UI/notification language (defaults to `'en'`; add a `locales/<code>.json` to translate into any language). |
| `Config.Debug` | Verbose development prints. |

### UI / theme — `Config.UI`

`accent` (one hex recolors the whole UI — Valora gold by default), `currency` symbol, and
`itemImagePath` (where the NUI loads item images from; defaults to ox_inventory's images,
with a letter placeholder fallback). Two helper functions, `Config.Notify` and
`Config.FormatMoney`, are also defined here and can be swapped for your own notify resource
or money format.

### World / interaction

| Key | Meaning |
|---|---|
| `Config.SpawnDistance` | Range at which vendor NPCs/props/blips spawn and despawn. |
| `Config.Interaction` | `useTarget` (ox_target when available) and interaction `distance`. |
| `Config.Stall` | `forwardOffset` — how far a stall prop sits in front of the NPC. |
| `Config.Blips` | `mode` (`'always'` or `'proximity'` + `proximityDistance`) plus per-state `free` / `owned` / `active` blip styling. |

### Economy — `Config.Economy`

`paymentMethods` (cash/bank), `salesTaxPercent`, `maxOwnedSpots`, `refundPercent`,
`renamePrice`, per-unit price bounds (`minItemPrice`/`maxItemPrice`), `maxStockPerSlot`,
and stand-name length bounds (`shopNameMin`/`shopNameMax`).

### State tax — `Config.Tax`

Governs the lifecycle (see below): `checkEvery` (sweep interval, seconds), `graceHours`,
`repossessHours`, `maxPrepaidIntervals`, `autoPayDefault`, and whether unpaid
`returnStockOnRepossess` / `returnLedgerOnRepossess`.

```
PAID ──(due date passes)──► DUE          stand open, owner warned
     ──(+ graceHours)─────► SUSPENDED    NPC refuses customers
     ──(+ repossessHours)─► REPOSSESSED  spot is for sale again;
                                         goods + register → /marketclaim
```

### Sellable items — `Config.Items`

`alwaysBlock` (hard list never sellable on any stand), `allowWeapons` (global gate for
trading weapons at all), `defaultMode` (the policy a freshly created stand starts with —
`'all'` / `'whitelist'` / `'blacklist'`; per-stand policy is set in the admin panel) and
`allowMetadata`.

### Commands & permissions — `Config.Commands`, `Config.Admin`

`Config.Commands` renames the `admin` and `claim` commands. `Config.Admin` holds the admin
`ace` permission and `groups`, the searchable `itemPicker` toggle, the admin-panel
suggestion lists (`pedModels`, `stallProps`, `scenarios`) and the `defaults` pre-filled in
the create-spot form.

### Discord webhook — `Config.Webhook`

Optional logging (`enabled`, `url`, `username`, `color`) for purchases, sales, suspensions,
repossessions and admin actions.

### Optional overrides

Commented stubs at the bottom of `config.lua` let you override the auto-detected
`Config.GetIdentifier`, `Config.GetCharacterName` and `Config.IsAdmin` if the bridge
defaults do not fit your setup.

## Commands

| Command | Who | Description |
|---|---|---|
| `/marketadmin` | Admins | Open the admin panel (create/edit/delete spots, evict owners, manage global bans). |
| `/marketclaim` | Everyone | Collect goods/money owed to you (after repossession or a full-inventory return). |

Both names are configurable in `config.lua` → `Config.Commands`.

## Developer API

All exports are **server-side**.

### `GetSpot(spotId)`

Returns a defensive read-only copy of a spot's data, or `nil` if unknown.

```lua
local spot = exports.vlr_marketplace:GetSpot(spotId)
```

### `GetPlayerSpots(identifier)`

Returns an array of spot ids owned by a character identifier.

```lua
local ids = exports.vlr_marketplace:GetPlayerSpots(citizenid)
```

### `IsSpotOwner(source, spotId)`

Returns `true` if the given player is the owner of the spot.

```lua
local isOwner = exports.vlr_marketplace:IsSpotOwner(source, spotId)
```

### Server events

Fired server-side with `TriggerEvent`; listen with `AddEventHandler`.

| Event | Signature | Purpose |
|---|---|---|
| `vlr_marketplace:spotPurchased` | `(source, spotId)` | A player just bought a stand. |
| `vlr_marketplace:itemSold` | `(buyerSource, spotId, item, amount, total)` | An item was sold from a stand. |

```lua
AddEventHandler('vlr_marketplace:spotPurchased', function(source, spotId) end)
AddEventHandler('vlr_marketplace:itemSold', function(buyerSource, spotId, item, amount, total) end)
```

### Callback

The NUI talks to the server through a single ox_lib callback, `vlr_marketplace:api`
(a method + params router). It is internal plumbing, listed only for integration awareness.

## Adding a language

Copy `locales/en.json` to `locales/<code>.json`, translate the values, and set
`Config.Locale = '<code>'`. Every player-facing string — notifications, target/TextUI
labels and the entire NUI — comes from that one file, independent of `ox:locale`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `"no supported inventory found"` in console | Start `ox_inventory` (or `qb-inventory`) **before** `vlr_marketplace`. |
| Vendor NPC does not spawn | Invalid ped model name; pick a valid model in the admin panel. Bad models are logged when `Config.Debug = true`. |
| Item images missing in the UI | `Config.UI.itemImagePath` points at ox_inventory's images by default — adjust it if yours live elsewhere. Items without an image show a letter placeholder; nothing breaks. |
| `/marketadmin` does nothing | The player lacks admin access — grant the ACE (`add_ace group.admin vlr.marketplace.admin allow`) or add their group to `Config.Admin.groups`. |
| Weapons cannot be put on display | `Config.Items.allowWeapons = false` blocks all weapons regardless of per-stand policy — set it to `true`. |
| An item still can't be sold on a stand | Check `Config.Items.alwaysBlock`, the live global ban list (admin panel → Global bans), and that stand's own whitelist/blacklist policy. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
