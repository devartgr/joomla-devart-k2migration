# Operator Guide (minimal)

For administrators running **DevArt K2 Migration** on a clean Joomla 6 destination.
Source is legacy Joomla 3 / K2 (SELECT-only). Do not edit the live legacy site from this tool.

First path on a clone: [`quick-start.md`](quick-start.md).
Full acceptance criteria: [`acceptance-a-d.md`](acceptance-a-d.md)
(current verified: **0.3.0** stable — A–D transferred from α202 on that SHA only).
Engine notes: [`convert-engine-design.md`](convert-engine-design.md).
Machine-readable run sheet: [`run-record.example.json`](run-record.example.json)
(template keyed to **0.3.0** stable — copy and retarget for any 0.3.1+ cut).

---

## Prerequisites

- Clean Joomla **6+**, PHP **8.3+**, modern MariaDB/MySQL
- Destination is a **clone** until acceptance A–D pass
- Source DB user with **SELECT** only on the legacy database (grants + product SQL guard)
- Enough disk for staging + native content/UCM + `images/` copy + `images/k2migration/YYYY/MM/` XL + media backups
- Browser batches are the supported shared-hosting path; CLI is an optional accelerator

Host performance profile (component options): `auto` / `shared` / `vps` / `dedicated`.
Optional override: **Batch time budget (ms)** — integer ≥ 500 (0 = profile default). Caps one HTTP tick.

---

## Safe order

1. **Database** — connect → Full (or incremental) import into staging  
2. **Media** — site `images/` then K2 XL (incremental resume supported)  
3. **Preparation** — resolve ID collisions / remaps; freeze Convert-owned creates  
4. **Convert** — dry-run (same generation) → apply tags → items → menus  
5. **Check** — verify completeness / ownership sentinels  
6. **Rollback** — scoped Undo only when needed (never as a substitute for Check)

Never skip Preparation when destination already has conflicting IDs.
Never treat helper cleanup success as product Undo.

---

## Admin UI vs CLI

| Work | Prefer |
|------|--------|
| Connection test, small Full import, Prep UI, sampling Check | Admin |
| Large Import, Convert densify PREPARE, long Convert | Browser batches; CLI optional acceleration |
| **Initial** site `images/` / K2 media on huge trees (tens of GB) | **CLI required** — see below |

Console plugin: `plg_console_devartk2migration` (enabled on package install).
Production packages omit `devartk2migration:integration-test` (repo overlay via
`scripts/run-integration-convert.php` only).

### Numeric CLI flags (integers only)

| Flag | Meaning | Example |
|------|---------|---------|
| `--batch=N` | Rows/files/statements per tick | `--batch=50` |
| `--time-limit=N` | Batch wall budget (≥ 500). Bare integer = **milliseconds**; also accepts `30s` / `5000ms` | `--time-limit=30000` or `--time-limit=30s` |
| `--convert-job=N` | Convert rollback scoped to job id `N` | `--convert-job=42` |
| `--sample=N` | Pilot sample size | `--sample=25` |

Useful commands:

```bash
php cli/joomla.php devartk2migration:preflight --json
php cli/joomla.php devartk2migration:import --incremental --json
# Initial / large media: raise PHP memory so huge directory listings do not fail.
# Resume the SAME paused job if the hub shows MEDIA_REQUIRES_CLI.
php -d memory_limit=-1 cli/joomla.php devartk2migration:media --target=images --json
php -d memory_limit=-1 cli/joomla.php devartk2migration:media --target=k2 --json
# Daily incremental (after first completed run; K2 resumes after last cursor):
php -d memory_limit=-1 cli/joomla.php devartk2migration:media --target=images --incremental --json
php -d memory_limit=-1 cli/joomla.php devartk2migration:media --target=k2 --incremental --json
php cli/joomla.php devartk2migration:convert --phase=tags --dry-run --json
php cli/joomla.php devartk2migration:convert --phase=items --batch=50 --time-limit=5000 --json
# Scoped Convert undo (preferred):
php cli/joomla.php devartk2migration:rollback --target=convert --convert-job=42 --json
# Global Convert ledger undo (all convert-* runs — use with care):
php cli/joomla.php devartk2migration:rollback --target=convert --json
```

Exit codes: `0` OK · `1` fail · `2` completed with errors · `3` interrupted · `4` busy (lease).

