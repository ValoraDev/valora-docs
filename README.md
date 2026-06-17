---
description: Official documentation for the Valora script suite for FiveM.
---

# Welcome to Valora

**Valora** is a suite of premium, performance-first FiveM resources sharing one design
language and one technical standard. Every script is built on the **ox_lib** baseline,
auto-detects your framework (**qbox → qb → esx**), and ships server-authoritative by
default — so it is safe, fast, and consistent across your server.

This site is the single source of truth for installing, configuring and integrating
every Valora resource.

{% hint style="info" %}
**New here?** Start with [Installation](getting-started/installation.md) and
[Dependencies](getting-started/dependencies.md), then open the page for the script you
bought.
{% endhint %}

## What's inside

| Area | Where to go |
|---|---|
| Get a script running | [Getting Started → Installation](getting-started/installation.md) |
| Shared dependencies (ox_lib, oxmysql, …) | [Getting Started → Dependencies](getting-started/dependencies.md) |
| Per-script setup, config, exports & events | The **Scripts** section in the sidebar |
| Help, bug reports, updates | [Support & Updates](getting-started/support.md) |

## The Valora standard

Every resource in the suite follows the same rules, so once you know one you know them all:

* **Framework-agnostic** — a single bridge auto-detects qbox, qb-core or ESX. No edits needed.
* **ox baseline** — `ox_lib` for notifications, callbacks and context menus; `oxmysql` for storage where applicable.
* **Server-authoritative** — money, items and permissions are always validated on the server. The client is never trusted.
* **Open config** — `config.lua`, `locales/*.json` and the NUI (`html/*`) are always left open (escrow-ignored) so you can fully theme and translate.
* **Localised** — English (`en.json`) plus additional locales, with full key parity.

## Need a license?

Browse and purchase the suite at **[store.valoragaming.com](https://store.valoragaming.com)**.
Buyers get a private support channel on the Valora Dev Discord — see
[Support & Updates](getting-started/support.md).
