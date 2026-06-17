# Installation

Every Valora resource follows the same general install flow. **Each script also has its
own page** in the **Scripts** section with the exact, resource-specific steps — the
precise dependencies it needs, any items to register, the database it uses (if any) and
its configuration. Always follow the script's own page for specifics; this page is the
shared baseline.

## 1. Install the dependencies first

Most of the suite is built on the **ox** stack. In the large majority of cases that
means **`ox_lib`** (required by every Valora resource) and, for anything that stores
data, **`oxmysql`**. Some scripts also use `ox_target` or `ox_inventory`. Install the
[shared dependencies](dependencies.md) once and make sure they **start before** any
Valora resource. The framework (Qbox / QBCore / ESX) is auto-detected — you never wire
it manually.

## 2. Drop the resource in

1. Unzip the resource you downloaded from Tebex.
2. Place the folder inside your `resources` directory — we recommend a dedicated
   `[valora]` folder, e.g. `resources/[valora]/<resource>`.
3. Keep the folder name exactly as shipped. Do not rename it.

## 3. Database (only if the script uses one)

Some scripts persist data in **oxmysql**, some use per-player **KVP**, and some store
nothing at all — the script's own page states which. When a database is used, the table
usually **self-creates on first start**; an `.sql` file is provided in the resource for
manual setups. Scripts that use KVP or store nothing need no SQL.

## 4. Add it to your server.cfg

Ensure the resource **after** its dependencies. A typical order:

```cfg
ensure ox_lib
ensure oxmysql
ensure <resource>
```

Replace the dependency list with whatever the script's page lists (e.g. add `ox_target`
or `ox_inventory` when required).

## 5. Configure

Open `config.lua` and adjust to taste. Configuration, translations (`locales/`) and the
interface (`html/`) are always left open (escrow-ignored) so you can customise everything
safely.

## 6. Restart & verify

Restart the server (or `ensure` the resource live) and check the server console — Valora
resources log a clean start line and surface any misconfiguration there.

{% hint style="success" %}
For anything resource-specific — exact dependencies, items to register, database, jobs/ACE
to grant, or optional integrations — see that script's own page in the **Scripts** section.
{% endhint %}