Convert densify **PREPARE** runs as durable browser batches. A permanent memory
admission refusal still requires more PHP memory or a smaller corpus; CLI does
not bypass that safety check.

### Recommended host settings for large migrations

These are operational recommendations, not hard installation requirements.
For destinations near 200,000–250,000 articles/assets, use:

- PHP `memory_limit = 512M` (`256M` may be sufficient for smaller corpora)
- PHP `max_execution_time = 120`
- PHP `max_input_time = 120`
- PHP `upload_max_filesize = 128M`
- PHP `post_max_size = 128M` or higher
- PHP-FPM `request_terminate_timeout = 120s` or higher
- MySQL/MariaDB `max_allowed_packet = 64M` or higher

Browser batches remain intentionally much shorter than these ceilings. Raising
timeouts is additional host tolerance; it is not a replacement for resumable
jobs. Asset densify still performs a memory admission check before loading its
in-memory graph.

Convert `run_generation` pins **input** fingerprints (staging / idmap / prep / media / source).
It does **not** include destination `#__content` watermarks — Convert’s own article writes must not
self-invalidate the run.

### Identity / completeness certainty (declared)

| Surface | Contract |
|---------|----------|
| Destination admission (cached preflight) | Bounded owners digest: COUNT + MAX(id/modified) + SUM/XOR CRC32(`id:alias:catid:state`) |
| SQL-file source fingerprint | **Sampled** — mtime/ctime/inode/size + head\|mid\|tail (not whole-file hash) |
| Import expected COUNT | Failure → `expected_unverified` / not verified completed |
| Live DB source fingerprint | host/port/db/user/prefix (no password) |

Do not treat sampled dump identity as exact byte-for-byte proof. Re-run Full import after
replacing a dump when certainty matters.

---

## Rollback scope

| Command | Effect |
|---------|--------|
| `--target=convert --convert-job=ID` | Undo ledger for `run_id=convert-{ID}` only |
| `--target=convert` (no `--convert-job`) | Undo **all** Convert ledger runs — not a single job |
| `--target=media` / `images` | Media ledger Undo (LIFO / ownership) |
| `--target=staging` / `audit` | Component staging / audit cleanup |

Prep **Create** (users/categories/menus) is destination state — Convert Rollback does **not** remove it.

Prep **Delete Undo** restores destination rows from snapshots. Item Undo also replays captured
native side-effects (tags, workflow stage, frontpage, article asset ACL). Category Delete Undo
replays captured category asset ACL. **Prep Create Undo is not delivered** — Create writes are
immediate destination state; reverse them with Prep Delete (or manual cleanup), not a Create Undo.

---

## Prep create vs Convert Rollback

Preparation **Create** for users, categories, and menus writes destination rows immediately.
There is **no Prep Create Undo** receipt. **Convert Rollback** undoes Convert journal/ledger work
only — it does **not** remove Prep-created users/categories/menus. Plan destination cleanup
separately (Prep Delete where eligible, or A delete for menu conflicts).

Menus are **optional** and never block Convert tags/articles. The product goal for this step
is **legacy public URLs** (correct menu **aliases** + htaccess) — prefer rebuilding navigation
yourself with those aliases rather than force-migrating the full K2 tree. Menu Create after A/B
(if used): preserved IDs + source aliases; unresolved `com_k2` links are created **unpublished**;
`template_style_id` is forced to `0`. Staging **Home** menus are excluded entirely. Menu Remap
writes idmap only (staging PKs stay); diagnosis excludes remapped conflicts — use **Undo remap**.
HTTP Create refuses above 2000 missing rows — use Convert → menus (batched) for larger trees.

---

## Identity, remaps, and assets

- Preserve source IDs by default; Prep remaps are explicit (`idmap` rule=remap).
- Convert does not silently rewrite colliding IDs.
- Convert-owned create remaps are **frozen** for the run generation — do not invent remaps mid-run.
- **Never copy legacy `#__assets` rows** from the source database.
- Destination article/category assets are created on the **destination** during Convert
  (`Content::store` or bulk `INSERT` of matching `com_content.article.{id}` names with
  placeholder nested-set, then `AssetNestedRebuild` densify). This is **not** a legacy
  `#__assets` import. Compensations: ownership fingerprint, nested-set verify, Check.
  See [`convert-engine-design.md`](convert-engine-design.md) (bulk assets note).

---

## Media destinations

