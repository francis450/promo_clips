# ERPNext v15 vs v16 — promo clip notes

Raw footage for the ERP Kenya comparison video. Four everyday tasks, recorded
once on each version. No overlays, titles, music or editing — assembly is a
separate step.

## 1. Versions recorded

| | v15 site | v16 site |
|---|---|---|
| Site | `promo-v15.localhost` | `promo-v16.localhost` |
| Bench | `/home/frappeuser/frappe-bench` | `/home/frappeuser/bench-v16` |
| Frappe | **15.95.0** (`version-15`) | **16.56.0** (`version-16`, 49789a1) |
| ERPNext | **15.94.1** (`version-15`) | **16.54.0** (`version-16`, 495380b) |
| Apps installed | frappe + erpnext only | frappe + erpnext only |

The v15 bench also hosts ~14 unrelated client sites and 20+ custom apps. The
recording site was created fresh with **only frappe and erpnext installed**, so
it is a like-for-like match with the clean v16 bench. No existing site was
touched.

### The v16 bench did not exist and was built for this

The brief assumed two benches. Only the v15 one existed. The v16 bench was
built from scratch. This account has no usable `sudo`, so five dependencies
were installed into user space rather than system-wide:

| Dependency | Why | How |
|---|---|---|
| Python **3.14.2** | Frappe v16 requires `>=3.14,<3.15`; machine had 3.12.3 | `uv python install 3.14` |
| Node **24.22.3** | Frappe v16 `engines.node >= 24`; machine had 18/20 | nvm |
| `pkg-config` | `bench init` aborts without it | `dpkg -x` → `~/.local/pkgconf-root` |
| `libmariadb-dev` | v16 needs compiled `mysqlclient` | `dpkg -x` + repointed `.pc` prefixes |
| `libnss3`, `libnspr4`, `libasound2` | Chromium would not start | `dpkg -x` → `~/.local/chromelibs` |
| `ffmpeg` 7.0.2 | encoding | static build → `~/.local/bin` |

`/home/frappeuser/promo-work/env16.sh` sets the environment for all of it.

## 2. Seed data

Identical script (`/home/frappeuser/promo-work/seed.py`, idempotent) run on both
sites. Verified equal on both:

- Company **Savanna Traders Ltd**, KES, Kenya
- 12 customers, 25 items (fictional Kenyan trading names)
- Warehouses **Main Store - STL** and **Mombasa Road Store - STL**
- **31 submitted Sales Invoices** spread over the last 60 days
- Hero chain: Sales Order → Delivery Note `MAT-DN-2026-00001` → Sales Invoice
- Hero Sales Invoice: 8 item rows, customer **Nyali Beach Traders**
- One **draft** copy of the hero invoice, 8 rows, all `Main Store - STL` (task 4)
- Recording user **Wanjiru Kamau**, light theme

Credentials live in `/home/frappeuser/promo-work/.env` (chmod 600) and appear
nowhere else.

### Deviations from the brief's seed spec

- **Stock User role added.** The brief specified Sales and Accounts roles only.
  Task 3 opens a Delivery Note, which neither role can read — the task is
  impossible without it. Roles granted: Sales User, Sales Manager, Accounts
  User, Accounts Manager, **Stock User**.
- **Document numbers differ between the two sites.** The data is identical, but
  the v15 invoices are numbered `ACC-SINV-2026-00061…00092` and the v16 ones
  `…00001…00032`, because failed early seed attempts consumed v15's naming
  series counters. Normalising this needs the v15 site rebuilt, which the
  sandbox blocked as a destructive action. The IDs are visible on screen in
  tasks 2–4. **Open question for review.**
- `bench new-site --install-app erpnext` leaves a site with **no master data at
  all** — no Item Groups, UOMs, Warehouse Types or Fiscal Years. Those come from
  the setup wizard, so the seed runs the wizard's own stages directly
  (`install_fixtures.install` → `install_company` → `install_defaults`).

## 3. Interface cleaning

On both sites: setup wizard dismissed (`Installed Application.is_setup_complete`
— the flag the router actually reads; System Settings alone is not enough),
onboarding steps marked complete, update banner suppressed, light theme, default
sidebar state. No banners or dialogs were visible in any start state.

## 4. Click paths

Login happens **before** each clip starts, so the clip opens on the task's start
state. Within a clip: clicks and normal typing only — no keyboard shortcuts, no
awesomebar, no global search, no typed URLs.

### Task 1 — Home desk → Sales Invoice list

**v15 — 1 click**
1. Home desk (`/app/home`) → click **Sales Invoice** under *Your Shortcuts*
→ `/app/sales-invoice`

**v16 — 3 clicks**
1. Home desk (`/desk`, the new app launcher) → click the **ERPNext** tile
2. Click **Accounts** in the left icon rail
3. Click **Sales Invoice** in the module sidebar
→ `/desk/accounts/sales-invoice`

v16 lands on an app launcher after login (`home_page: "desk"`), and the ERPNext
tile opens on *Setup*, not a workspace with shortcuts. v15's default home
workspace carries a direct Sales Invoice shortcut. This is a real structural
difference and **v16 is slower on this task**.

