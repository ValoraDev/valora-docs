---
description: >-
  Valora Roleplay's signature FiveM loading screen — an industrial-luxe NUI splash
  with a video or animated-static background, a configurable info panel, an audio
  player and framework-aware auto-shutdown. Standalone, no build step, no database.
---

# vlr_loading — Loading Screen

`vlr_loading` is Valora's signature FiveM loading screen. It replaces the default
splash with a fully themed NUI layer — a looping video (or animated static scene),
a multi-tab info panel, a Discord card, an in-screen music player and live FiveM
load-progress readouts. It closes itself cleanly once the player session is ready,
deferring to character-selection resources when present. Everything an owner usually
edits lives in a single open `html/config.js`.

| | |
|---|---|
| **Version** | 1.0.0 |
| **Author** | Valora |
| **Framework** | Qbox / QBCore / ESX player-loaded events are detected for shutdown; works standalone otherwise |
| **Storage** | None (no database, no KVP) |
| **Built on** | Native FiveM `loadscreen` + vanilla HTML/JS (no ox_lib, no build step) |

## Features

* **Two background modes** — a full-screen looping **video** layer with the Valora UI
  on top, or an **animated static** scene; switchable at runtime (`allowPlayerToggle`).
* **Industrial-luxe NUI** — graphite + gold theme, particles, scanlines and grain
  overlays, an animated lion-crest / city scene and code-drawn detailing.
* **Configurable info panel** — any number of right-side tabs of type `text`, `list`
  or `keybinds`, with optional auto-cycling between them.
* **Brand block** — logo, name/suffix, eyebrow, season, location, network label and a
  multi-line motto, all text-driven.
* **Server status widget** — online/offline label, live clock and season badge.
* **Discord card** — invite link, label and note for the community panel.
* **In-screen audio player** — playlist with autoplay, random/first start, volume,
  loop, and next/previous/play/pause/seek/mute; supports remote URLs or local files.
* **Live load progress** — maps the FiveM loading events (data files, map load, init
  functions, ready) to friendly, localized status labels.
* **Keyboard shortcuts** — bindable keys for tab navigation, music control and the
  background toggle.
* **Framework-aware shutdown** — closes on QBCore / Qbox / ESX *player-loaded* events,
  with a universal network-session fallback and a maximum-wait safety timeout.
* **Character-selection aware** — when `vlr-multicharacter` is running, it waits for
  that resource to explicitly close the screen so the preview scene streams in first.
* **No build step / no DB** — vanilla HTML/JS edited directly; nothing is persisted.

## Dependencies

`vlr_loading` declares **no hard dependencies** in its `fxmanifest.lua` — it is a
standalone loading screen. The integrations below are detected at runtime and are all
optional.

| Dependency | Required | Notes |
|---|---|---|
| Framework (QBCore / Qbox / ESX) | ⬚ Optional | Their *player-loaded* events are used to close the screen at the right moment. Without one, the network-session fallback handles shutdown. |
| `vlr-multicharacter` | ⬚ Optional | If running, the loading screen waits for it to close the screen after its preview scene is ready. |

{% hint style="info" %}
This resource does **not** use ox_lib, ox_inventory, ox_target or oxmysql. If your
server already runs those, nothing changes — they are simply not involved here.
{% endhint %}

## Installation

{% hint style="warning" %}
A server may only run **one** loading screen. Remove or stop any existing loadscreen
resource before enabling `vlr_loading`.
{% endhint %}

1. Copy `vlr_loading` into your server `resources/` folder (we recommend
   `resources/[valora]/vlr_loading`).
2. Remove or stop your previous loading screen resource.
3. Add it to `server.cfg` — **early**, so it is registered as the loadscreen before
   the world starts loading:

```cfg
ensure vlr_loading
```

That's it. There is no SQL to import, no items to register and no framework export to
wire up.

## Configuration

There are two configuration surfaces. The **look and content** of the screen live in
`html/config.js`; the **shutdown timing** lives in the shared `config.lua`.

### Shutdown timing — `config.lua`

| Key | Default | Meaning |
|---|---|---|
| `Config.ManualShutdown` | `true` | The screen is closed manually by the script logic (matches `loadscreen_manual_shutdown 'yes'` in the manifest). Set `false` to disable all manual-close handling. |
| `Config.ShutdownDelay` | `150` | Delay in ms between sending the close message to the NUI and actually shutting the loadscreen down (lets the fade-out play). |
| `Config.MaximumWait` | `15000` | Safety timeout in ms — the screen force-closes after this long even if no player-loaded / session-ready signal arrives. |
| `Config.Debug` | `false` | When `true`, prints the close reason to the console. |

### Appearance & content — `html/config.js`

Everything an owner usually changes is in `window.VLR_LOADING_CONFIG`. Paths are
relative to the `html/` folder. The top-level sections:

