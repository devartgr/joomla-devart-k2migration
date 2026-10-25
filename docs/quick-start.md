# Quick start (clone destination)

First-time operator path on a **clean Joomla 6 clone** (not the live legacy site).
Flags below are verified against `ConvertCommand` / `ImportCommand` / `MediaCommand`
in this repository (Sonnet J3).

Longer detail: [`ops-guide.md`](ops-guide.md). Acceptance: [`acceptance-a-d.md`](acceptance-a-d.md).

---

## 0. Install

1. Install `pkg_devartk2migration_v*.zip` via Joomla admin or CLI
   `extension:install --path=…`.
2. Confirm `pkg_devartk2migration` / `com_devartk2migration` / console plugin
   versions match the ZIP under test.
3. Enable `plg_console_devartk2migration` if not already enabled by the package.

---

## 1. Connect source (admin)

Administrator → Components → DevArt K2 Migration → Database.

- Host / DB / user / prefix for the **legacy** Joomla 3 / K2 database
- Source DB user: **SELECT only**
- Password is stored encrypted in component params today; optional non-persisting
  override on import CLI: `--password-env=ENV_VAR_NAME`

Run the **admin connection test**, then Full (or incremental) import into staging.
(`devartk2migration:preflight` is Convert item readiness over staging — run it
**after** Import + Preparation, not as a source connection test.)

CLI import:

```bash
php cli/joomla.php devartk2migration:import --json
# resume / new rows only:
php cli/joomla.php devartk2migration:import --incremental --json
# optional: php cli/joomla.php devartk2migration:import --password-env=DKM_SOURCE_PASSWORD --json
```

---

## 2. Media

```bash
php cli/joomla.php devartk2migration:media --target=images --incremental --json
php cli/joomla.php devartk2migration:media --target=k2 --incremental --json
```

Prefer CLI for large trees. HTTP ticks honour wall-clock; stalled listing may
pause with `requires_cli` — resume the same Media job on CLI (do not re-click Run).

---

## 3. Preparation

Preparation and Convert open immediately. On a large site their counts run in
short browser batches and stay on the page after that; use **Refresh diagnosis**
when staging or the destination has changed.

In admin Preparation: resolve category/item/tag/menu collisions (delete disposable
destination rows or remap staging IDs). Convert does **not** silently rewrite
colliding IDs.

Prep Delete Undo restores snapshots (items: native relations incl. frontpage
ordering + modified; categories: asset ACL). There is **no Prep Create Undo**.

Optional Convert staging readiness (titles/aliases/categories — not DB connect):

```bash
php cli/joomla.php devartk2migration:preflight --json
```

---

## 4. Pilot sample (before full Convert)

Verified convert pilot flags (`devartk2migration:convert`):

| Flag | Role |
|------|------|
| `--sample[=N]` | Pilot of N items (default **200** if `N` omitted); forces items path |
| `--check` | After sample, run sample native verifies (+ optional hub Check) |
| `--promote` | After PASS sample+check, unlock full Convert (fingerprint gate) |
| `--rollback-run` | After sample (+ optional check), rollback tag+item sample jobs |
| `--dry-run` | Planner + preflight only (no destination writes) |
| `--phase=tags\|items\|menus` | Full-phase apply (not used with `--sample`) |
| `--batch=N` | Plan chunk size per tick |
| `--time-limit=N` | Wall budget **ms** (integer ≥ 500; CLI default 30000) |
| `--json` | NDJSON progress + final JSON |

Typical pilot on a clone:

```bash
# Dry-run planner (no writes):
php cli/joomla.php devartk2migration:convert --phase=items --dry-run --json

# Sample convert + verify (default N=200):
php cli/joomla.php devartk2migration:convert --sample --check --json

# Smaller pilot:
php cli/joomla.php devartk2migration:convert --sample=25 --check --json

# Unlock full Convert only after PASS:
php cli/joomla.php devartk2migration:convert --sample --check --promote --json

# Tear down sample jobs if you need a clean retry:
php cli/joomla.php devartk2migration:convert --sample=25 --check --rollback-run --json
```

Do **not** start full Convert until pilot gate allows it (admin hub / `--promote`).

---

## 5. Full Convert (after promote)

```bash
php cli/joomla.php devartk2migration:convert --phase=tags --batch=50 --time-limit=5000 --json
php cli/joomla.php devartk2migration:convert --phase=items --batch=50 --time-limit=5000 --json
php cli/joomla.php devartk2migration:convert --phase=menus --batch=50 --time-limit=5000 --json
```

If an older package left densify paused, install the current package and resume the
**same** job from the browser progress panel. PREPARE and flush are durable HTTP
batches; CLI remains an optional accelerator, not a requirement.

---

## 6. Check and scoped Rollback

```bash
php cli/joomla.php devartk2migration:verify --json
# Preferred scoped undo:
php cli/joomla.php devartk2migration:rollback --target=convert --convert-job=ID --json
```

Exit codes: `0` OK · `1` fail · `2` completed with errors · `3` interrupted · `4` busy.

---

## 7. Real-clone rehearsal (operator)

Use a disposable clone of the production destination (or a restored backup), not
the live site. Record one run sheet per artifact:

