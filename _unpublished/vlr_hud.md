---
description: >-
  Signature industrial-luxe HUD for FiveM — player vitals, identity & money,
  compass/location, a repositioned square minimap, a full vehicle cluster
  (speedometer, seatbelt, indicators, nitro), a clickable car-control panel,
  a custom navigation/waypoint panel and an in-vehicle CarPlay dashboard with
  music & shared playlists. Built on ox_lib, framework auto-detected, server-authoritative.
---

# vlr_hud — Valora HUD

`vlr_hud` is Valora's signature **industrial-luxe HUD**: a single graphite + gold NUI that
replaces the default interface with a player vitals cluster, an identity/money block, a
top-center compass, a cleanly repositioned square minimap, and a complete vehicle cluster.
It also ships three opt-in panels — **Car Control**, **Navigation** and an in-vehicle
**CarPlay dashboard** (music + cross-player playlists) — all reachable from one quick-menu
hub. Built on `ox_lib`, framework auto-detected (Qbox / QBCore / ESX), and authoritative on
the server for everything that matters.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | Qbox / QBCore / ESX — auto-detected (`Config.Framework`); runs standalone too |
| **Storage** | Per-PC KVP (settings/pins/volume) + optional oxmysql (shared playlists only) |
| **Built on** | ox_lib |

## Features

* **Player vitals cluster** — segmented 3D-perspective bars for health, armor, stamina,
  hunger, thirst and stress; survival needs read from the framework and auto-hide when the
  framework doesn't expose them. Bars can fade out when resting and reveal on change.
* **Identity & money block** — character name, job + grade, server id, cash and bank,
  live online count and a configurable server name.
* **Society / organisation money** — a job's boss sees the org balance on the identity
  block; "boss" is decided **server-side** by job + grade, never trusted from the client.
  The balance source tries Renewed-Banking, qb-banking, qb-management and nc-banking.
* **Location / compass** — top-center cardinal strip + heading, nearest street/crossing,
  district/zone name and an in-game clock.
* **Repositioned minimap** — the engine radar is reframed into a clean square at the
  bottom-left with a matching mask (no diamond artifact) and a locked Valora frame.
  Includes an in-game drag-to-move/resize editor that persists per-PC.
* **Vehicle cluster** — speed (metric/imperial), RPM, gear, fuel, engine health, lights
  and optional siren readout. Fuel auto-adapts to the common fuel resources (ox_fuel,
  LegacyFuel, cdn-fuel, ps-fuel, nd-fuel) with a native fallback.
* **Seatbelt system** — buckle/unbuckle with sound, periodic warning, and a realistic
  ejection-through-the-windshield model for unbelted occupants on a crash.
* **Turn signals & hazards** — keybound indicators with optional drive-lights mirroring.
* **Nitro readout** — display-only, reads an external nitro resource's statebag/event.
* **Optional odometer** — shows vehicle mileage when a mileage resource (default
  `t1ger_mechanic`) is running.
* **Car Control panel** — a clickable Valora schematic for doors, windows, seats and lock,
  with optional routing of the lock through your vehicle-keys resource.
* **Navigation / waypoint panel** — drop a waypoint by coordinates, save personal pins
  (per-PC KVP), or route to the nearest place from a curated catalog (gas/hospital/mechanic/
  atm). Includes an in-world gold beam marker with an off-screen arrow and live distance.
* **CarPlay dashboard** — an in-vehicle app grid; the Music app is live (HTML5 audio in
  the NUI), Nav re-uses the waypoint panel, other tiles are placeholders.
* **Music & shared playlists** — curated stations/tracks out of the box, optional direct-URL
  / resolver / opt-in YouTube paste-to-play (server-resolved), and **cross-player public/
  private playlists** when oxmysql is available (graceful per-PC fallback otherwise).
* **Quick-menu hub** — one key opens a launcher listing every player/vehicle app; features
  self-register their hub entry, so new apps appear with no hub edits.
* **Theming & i18n** — graphite by design with a single `accent` recolor (Valora gold by
  default); config-driven locales (English + Bosnian ship) independent of the `ox:locale`
  convar; vanilla-JS NUI, no build step, no `backdrop-filter`.

## Dependencies