| Section | What it controls |
|---|---|
| `language` | UI language tag (e.g. `'bs-BA'`). |
| `brand` | Logo, name/suffix, eyebrow, season, location, network label and the motto block. |
| `theme` | `accent` / `accentBright` colours, `interfaceScale`, and the right-panel `panelWidth`. |
| `serverStatus` | Online flag, online/offline labels, status description, clock and season toggles. |
| `background` | `defaultMode` (`'video'` or `'static'`), `allowPlayerToggle`, the `video` block (source, poster, muted, loop, playbackRate), `overlayOpacity`, and `particles` / `scanlines` / `grain` / `coordinates` effects. |
| `discord` | `enabled`, `url`, `invite`, `label` and `note` for the Discord card. |
| `panel` | `enabled`, panel codes, `autoCycle` / `autoCycleDelay` / `pauseAutoCycleAfterInteraction`, and the `tabs` array. |
| `audio` | `enabled`, `showPlayer`, `autoplay`, `startMode` (`'first'`/`'random'`), `volume`, `loopPlaylist`, labels, and the `tracks` playlist. |
| `loading` | All loading status strings, the hint, and the `eventLabels` mapping for FiveM load events. |
| `controls` | `tooltips`, and the `shortcuts` keymap (previous/next tab, toggle music, previous/next track, toggle background). |

### Panel tabs

`panel.tabs` is an array — add, remove or reorder tabs without touching `app.js`.
Each tab supports `type: 'text'`, `'list'` or `'keybinds'`. Example (a keybinds tab):

```js
{
    id: 'keybinds',
    navigationLabel: 'KOMANDE',
    index: '02',
    eyebrow: 'BRZI PREGLED',
    title: 'GLAVNE KOMANDE',
    description: 'Most important controls after you spawn.',
    type: 'keybinds',
    items: [
        { key: 'TAB', label: 'Inventory' },
        { key: 'F1',  label: 'Radial menu' },
    ],
},
```

### Music

For streamed music, add an entry to `audio.tracks` with a remote `source` URL. For
local music, drop an `.mp3` / `.ogg` into `html/assets/audio/` and reference it with a
relative `source` path. The player supports previous/next track, play/pause, seek,
mute and volume.

## How shutdown works

`vlr_loading` keeps the loadscreen up until it has a reliable "player is in the world"
signal, then fades out:

* It listens for the framework **player-loaded** events: `QBCore:Client:OnPlayerLoaded`,
  `qbx_core:client:onPlayerLoaded` and `esx:playerLoaded`.
* A background thread also polls the **network session** (`NetworkIsSessionStarted` +
  `NetworkIsPlayerActive`) as a universal fallback.
* When `vlr-multicharacter` is `started`, the network-session fallback is **suppressed**
  so the character-selection scene can stream in; that resource (or any script) closes
  the screen by triggering `vlr_loading:client:close`.
* `Config.MaximumWait` is a hard safety timeout — the screen always force-closes after
  it, so a stuck load never traps the player behind the splash.

## Developer API

`vlr_loading` exposes **no exports and no callbacks**. It can be closed from any other
client resource with a single net event.

### Net event — `vlr_loading:client:close`

Trigger this on the client to dismiss the loading screen manually (this is how a
character-selection resource hands control over once its scene is ready):

```lua
-- from any client script
TriggerEvent('vlr_loading:client:close', 'my-resource-ready')
```

| Parameter | Type | Purpose |
|---|---|---|
| `reason` | `string?` | Optional label recorded as the close reason (shown in the console when `Config.Debug` is on). Defaults to `'external-resource'`. |

### Events it listens to

| Event | Source | Effect |
|---|---|---|
| `QBCore:Client:OnPlayerLoaded` | QBCore | Closes the loading screen. |
| `qbx_core:client:onPlayerLoaded` | Qbox | Closes the loading screen. |
| `esx:playerLoaded` | ESX | Closes the loading screen. |
| `vlr_loading:client:close` | Any resource | Closes the loading screen with the given reason. |

## Database

None. `vlr_loading` does not use oxmysql or KVP and stores nothing — there is no table
to import.

## Commands

None. `vlr_loading` registers no chat commands. In-screen keyboard shortcuts (tab
navigation, music and background toggle) are configured under `controls.shortcuts` in
`html/config.js`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Two loading screens / the default one still shows | Only one loadscreen resource can run — stop your previous one and keep only `vlr_loading`. |
| Screen never disappears | Ensure your framework fires its player-loaded event, or rely on the network-session fallback; `Config.MaximumWait` force-closes regardless after its timeout. |
| Screen closes too early with character selection | This is what the `vlr-multicharacter` handling prevents — make sure that resource triggers `vlr_loading:client:close` when its scene is ready. |
| Music doesn't autoplay | Browsers/CEF may block autoplay; the in-screen player still lets the user start it. Check `audio.enabled`, `autoplay` and the `tracks` sources. |
| Video background is blank | Check `background.video.source` path (relative to `html/`) and that the file is included by the manifest `files` block. |
| No console output when closing | `Config.Debug` is `false` by default — set it `true` to log the close reason. |

---

_Need something not covered here? Open a ticket — see
[Support & Updates](../getting-started/support.md)._
