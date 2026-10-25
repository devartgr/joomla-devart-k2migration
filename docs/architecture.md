# DevArt K2 Migration — Architecture

Post-alpha75 audit synthesis and locked roadmap:
[`docs/audits/2026-10-11-synthesis.md`](audits/2026-10-11-synthesis.md).

**Permanently out of scope** (do not audit or implement): K2 comments,
attachments, galleries, video, extra fields → `com_fields`, Finder reindex,
and native J3 `com_content` migration. See `AGENTS.md` / `PROJECT_STATUS.md`.

## Repository Layout

`source/` is the only development source:

- `source/package/` — outer Joomla package manifest, `script.php`, package languages.
- `source/extensions/component/` — `com_devartk2migration` source.
- `source/extensions/plugins/` — `plg_console_devartk2migration` (CLI import/media/preflight/convert/rollback).
- `source/extensions/modules/` — unused (modules deferred).

Generated packages belong only under `builds/`.

## Package Composition

Package: `pkg_devartk2migration`

Declared extensions:

- `com_devartk2migration`
- `plg_console_devartk2migration` (console group; enabled on package install)

Supported administrator languages:

- en-GB
- el-GR

The package manifest is the source of truth for the package name, version,
inner archive names, and inner archive folder.

Outer package filename pattern:

`pkg_devartk2migration_v{version}.zip`

## Product Architecture

DevArt K2 Migration is an administrator-only Joomla 6 component that prepares
and executes migration from a live legacy **Joomla 3 / K2** site into a clean
Joomla 6 destination.

Core principles:

1. The legacy site remains completely unchanged.
2. Source database access is SELECT-only at the MySQL account level and
   application-enforced as read-only.
3. Destination content uses native Joomla Articles; K2 is not installed on J6.
4. Original IDs are preserved for articles, categories, tags, users, groups,
   access levels, menu items and modules.
5. Joomla assets are created correctly by the destination, never copied from
   legacy asset rows.
6. The component is independent from DevArt News, Gantry and Audacity.

### Explicit non-goal — native J3 com_content

This product is **not** a general Joomla 3 → Joomla 6 content migrator.
Native legacy `#__content` / `com_content` article migration is unrelated to
DevArt K2 Migration and will not be implemented here. Source articles come
from **K2** (`k2_items` and related). Joomla structural tables (users, groups,
viewlevels, menus, …) may be staged to support Prep/Convert — that does not
imply migrating native J3 articles.

### IMPLEMENTED (Phase 1 — 0.1.0-alpha1)

- Installable package and component shell (Joomla 6 / PHP 8.3)
- Secure source connection configuration (host, port, database, user, password,
  prefix, optional source root path)
- Connection test with required table detection
- Application-level SQL read-only guard
- Source inventory aggregates for K2 and Joomla structures
- Module inventory grouped by type
- Menu inventory grouped by link family
- Destination inventory of reserved IDs
- Informational ID collision reporting
- Media / batch architecture documentation and path helpers

### IMPLEMENTED (Phase 2 — 0.2.0-alpha1)

- Source mode: live SELECT-only DB **or** absolute-path SQL dump → staging
- Resumable SQL staging import + staging rollback
- Full ID + uniqueness audit gate (`READY` / `WARN` / `BLOCK`)
- Migration Plan view
- Media settings: selectable K2 sizes + intro/full mapping
- Controlled rollback UI for staging / audit / `images/k2migration`
- Job + ledger + audit + staging tables

### IMPLEMENTED (Media hub — 0.2.1-alpha1)

- Media hub: folder settings, path/size stats, app vs manual modes
- K2 sizes-to-copy only (intro/full mapping deferred to articles)
- Resumable/incremental site images copy into `images/`
- Resumable/incremental K2 size copy into `images/k2migration/YYYY/MM/`
- Ledger-based site images rollback + k2migration tree rollback

### IMPLEMENTED (Preparation + Convert + Check)

