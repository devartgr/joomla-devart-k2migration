# Media and Batch Architecture

Status: **IMPLEMENTED** in Media hub (`0.2.1-alpha1`)

## Media hub (submenu Media)

- Image folder settings (optional source images override)
- Source / destination path info and bounded size stats (with newest file date)
- App vs manual copy modes for site images and for K2 media
- K2 sizes to copy only (intro/full mapping deferred to articles step)
- Full + incremental copy buttons, continue batch, rollback

## Job A — Site images tree

| Item | Value |
|------|-------|
| Status | IMPLEMENTED |
| Source | `{sourceRoot}/images/` or override path |
| Destination | `{j6Root}/images/` |
| Extensions | Allowlist: images, web assets, PDF/Office, ZIP/RAR/7z, common audio/video — not php/exe/scripts |
| Mode | App copy or manual (outside component) |
| Resume | Two-phase: (1) manifest build with lexicographic `manifest_cursor` + sorted per-directory walk (prune dirs before cursor — no full-tree restart); (2) copy via keyset `rel_path > cursor` on `#__devartk2migration_media_manifest` |
| Incremental | **Fast** (default): skip when size + mtime match (±2s). **Strict**: SHA-256 both sides. Setting `media_verify_mode`. |
| Owned refresh | If dest matches a prior published ledger `pub_sha256` and source changed → backup + replace as `source_updated` (not a foreign conflict) |
| Ledger scale | Checkpoint `ledger_floor_id` bounds unpublished/path scans to recent rows (avoids O(N²) payload LIKE over the whole run) |
| Rollback | Deletes only ledgered `entity_type=images` files |

## Job B — K2 selected sizes

| Item | Value |
|------|-------|
| Status | IMPLEMENTED |
| Selectable sizes | XL, L, M, S, XS, Generic |
| Source pattern | `{sourceRoot}/media/k2/items/cache/{md5('Image' . itemId)}_{SIZE}.jpg` |
| Destination | `images/k2migration/YYYY/MM/{md5}_{SIZE}.jpg` |
| Folder date | From `k2_items.created` (staging or live DB) |
| Filename | Keep original K2 MD5 basename |
| Mapping | Deferred to articles UI; choices limited to copied sizes |
| Discovery | By item ID from staging / live DB — not full cache scan |
| Incremental | Same verify modes as Job A; starts `cursor_id` after last completed* job for the same dataset (new items). Full (non-incremental) rescan from 0. |
| Resume size | Unpublished ledger resume recovers size from dest basename (`_M.jpg` → M), not hard-coded XL |
| Rollback | Deletes `images/k2migration` tree |

Helper class:

`DevArt\Component\Devartk2migration\Administrator\Helper\K2ImageHelper`

## Batch execution model

Status: **IMPLEMENTED** (HTTP batches via Media page)

- Small administrator HTTP batches
- Persistent checkpoints in `#__devartk2migration_jobs`
- Pause / Resume via Continue button
- Idempotent incremental skips
- Ledger entries for copied files
- Initial huge site-images trees: prefer **CLI** with `php -d memory_limit=-1 cli/joomla.php devartk2migration:media …` (HTTP may pause with `requires_cli` on listing memory / wall-clock stalls)

## Tables in use

- `#__devartk2migration_jobs`
- `#__devartk2migration_ledger`