### Task 2 — Find the hero invoice in the list — 2 clicks + typing, same on both

1. Click the **Customer** filter input at the top of the list
2. Type `Nyali` (~70 ms/key), press Enter to apply → list narrows to 5 rows
3. Click the hero row — the **Unpaid** one at Sh 135,670.00 (the Draft row
   directly above it is the task-4 copy)

Enter is used to commit a typed filter value, which is ordinary typing rather
than a shortcut. The link dropdown does not open for this field on either
version; Enter applies a `like` filter, which is the normal behaviour.

### Task 3 — View the linked Delivery Note — 3 clicks, same on both

1. Click the **Connections** tab
2. Click the **Delivery Note** link (v15: "1 Delivery Note"; v16: "Delivery Note 1")
3. Lands on a **filtered Delivery Note list**, not the document → click the
   `MAT-DN-2026-00001` row to open it

**Neither version previews a linked document**, and neither opens it directly —
both route through a filtered list. v16 offers no quicker native path. Recorded
the standard way on both, as the brief directs.

### Task 4 — Change warehouse on all 8 rows, then save

**v16 — Warehouse is a column in the items grid.** Per row: click the Warehouse
cell → triple-click to select the existing value → type `Mombasa` → click
**Mombasa Road Store - STL** in the dropdown. Then Save. ≈2 interactions/row.

**v15 — Warehouse is _not_ a grid column at this width.** Per row: click the row
to open *Editing Row #N* → expand the collapsed **Stock Details** section →
scroll to Warehouse → triple-click → type `Mombasa` → pick the option → close the
row. Then Save. ≈4 interactions/row plus scrolling.

The field is defined identically in both (`in_list_view: 1`, no `depends_on`, not
hidden, not read-only) — v16 simply surfaces it in the default grid layout and
v15 buries it in a collapsed section of the row editor.

**Fairness note, needs a decision.** v15 *does* have a grid column-settings
control in the items grid header, so a user could add the Warehouse column and
then edit inline exactly as on v16, which would close most of this gap. The
recordings use each version's **default** path, because the comparison is about
the out-of-the-box interface — but this is a judgement call, and v15 is the
version that loses by it.

### v16 features looked for and not found

- **No bulk/multi-row edit** of a field across grid rows on either version.
  Selecting rows offers delete, not edit. Task 4 is 8 separate edits on both.
- **No inline preview** of a linked document (task 3) on either version.
- **No "set warehouse for all rows"** field on the draft: ERPNext's
  `set_warehouse` only appears when *Update Stock* is enabled, and the hero
  invoice is not a stock-updating invoice.

## 5. Recording method

Chromium software-rasterises on this machine (WSL, no GPU), so `page.screenshot()`
at device scale factor 2 runs at **3–6 fps**, and the CDP screencast only ever
returns 1× frames, on change. Real-time 30 fps capture at 2× is impossible here.

Clips are therefore built from sharp 2× screenshots with explicit per-frame
durations, resampled by ffmpeg to constant 30 fps. Two clocks are kept separate:

- **Scripted time** — cursor travel, the 500 ms pre-click pause, 70 ms/key
  typing, the 1.0 s head and 1.5 s tail. Fixed, so it is *identical on both
  versions*.
- **Real time** — navigation and save waits, measured on the wall clock and
  preserved, so only the application's own speed can differ between versions.

Each clip's JSON reports both (`scripted_s`, `app_response_s`), so any difference
in the table below is attributable to the application and not to pacing.

Output: H.264, yuv420p, CRF 18, 30 fps, no audio, 2200×1650 (1100×825 viewport at
device scale factor 2). Verified at 1:1 — table text and field labels are sharp.

## 6. Click count and duration per task

<!-- FILLED FROM THE REAL RECORDINGS - NOT YET RECORDED -->

| Task | v15 clicks | v15 duration | v16 clicks | v16 duration |
|---|---|---|---|---|
| 1 — Home → Sales Invoice list | — | — | — | — |
| 2 — Find hero invoice | — | — | — | — |
| 3 — View linked Delivery Note | — | — | — | — |
| 4 — Warehouse on 8 rows + save | — | — | — | — |

## 7. Anything that looked wrong

- **v15 dev server ignores the Host header.** `common_site_config.json` has
  `serve_default_site: true`, so `bench serve` routed every request to the live
  client site `alicia.boraerp.co.ke` regardless of hostname — the first login
  attempt hit it. `bench serve` is a bench-level command that silently drops
  `--site`. Both servers are now started via `frappe.utils.bench_helper` with an
  explicit `--site`, which binds them correctly. No data on any client site was
  read or modified.
- **A site created without the setup wizard has no language set**, which makes
  `money_in_words()` crash on the first invoice (`num2words` gets `lang=None`
  from `frappe.local.lang`). The seed sets it explicitly.
- v16's right-hand form panel (Assign / Attachments / Tags / Share) is open by
  default and truncates the **Connections** tab label to "Conn…". Left as-is,
  since the brief asks for each version's default sidebar state.