- Site images: incremental copy into destination `images/`
- K2 XL: `images/k2migration/YYYY/MM/` with basename `md5('Image' . id) . '_XL.jpg'`
- Folder dates from `k2_items.created`, not filesystem mtime
- Verify backups before Undo; corrupt/newest-owned backups block unsafe restore
- Large site-images listing / stale purge / empty-dir cleanup respect the HTTP tick
  wall-clock: incomplete work resumes next tick; a stalled listing (same cursor across
  consecutive deferred ticks) pauses with `requires_cli` — resume the **same** Media
  job on CLI (`devartk2migration:media`), not by hammering the browser Run button
  (HTTP returns `MEDIA_REQUIRES_CLI` and must not spawn a second run)
- Ambiguous destination bytes (match neither journaled original nor published digest)
  pin a durable `recovery_required` ledger flag — files stay untouched, auto-resume
  does not retry forever; resolve with Media Undo or manual inspection

---

## Pilot gate (before full Convert)

Default: `convert_require_pilot=1` (component Options → Convert).

1. Convert **sample** (admin or `devartk2migration:convert --sample=N`) —
   allowed when common readiness is green; a missing/failed/unpromoted pilot
   does **not** block Sample (checklist still shows the pilot row)
2. **Evaluate** native checks (asset / workflow / tags / images / UCM)
3. **Promote** while the staging fingerprint is unchanged
4. Only then run full tags → items → menus

Full Convert (HTTP + CLI) refuses without PASS + Promote + matching fingerprint
(`assertFullConvertAllowed` / hub `$fullAllowed`). Setting
`convert_require_pilot=0` is an **explicit bypass for disposable test sites
only**; each allowed full Convert start records `convert_pilot_bypass_at` /
`convert_pilot_bypass_count` in component params. Readiness shows the pilot row
as warn when bypass is active.

Frontend SEF article URLs are **not** generated by this component — after pilot
PASS, spot-check sample articles in the browser (native `com_content` view) as
part of acceptance A.

```bash
# Promote only on the sample path (standalone --promote without --sample is ignored):
php cli/joomla.php devartk2migration:convert --sample=25 --check --promote --json
```

---

## Convert safety (site offline + densify)

- Keep the destination **Site Offline** (Global Configuration) for the whole Convert /
  densify window. Bulk article create inserts placeholder `#__assets` rows
  (`lft`/`rgt` = 0) until end-of-job densify; ACL that relies on nested-set is
  unreliable until densify finishes.
- Do **not** publish or use the destination as a public site during Convert.
- After an interrupt, timeout, or a legacy HTTP `paused` checkpoint: **resume densify
  on the same job** from the Convert progress panel — do not start a second Convert.

```bash
# Optional CLI acceleration for the same Convert items checkpoint:
php cli/joomla.php devartk2migration:convert --phase=items --json
# Tagged densify / tags phase if that job is paused:
php cli/joomla.php devartk2migration:convert --phase=tags --json
# Paused Convert rollback densify:
php cli/joomla.php devartk2migration:rollback --target=convert --json
```

Browser Resume uses durable `snapshot → repair → number → flush → verify` ticks.
Reloading or retrying the same job is safe; never start a second logical run.

---

## Interrupt and resume

- Kill / drop mid-batch: start again; resume uses journal / ledger / media publish / PREPARE `flush_offset`
- Do not start a second overlapping Convert of the same phase while a lease is held
- After interrupt, re-run dry-run only when you need a fresh summarize — apply continues from checkpoints
- Cancelled nested preflight under dry-run is **not** auto-replaced — restart the dry-run explicitly
- Convert densify interrupt: see **Convert safety** above (browser resume of the same paused job;
  CLI is optional acceleration)

---

## Source safety

- Live DB mode: SELECT-only grants on the legacy server
- SQL file mode: dump path on the destination host; product SQL guard rejects write statements
- Never expose filesystem paths or DB passwords in frontend responses

---

## Out of scope (do not expect)

K2 comments, attachments, galleries, video, extra fields → `com_fields`, Finder reindex,
native Joomla 3 `#__content` migration, modules/templates migration.

---

## Recording a run

Always capture package ZIP name, SHA-256, installed `pkg_` / `com_` versions, target host,
PHP/DB/collation, profile, and operator notes. Acceptance A–D requires a full run record per letter
(plain text in [`acceptance-a-d.md`](acceptance-a-d.md) or JSON from [`run-record.example.json`](run-record.example.json)).
