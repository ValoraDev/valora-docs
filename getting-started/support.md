# Support & Updates

## Getting help

The fastest way to get support is the **Valora Dev Discord**. After purchase you get
access to a private buyers area with a ticket system:

* **Pre-sales** — questions before you buy.
* **Bug reports** — something not working as documented.
* **Installation** — help getting a script running.
* **Other** — anything else.

When opening a ticket, please include:

1. The resource name and **version** (see `fxmanifest.lua` → `version`).
2. Your framework (Qbox / QBCore / ESX) and core dependency versions.
3. The **full** server console output around the error (`txAdmin` console or `server.log`).
4. Steps to reproduce.

## Updates

Releases are announced in the Discord **#updates** channel. Each release lists the
changes for that version. Always read the changelog before updating — occasionally an
update needs a database migration or a `config.lua` change, which will be called out.

## Licensing

Valora resources are distributed via Tebex with CFX asset escrow. The following are
always left **open** so you can fully customise without touching protected code:

* `config.lua` — all settings.
* `locales/*.json` — translations.
* `html/*` — the interface (NUI).
* `README.md` / install docs.

Everything else (client/server logic) is encrypted. If you need a hook that isn't exposed,
ask on Discord — most needs are already covered by the resource's **exports and events**
(listed on each script's page).
