# Convert Engine Design

DevArt K2 Migration — native `#__tags`, `#__content`, and (post-MVP ordering) `#__menu`
writes from mirror staging. Staging primary keys remain **original source IDs**; Convert
resolves destination IDs through `#__devartk2migration_idmap` and `IdMapRepository::destIdOrSource()`.

## Identity and idmap

- **Preserve** (default): destination row id equals staging source id when no active remap exists.
- **Remap**: Prep stores `source_id → dest_id` pairs in idmap (`rule=remap`, `active=1`).
- Convert never silently rewrites colliding IDs; Preparation must clear conflicts first.
- Ledger/journal keys use `(entity_type, source_id)` so resume stays idempotent.

## CLI

- Exit codes: `0` OK, `1` hard failure, `2` completed_with_errors / stop-on-fail,
  **`3` interrupted** (SIGINT/SIGTERM — **not** shell `130`), `4` busy (lease).
- Real CLI via package plugin `plg_console_devartk2migration` (enabled on install).
  Commands (same services as admin UI):
  - `devartk2migration:import [--incremental] [--batch=N] [--stop-on-fail] [--json]`
  - `devartk2migration:media --target=images|k2 [--incremental] [--dry-run] [--batch=N] [--stop-on-fail] [--json]`
  - `devartk2migration:preflight [--json]`
  - `devartk2migration:convert --phase=tags|items|menus [--dry-run] [--batch=N] [--time-limit=ms] [--stop-on-fail] [--json]`
  - `devartk2migration:rollback [--target=convert|staging|audit|media|images] [--stop-on-fail] [--json]`
- Command classes stay in the component; the console plugin only registers them.
  Component `boot()` must not register CLI commands — `ConsoleApplication` does not boot components.

## HTTP batch executor

- Job types: `convert_tags`, `convert_items`, `convert_menus` in `#__devartk2migration_jobs`.
- Each admin HTTP request acquires a per-type `JobLeaseService` lock for that batch only
  (released in `finally`), updates checkpoint (`cursor_id`, `rebuild_tags`, etc.) and stats,
  then returns redirect + message.
- Terminal status: `completed`, or `completed_with_errors` when `stats.failed > 0`.
- Resume: fetch staging rows with `id > cursor_id` ordered by `id ASC`.
- Planner actions: `OP_CREATE` for missing dest rows; `OP_SKIP` when journal
  already succeeded or destination fully matches; `OP_FINALIZE` when a matching
  half-written native needs side-effect / asset repair (never a blind title/alias
  skip — see `ConvertPlanner`). `OP_CONFLICT` when dest exists but mismatches.
- Dry-run / summarize uses SQL aggregates (journal + dest existence + idmap) plus
  item preflight — **no destination writes**. Write-path probing is permanently
  closed: Nested/Table asset paths may `LOCK TABLES` (implicit commit).
- **C3 telemetry**: job `stats` accumulate batch / store / relations / rebuild
  milliseconds for performance decisions — see `docs/convert-asset-benchmarks.md`.

## Per-article durability and operation journal

- **Full items path (alpha125+)**: bulk multi-row `#__assets` + `#__content`
  (`ConvertBulkItemWriter`), then densify. Samples / finalize / conflict repair
  still use per-row `Content::store` (may `LOCK TABLES` — implicit commit).
- **Items (F1)**: write-ahead journal `pending` before durable native writes; after
  successful create/finalize record `create`. Incomplete rows plan as `OP_FINALIZE`
  (side-effects only — no second insert). Post-store failures open `needs_repair`.
- **Journal**: `#__devartk2migration_ledger` with `entity_type` `convert_tag` /
  `convert_item` / `convert_menu`, `entity_id` = source id, `action` =
  `pending` | `create` | `skip`, `run_id` = `convert-{jobId}` for job-scoped rollback.

## Phase ordering

1. **Tags** — must exist before item tag maps.
2. **Items** — K2 → `com_content` articles.
3. **Menus** — **not** in tags/items MVP UI scope; separate `convertMenus` after items exist.

Menus rewrite K2 `link` query ids through idmap (item, category, tag). Htaccess helper stays manual.

## Nested set strategy