1. Note package filename + SHA-256 + installed `pkg_` / `com_` versions.
2. Run steps 1–6 above (pilot → promote → full → verify).
3. Fill [`run-record.example.json`](run-record.example.json) (retarget package/SHA;
   do not reuse a PASS from an older ZIP).
4. Only then consider VPS / promotion decisions.

Harness readonly scenario IDs (product snapshot): [`expected-assertions.harness.json`](expected-assertions.harness.json).

Longer clone rehearsal: [`real-data-rehearsal.md`](real-data-rehearsal.md).
Release cut steps: [`release-checklist.md`](release-checklist.md).
Evidence verify: `php scripts/verify-evidence.php` (after
`scripts/run-integration-convert.php --json` → `qa/evidence/<version>/integration-<sha12>.json`).

---

## 8. Host sizing (indicative)

Conservative planning bands for a **clean Joomla 6 destination** during migration
(Site Offline recommended for Convert densify). Numbers are planning floors, not
guarantees — measure on your clone.

| Corpus (articles) | PHP memory_limit | MySQL max_allowed_packet | Free disk (staging + images + backups) | Prefer |
|-------------------|------------------|--------------------------|----------------------------------------|--------|
| ≤ 100k | ≥ 256MB | ≥ 16MB | ≥ 2× source `images/` + DB dump | Shared/browser supported; CLI optional |
| ~300k | ≥ 512MB | ≥ 64MB | ≥ 2–3× source media + staging | Browser batches supported; CLI optional |
| ~500k–1M | ≥ 1GB | ≥ 64MB | Large SSD headroom + backup window | Dedicated; long CLI windows |

Also: disable/ignore versionable + actionlog noise on disposable clones; keep
`convert_require_pilot=1` until sample PASS + Promote.

Where host controls are available, use PHP `max_execution_time=120`,
`max_input_time=120`, `upload_max_filesize=128M`, `post_max_size=128M`, and
PHP-FPM `request_terminate_timeout=120s` or higher. These ceilings supplement
the resumable browser jobs; they do not replace batching.

Asset densify cost grows with `#__assets` size (placeholder nested-set until
end-of-job densify). On a healthy local/VPS host the densify **rebuild_assets**
slice alone is typically **seconds** at ~100k–300k (see the table in §9 —
~2s / ~6s on the Herd benchmark host). Full Convert wall time is much larger
(create + densify + media + import). Keep the site offline until densify
completes; do not confuse densify seconds with total Convert minutes.

---

## 9. Measured Convert rates (local Herd benchmarks)

From [`convert-asset-benchmarks.md`](convert-asset-benchmarks.md) on
`joomla6_tests` (do not invent other numbers). **Bulk** path is the production
Convert create path:

| Size | Bulk rate (arts/s) | Notes |
|------|--------------------|-------|
| 5k | ~272 | rebuild_assets ~0.1s |
| 20k | ~271 | rebuild_assets ~0.4s |
| 100k | ~282 | rebuild_assets ~2s |
| 300k | ~255 | rebuild_assets ~6s; wall_rate ~253 |

Older row-at-a-time `Content::store` path fell to ~14 arts/s at 300k — not an
operator process. Estimate full Convert from a **pilot sample** wall time ×
`(total / sample_N)`, then add densify + media + import separately.

---

## 10. Exit codes and job statuses

| CLI exit | Meaning | Operator action |
|----------|---------|-----------------|
| `0` | OK | Continue |
| `1` | Fail | Read errors/stats; fix; do not blind-retry recovery |
| `2` | Completed with errors | Inspect failed/conflicts; decide promote vs repair |
| `3` | Interrupted | Resume **same** job if still running/paused |
| `4` | Busy (lease) | Wait; do not start a second logical run |

| Job / stats signal | Action |
|--------------------|--------|
| Legacy `paused` Convert densify | Resume the same job in the browser after updating |
| `paused` + Media `requires_cli` / MEDIA_REQUIRES_CLI | `media --target=…` same job on CLI |
| `recovery_required` (Convert) | Stop; inspect destinations/receipts |
| `recovery_required` (Media) | Ambiguous bytes untouched; Undo or manual |
| Prep `dest_id_occupied` / needs_repair | Resolve in Preparation before Convert |

---

## 11. Incremental cutover plan

1. **Freeze window** on the legacy site (or accept a short delta after final Full).
2. Final **Full** (or last Incremental) Database import + Media incremental resume.
3. Preparation remaps frozen — do not change idmap mid-cutover.
4. Pilot → Promote → full Convert on the clone while **Site Offline**.
5. After densify + Check PASS: optional `:sync` / Check sync for last editorial
   deltas (tombstones/edits) — not a substitute for a planned freeze.
6. DNS / reverse-proxy cutover only after VPS A–D on the **same** ZIP SHA.
7. Keep the legacy site read-only until you accept the new URLs.

---

## 12. SEO / aliases / redirects

- Convert preserves source aliases when Prep remaps allow it; colliding aliases
  must be resolved in Preparation (no silent rewrite of colliding IDs).
- This component does **not** generate Joomla SEF redirects or Smart Search indexes.
- After menus match, use the Convert hub **.htaccess helper** for legacy K2
  `/item/ID-alias` → new `com_content` paths (manual install into site root).
- Alias changes after remap require your own redirect plan — record them in the
  run sheet.
