# Installation

Every Valora resource installs the same way. These steps apply to **all** scripts in
the suite — individual pages only add what is unique to that resource.

## 1. Install the dependencies first

Make sure the [shared dependencies](dependencies.md) are already on your server and
**start before** any Valora resource. At minimum that means `ox_lib` and (for most
scripts) `oxmysql`.

## 2. Drop the resource in

1. Unzip the resource you downloaded from Tebex.
2. Place the folder inside your `resources` directory — we recommend a dedicated
   `[valora]` folder, e.g. `resources/[valora]/vlr_armour`.
3. Keep the folder name exactly as shipped (e.g. `vlr_armour`). Do not rename it.

## 3. Import the database (if required)

If the resource ships an `.sql` file (or an `install/` folder), import it into your
database **once** before first start. Pages that need this say so explicitly. Scripts
that store nothing, or use per-player KVP, need no SQL.

## 4. Add it to your server.cfg

Ensure the resource **after** its dependencies:

```cfg
ensure ox_lib
ensure oxmysql
ensure vlr_armour
```

## 5. Configure

Open `config.lua` and adjust to taste. This file is always left open (escrow-ignored)
so you can edit everything safely. Translations live in `locales/*.json` and the
interface in `html/`.

## 6. Restart & verify

Restart the server (or `ensure` the resource live) and check the server console — Valora
resources log a clean start line and surface any misconfiguration there.

{% hint style="success" %}
That's it. For anything resource-specific — items to register, jobs/ACE to grant, or
integrations to enable — see that script's own page in the **Scripts** section.
{% endhint %}