| Dependency | Required | Notes |
|---|---|---|
| [ox_lib](https://github.com/overextended/ox_lib) | Yes | NUI bus, callbacks, keybinds, notifications. The only hard dependency. |
| [oxmysql](https://github.com/overextended/oxmysql) | Optional | **Only** for cross-player shared playlists. Without it, playlists fall back to per-PC KVP (private only). |
| [ox_inventory](https://github.com/overextended/ox_inventory) | Optional | Server-side phone-item check for the minimap (`requirePhone`). |
| Framework (Qbox / QBCore / ESX) | Optional | Job/grade, identity, survival needs, society money. Auto-detected; HUD runs standalone otherwise. |
| Fuel resource (ox_fuel / LegacyFuel / cdn-fuel / ps-fuel / nd-fuel) | Optional | Auto-detected for the fuel gauge; falls back to the native fuel level. |
| Banking (Renewed-Banking / qb-banking / qb-management / nc-banking) | Optional | Source for society/organisation money. |
| `t1ger_mechanic` (or your mileage resource) | Optional | Vehicle odometer; the widget hides when absent. |

{% hint style="info" %}
The only entry in `fxmanifest.lua` `dependencies{}` is **ox_lib**. Everything else is a
soft integration: the HUD detects whatever you're running and adapts, and hides any widget
whose data source is missing.
{% endhint %}

## Installation

### 1. Install the resource

1. Copy `vlr_hud` into `resources/` (we recommend `resources/[valora]/vlr_hud`).
2. Start it **after** ox_lib (and after oxmysql if you want shared playlists):

```cfg
ensure ox_lib
ensure oxmysql        # optional — only for shared playlists
ensure vlr_hud
```

### 2. Set your framework & language (optional)

Open `config.lua` and adjust `Config.Framework` (default `'auto'`), `Config.Locale`
(`'en'` / `'ba'`) and `Config.UI.accent`. The defaults work plug-and-play for most servers.

### 3. Database (optional — shared playlists only)

If `oxmysql` is started and `Config.CarPlay.Music.playlists.shared = true`, the three
playlist tables **self-create on first start** — no manual SQL import is required. If you
leave oxmysql out, the script silently uses per-PC KVP and hides the Public playlist tab.

{% hint style="warning" %}
There is **no items step and no SQL import** to run by hand. `vlr_hud` registers no
ox_inventory items, and the only database it touches (shared playlists) is self-creating.
{% endhint %}

## Database

`vlr_hud` is KVP-first. Most state — minimap layout, saved waypoint pins, music volume and
private playlists — is stored **per-PC via the resource KVP store** (no database).

The **only** SQL usage is the optional shared-playlist feature, which requires `oxmysql`.
When enabled, three tables are created automatically on first start:

```sql
CREATE TABLE IF NOT EXISTS `vlr_cp_playlists` (
    `id`         INT UNSIGNED NOT NULL AUTO_INCREMENT,
    `identifier` VARCHAR(64)  NOT NULL,
    `owner_name` VARCHAR(64)  NOT NULL DEFAULT 'Unknown',
    `name`       VARCHAR(64)  NOT NULL,
    `is_public`  TINYINT(1)   NOT NULL DEFAULT 0,
    `likes`      INT UNSIGNED NOT NULL DEFAULT 0,
    `created_at` TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    KEY `idx_owner`  (`identifier`),
    KEY `idx_public` (`is_public`, `likes`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `vlr_cp_playlist_songs` (
    `id`          INT UNSIGNED  NOT NULL AUTO_INCREMENT,
    `playlist_id` INT UNSIGNED  NOT NULL,
    `url`         VARCHAR(2048) NOT NULL,
    `title`       VARCHAR(255)  NOT NULL,
    `artist`      VARCHAR(255)  NOT NULL DEFAULT 'Unknown',
    `art`         VARCHAR(2048) NULL,
    `duration`    INT UNSIGNED  NOT NULL DEFAULT 0,
    `position`    INT UNSIGNED  NOT NULL DEFAULT 0,
    PRIMARY KEY (`id`),
    KEY `idx_pl` (`playlist_id`, `position`),
    CONSTRAINT `fk_song_pl` FOREIGN KEY (`playlist_id`)
        REFERENCES `vlr_cp_playlists`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS `vlr_cp_playlist_likes` (
    `playlist_id` INT UNSIGNED NOT NULL,
    `identifier`  VARCHAR(64)  NOT NULL,
    `created_at`  TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (`playlist_id`, `identifier`),
    CONSTRAINT `fk_like_pl` FOREIGN KEY (`playlist_id`)
        REFERENCES `vlr_cp_playlists`(`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

Ownership is verified server-side by per-character identifier; all queries are parameterized.

## Configuration

Everything lives in the open `config.lua`. The key groups:

### Global

| Key | Meaning |
|---|---|
| `Config.Locale` | UI/notification language (`'en'`, `'ba'`, or any `locales/<code>.json`). Independent of `ox:locale`. |
| `Config.Debug` | Verbose prints + on-screen diagnostics for development. |
| `Config.Framework` | `'auto'` detects qbox > qb > esx; force one if auto-detect misfires. |

### UI / theme

`Config.UI` — `accent` (highlight color, gold by default), `currency` symbol, `scale`
(master HUD scale) and `units` (`'metric'` km/h or `'imperial'` mph). `Config.Notify` and
`Config.FormatMoney` are overridable helper functions.

### Widgets & visibility

* `Config.Widgets` — master toggles for the `vitals`, `info`, `location`, `minimap` and
  `voice` clusters.
* `Config.Keybinds` — `toggleHud` (command `hud`, no default key) and `focusHud`
  (`+vlrfocus`, default `LMENU`, hold to reveal exact numbers).
* `Config.AutoHide` — hide the HUD on the pause menu and/or while dead.

### Vitals

`Config.Vitals` — per-bar `enabled` + warning thresholds for health, armor, stamina,
hunger, thirst and stress; `fadeWhenResting` / `restHoldMs`, the `tilt.rotateY` perspective
amount, and the cluster `position`.

### Identity, society & location

* `Config.Info` — which identity fields show (name, job, id, cash, bank, society, online
  count), `serverName`, and an optional `getPlayerId` override.
* `Config.Society` — `enabled`, per-job `bossGrades` + `defaultBossGrade`, and a server-side
  `getBalance(source, jobName)` function that tries the common banking systems.
* `Config.Location` — `compass`, `street`, `zone`, `clock`, `showSeconds`.

### Minimap

`Config.Minimap` — `vehicleOnly`, the `rect` nudge (x/y/scale/fit/vfit), the in-game
`editor` (command, default `minimap`), per-setup `frameAdjust` pixel trim, and `requirePhone`
(server-authoritative phone-item gate with an item name/list or a custom `check` function).

### Vehicle

`Config.Vehicle` — `enabled`, `refresh`/`slowEvery` tick rates, `dynamicEngine`, `showLights`,
`showSiren`, a `getFuel(veh)` function, `lowFuel` warning, `electricModels`, the `seatbelt`
sub-table (warning, sound, damage model, eject tuning), `indicators`, `nitro`, `carControl`
(panel + lock routing), `odometer` (resource + `get(veh)`) and `keybinds` (seatbelt /
indicators / hazard).

### Navigation (waypoint)

`Config.Waypoint` — `enabled`, opener `key` (default `K`), `maxSaved` pins, the in-world
`marker` (beam height/rings/pulse, `maxDistance`, `offscreenArrow`, distance chip), and a
curated `categories` place catalog (gas / hospital / mechanic / atm) you can edit freely.

### CarPlay & music

`Config.CarPlay` — `enabled`, opener `key`, `inVehicleOnly`, the `apps` grid, and `Music`:
the URL `source` (`'curated'` / `'direct'` / `'resolver'`), `stations`, `tracks`, the opt-in
`youtube` resolver chain (cobalt / piped / innertube / rapidapi / scrape) and the `playlists`
settings (`shared`, `pageSize`, `maxPerUser`, `maxTracks`, `nameMax`, `likeCooldownMs`).

### Hub

`Config.Hub` — `enabled` and the quick-menu `key` (default `U`), the primary opener for all
player/vehicle apps.

## Commands

Most openers are **ox_lib keybinds** bound in `FiveM > Settings > Key Bindings` (the hub on
`U`, navigation on `K`, etc.); the console commands below are the player-facing fallbacks and
tools.

| Command | Description |
|---|---|
| `/hud` | Toggle the entire HUD on/off. Name configurable via `Config.Keybinds.toggleHud.command`. |
| `/minimap` | Open the in-game minimap editor (drag to move, corners to resize, Save). `/minimap reset` restores the default. Name configurable via `Config.Minimap.editor.command`. |
| `/vlrhub` | Open the quick-menu hub (console fallback for the `U` keybind). |
| `/vlrwaypoint` | Open the Navigation panel (console fallback for the `K` keybind). |
| `/vlrcarplay` | Open the CarPlay dashboard (console fallback). |
| `/vlrhud` | Diagnostic — prints HUD state and probes your inventory to reveal the real phone item name (useful when configuring `requirePhone`). |

`+vlrfocus` / `-vlrfocus` back the hold-to-focus keybind (default `LMENU`); seatbelt,
indicators and hazard keys are registered as rebindable ox_lib keybinds.

## Developer API

### Client exports

```lua
-- Show/hide the whole HUD (other Valora resources can drive it).
exports.vlr_hud:setVisible(visible)         -- boolean

-- Read whether the HUD is currently active.
local active = exports.vlr_hud:isVisible()  -- returns boolean

-- Send a raw NUI message to the HUD (advanced — action + data table).
exports.vlr_hud:sendMessage(action, data)

-- Force the seatbelt state on the HUD/model.
exports.vlr_hud:setSeatbelt(state)          -- boolean

-- Externally show/hide the repositioned minimap.
exports.vlr_hud:displayMinimap(show)        -- boolean
```

### Net events (client)

| Event | Direction | Purpose |
|---|---|---|
| `vlr_hud:client:displayMinimap` | server → client | Show/hide the minimap for a player (`show` boolean). |

### Internal events

`vlr_hud:refresh` (re-read/redraw all widgets after a player (un)loads) and
`vlr_hud:waypoint:open` (open the navigation panel) are fired internally between modules;
listed for integration awareness.

### Server callbacks (`lib.callback`)

Driven by the NUI/client; listed for awareness. Identity/society are resolved server-side.

| Callback | Purpose |
|---|---|
| `vlr_hud:getServer` | Live online/max count + server name for the identity block. |
| `vlr_hud:getSociety` | Authoritative org balance — returned only if the player is genuinely their job's boss. |
| `vlr_hud:hasItem` | Server-side inventory check (phone gate); accepts one name or a list. |
| `vlr_hud:phoneDiag` | One-shot inventory probe behind `/vlrhud`. |
| `vlr_hud:resolveAudio` | Server-side YouTube/URL → direct-audio resolver for CarPlay music. |
| `vlr_hud:cp:capabilities` | Reports whether shared (oxmysql) playlists are available. |
| `vlr_hud:cp:getMyPlaylists` / `:getPublicPlaylists` | Paged playlist lists. |
| `vlr_hud:cp:createPlaylist` / `:updatePlaylist` / `:deletePlaylist` | Playlist CRUD (ownership-checked). |
| `vlr_hud:cp:addTrack` / `:removeTrack` | Track CRUD. |
| `vlr_hud:cp:setPublic` / `:toggleLike` | Visibility + likes (per-player like cooldown). |

## Troubleshooting

| Symptom | Fix |
|---|---|
| HUD doesn't appear at all | `vlr_hud` started before `ox_lib`, or the framework hasn't loaded the player yet. Check the start order in `server.cfg`. |
| Hunger / thirst / stress bars missing | The framework doesn't expose those needs (or you're standalone) — those bars hide automatically. Set `enabled=false` to silence them. |
| Society money not showing | You must be the boss of your job (grade ≥ `bossGrades[job]` / `defaultBossGrade`), and `Config.Society.getBalance` must point at the bank that actually holds your org money. |
| Fuel gauge wrong / stuck | Your fuel resource isn't covered — edit `Config.Vehicle.getFuel` to call your resource's export. |
| Odometer never shows | The mileage resource (`Config.Vehicle.odometer.resource`, default `t1ger_mechanic`) isn't running, or its export returns nil. |
| Minimap looks off / has a seam | Open `/minimap`, drag/resize/fit and Save; fine-tune with `frameAdjust`. `/minimap reset` restores the baseline. |
| Public playlists tab missing | `oxmysql` isn't started or `playlists.shared = false` — the script falls back to per-PC private playlists by design. |
| Phone gate not letting me see the minimap | Run `/vlrhud` in-game; it prints the real phone item name to put in `Config.Minimap.requirePhone.item`. |
| YouTube links won't play | The public resolvers (piped/innertube/scrape) rot frequently — run a self-hosted **cobalt** instance and set `youtube.cobalt.url`. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