- Prep create/remap for users, categories, tags, items, menus; undo journal
- Remap ID allocation floor = max(dest, staging, idmap) + 1 + 1_000_000 margin
- CLI exit 4 = busy; NDJSON progress; `devartk2migration:verify`; alias_changed journal
- SqlGuard keyword scans strip string/identifier literals first
- Convert hub readiness defers full preflight; Convert start / dry-run / CLI still gate
- Convert tags / items / menus (HTTP batches + full CLI via console plugin)
- Native article side-effects: workflow, UCM/tags, frontpage, intro/fulltext, dates
- Convert rollback (ledgered menus → articles → tags)
- Check: assets, counts, media, menus, ACL; editorial sync report/apply
- Job leases, `completed_with_errors`, batch/store metrics telemetry
- Media site-images manifest resume via sorted lexicographic path cursor

### CLOSED (by design)

- Write-path dry-run probes — Nested/Table asset paths can `LOCK TABLES` and
  implicitly commit; dry-run stays planner aggregates + preflight (no destination writes)
- Legacy `SqlFileImporter` removed — SQL file import is Database hub /
  `K2DatabaseImporter` only

### DEFERRED / NOT YET

- Modules / templates migration
- Bulk asset write-path optimisation (measured only; insert+`Content::store` remains)
- Article intro/full image mapping UI polish
- Automated collision resolution beyond Prep remaps
- Optional later RAXO → DevArt News converter (separate product)

## Source Connection

Configuration is stored in component parameters.

Required:

- Database host / port / name / username / password / table prefix

Recommended later for media:

- Absolute filesystem path to the legacy Joomla root on the same server

Security rules:

- Prefer a MySQL user with SELECT-only grants, e.g.
  `GRANT SELECT ON legacy_db.* TO 'k2migration'@'%';`
- Application `SqlGuard` blocks write/DDL and SELECT side-channels
  (`INTO OUTFILE` / `LOAD_FILE` / `SLEEP` / `BENCHMARK`) on the live source
- Source DB password sealed at rest (`SourceSecretCipher`, site secret; fail-closed if secret missing)
- Configured filesystem paths constrained by `PathAllowlist` (defaults + optional `path_extra_roots`)
- Connection test may warn (soft) when `SHOW GRANTS` looks writable
- Never log passwords
- Never expose passwords in frontend responses
- Parameterized queries / query builder (`createQuery`) only
- No unrestricted SQL UI
- CSRF + ACL on privileged actions

## Media Architecture (IMPLEMENTED — 0.2.1-alpha1)

See `docs/media-and-batch-architecture.md`.

Two resumable in-component jobs run through small administrator HTTP batches
with checkpoints. No SSH/CLI/Plesk dependency. App or manual modes available.

### Job A — Site images tree

- Source: `{sourceRoot}/images/` or override
- Destination: `{j6}/images/`
- Incremental skip by size + mtime; ledger-based rollback

### Job B — K2 selected sizes

- Source resolved by item ID from staging / live DB
- Destination: `images/k2migration/YYYY/MM/{md5}_{SIZE}.jpg`
- Folder date from `k2_items.created`
- Intro/full mapping deferred to articles step

## Batch Engine Architecture (PARTIAL)

Jobs/ledger tables exist. Media and SQL staging use HTTP batches with
checkpoints. Broader native-import batch workers remain planned.

## Anti-patterns from legacy SQL migrations

Do not repeat the old manual SQL approach as the primary strategy:

- Do not insert articles with `asset_id = 0` and rely on post-hoc ACL repair
- Do not insert tags with invalid nested-set values
- Do not map tags before UCM rows exist
- Do not leave incremental imports without images / tag relations / verification

## Source of Truth

- Develop only in `source/`
- Treat generated archives as disposable output
- Keep all extension manifest versions identical to the package version
- Keep every manifest `creationDate` in `YYYY-MM-DD` format
- Keep every manifest `php_minimum` identical to the DevArt PHP minimum (`8.3.0`)

## Build and Validation

- `php scripts/validate.php`
- `php scripts/build.php`
- `php scripts/test-phase1.php` — helper unit checks for Phase 1