| Entity   | Recommendation |
|----------|----------------|
| Tags     | **Rebuild-at-end** (or after each tag batch on large sites): flat insert under ROOT (`parent_id=1`, placeholder `lft`/`rgt`), then `TagTable::rebuild()`. Assumes K2 tags are flat tags under Joomla ROOT — not a deep K2 tree. |
| Articles | **Bulk OP_CREATE** (alpha125+): multi-row `#__assets` (placeholder lft/rgt) + `#__content`, link `asset_id`, batch workflow/frontpage/UCM/tags, then `AssetNestedRebuild` densify at job end. Samples / `OP_FINALIZE` / conflicts keep per-row `Content::store`. |

### Bulk destination assets (documented exception)

Product rule: never copy legacy `#__assets`. Destination assets must still be created
for Joomla ACL. The bulk path inserts **new** destination `#__assets` rows
(`name = com_content.article.{id}`, `rules = '{}'`, placeholder `lft`/`rgt`) then densifies
nested-set — it does **not** import source asset ids/rules. Compensations:
`ConvertOwnershipFingerprint`, `AssetNestedRebuild` verify / `assertNestedSetIntact`, and
Check. Keep Site Offline until densify completes; resume the same paused densify
job through browser batches (CLI remains optional)
(see [`ops-guide.md`](ops-guide.md)).
| Menus    | Staging nested set copied with remapped ids; **rebuild** after batch or job completion (`Menu::rebuild()`). |

High tag volume (&gt; ~2000): prefer single rebuild when job status becomes `completed`, not per row.

## Field mapping (MVP)

- **Categories**: `catid` via idmap `category` (categories must already exist from Prep).
- **Body**: K2 `introtext` / `fulltext` stored as separate columns (no
  `<hr id="system-readmore">` stuffed into intro). `{readmore}` markers stripped.
- **Tags on items**: staging xref → remapped destination tag ids →
  `TagsHelper::preStoreProcess` + `postStore` (creates `#__ucm_content` /
  `#__ucm_base` and correct `core_content_id` in `#__contentitem_tag_map`).
- **Featured**: K2 `featured` → `#__content.featured` **and**
  `#__content_frontpage` row when featured=1 (removed when 0).
- **Dates**: K2 `created`, `modified`, `publish_up`, `publish_down` preserved when
  valid. After `Content::store()` (which overwrites `modified`/`modified_by` on
  existing ids), `NativeArticleRelations` restores source modified values.
- **Authors / access**: `PrepSettings` (`preserve` vs `force`) applied to staging row before map.
- **Images**: `ConvertImageSettings` — K2 size paths under `images/k2migration/YYYY/MM/` (basename uses **source** item id) or fixed intro/full paths.
- **workflow_associations**: required side effect — default stage for
  `com_content.article` via `Workflow::getDefaultStageByCategory` +
  `createAssociation` (admin Articles list INNER JOINs this table).
- **Store failures**: per-row `Content::store()` returning false aborts that item
  (no journal success). Bulk create failures purge partial dest ids then may
  fall back to per-row create.
- **Plugin side effects**: bulk create + shared `NativeArticleRelations` (or
  per-row store + finalize). Full `ArticleModel::save` (plugins/history) is
  intentionally deferred.
- **Rollback**: before `#__content` delete, purge tag maps, frontpage, workflow
  associations, and UCM rows (`CoreContent::deleteByContentId`).

## Planner vs executor

- `ConvertPlanner` builds the same operation list for **dry-run** and **apply**.
- Dry-run: planner only; no durable commit, no ledger `create` rows.
- Apply: `ConvertExecutor` consumes planner output and commits.

## Explicit non-goals (Convert MVP)

- Copying legacy `#__assets` rows
- Modules, templates (deferred, no rush)
- Editorial sync apply is available on Check (batched overwrite + trash
  tombstones); Incremental Database import remains new-articles-only

## Out of scope permanently (will not be implemented — do not re-audit)

- K2 comments, attachments, galleries, video
- K2 extra fields/groups → `com_fields`
- Smart Search / Finder reindex after migration
- Native Joomla 3 `#__content` / `com_content` article migration (unrelated)

See `AGENTS.md` Project-Specific Rules and `PROJECT_STATUS.md` Out of scope.
