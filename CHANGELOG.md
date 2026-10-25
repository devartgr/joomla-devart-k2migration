# Changelog

## 1.0.0 — 2026-10-25

First stable public line for the live-corpus proven product: Database → Media →
Preparation → Convert → Check → Rollback on Joomla 6 / PHP 8.3+. Includes
incremental import/media, Prep category parent repair, alias unique-ify UX fixes,
and Convert densify at ~228k articles scale. Supersedes 0.3.1-rc101.

## 0.3.1-rc101 — 2026-10-25

Fix: Prep duplicate-alias sample table showed oldest as Keep (re-sorted by id ASC
after newest-first). Make unique button now appears after Verify completes even
when alias_review_required remains set (previously only Verify was visible).

## 0.3.1-rc100 — 2026-10-24

Fix: Prep Create missing categories no longer silently attaches children under
ROOT when a declared parent is missing on the destination (same refuse/skip
policy as menus). Adds Prep **Repair category parents from staging** for trees
already flattened by older Create runs, plus mismatch count on the Categories
step.

## 0.3.1-rc99 — 2026-10-24

Fix: items Dry-run always failed at `await_preflight` with
`CONVERT_BLOCK_GENERATION` after a valid Item Preflight. The dry-run compared
its admission `runGeneration` (includes source) to the preflight’s
`inputFingerprint` stored as `run_generation` — different hashes by design.
Freshness remains via dry-run pin drift + `peekCompletedResult`.

## 0.3.1-rc98 — 2026-10-24

Convert images + pilot Check for imageless / missing XL:
- K2 mode always checks the destination file; missing XL uses configured
  `convert_image_fallback` when that file exists, otherwise leaves intro/full
  blank (no phantom `images/k2migration/…_XL.jpg`).
- Sample Check no longer fails on missing image files (legacy imageless OK).
- Help text for the fallback field updated.

## 0.3.1-rc97 — 2026-10-24

Fix: Convert sample completed but Check sample stayed disabled.
- Soft-heal with empty CREATE ledger no longer runs while Convert tags/items
  are still running (it wiped `convert_pilot_sample_ids` before the first
  CREATE).
- Recover sample markers from the completed sample job on Convert page load
  and re-pin them when the sample job finishes — unlocks Step 3 without
  re-converting.

## 0.3.1-rc96 — 2026-10-24

Fix: Make unique was creating new alias conflicts while “fixing” old ones.
- Plan occupancy is global per destination category (all existing effective
  aliases + aliases already chosen earlier in the same plan), not per-group
  only — so renaming peers of `foo` cannot steal a singleton `foo-2` and the
  next run invent `foo-2-2` / reopen dups.
- Convert Item Preflight FAILs on alias conflicts (`max(effective, raw
  same-cat)`); PASS cannot coexist with hub staging dup aliases.

## 0.3.1-rc95 — 2026-10-24

Break Make unique / Preflight leftover loop:
- Do not clear hub alias-quality snapshot when starting Make unique (that
  reopened thousands of raw dups mid-run).
- Numeric suffix allocation strips trailing -N stems (…-2 → …-3, never …-2-2).
- Items Solutions warns to prefer manual rename of the Preflight sample over
  re-running Make unique for thousands of groups.

## 0.3.1-rc94 — 2026-10-24

Fix: Make unique must not leave effective alias duplicates across remapped
destination categories. Occupancy checks now use dest-cat scope (same as
Convert Preflight), not only staging source catid. Prep Items also keeps a
Make unique / manual rename path when Preflight still reports a leftover
while hub raw dups are already 0.

## 0.3.1-rc93 — 2026-10-24

Fix: Items Prep Verify no longer infinite-loops after completion.
- Auto-start only when no completed `prep_item_eff_alias_scan` exists yet.
- Live overlay of alias-quality flags so Make unique appears without Recheck
  while the hub snapshot is still stale.

## 0.3.1-rc92 — 2026-10-24

Convert + Media UX from primary live corpus (~230k):
- Prep Step 7: optional K2-mode image fallback (`convert_image_fallback`) —
  when the chosen size file is missing under `images/k2migration`, Convert
  (and editorial sync) writes the configured `images/…` path instead of a
  broken K2 path. Fixed mode unchanged; empty fallback keeps prior behaviour.
- Media job summary line now includes `missing` (imageless K2 / absent XL),
  distinct from `skipped` (already present / incremental).

## 0.3.1-rc91 — 2026-10-23

Media scale + daily sync hardening before large-corpus rehearsal:
- Ledger `ledger_floor_id` bounds unpublished/path scans (avoids O(N²) payload LIKE).
- K2 unpublished resume recovers size from dest basename (not hard-coded XL).
- `media_verify_mode` fast (size+mtime, default) / strict (SHA-256).
- Owned dest + changed source → backup refresh as `source_updated` (not foreign conflict).
- K2 incremental starts after last completed cursor for the same dataset.
- Listing memory refusal → paused + `requires_cli` (not failed).
- Incremental import surfaces `skipped_missing_tag_xref` / `orphan_catid_items` counts.
- K2 zero/invalid `created` → stable 1970/01 folders (never year -0001).

## 0.3.1-rc90 — 2026-10-19

Fix: asset densify uses Joomla-canonical nested set (ROOT `lft=0`,
`max(rgt)=2N-1`) so PWT ACL / DevArt Utils no longer report a misleading
root-lft error that tempts a destructive full rebuild on large sites.
Sites already densified with 1-based numbering need one densify re-run
after update (or Utils safe root-lft fix).

## 0.3.1-rc89 — 2026-10-19

Media: site `images/` copy allowlist now includes common editorial attachments
(PDF, ZIP/RAR/7z, Office doc/xls/ppt + OpenDocument, MP3 and related audio/video)
while still excluding server executables (php, phar, exe, …).

## 0.3.1-rc88 — 2026-10-19

Fix: Check browser scan no longer stalls behind Cloudflare 524 / 22s AJAX
retries on large destinations. Article/pending phases use small keyset pages,
batched ownership + workflow lookups (no correlated EXISTS), and the same
PerformanceProfile time budget as Convert / Import / Media.

## 0.3.1-rc87 — 2026-10-19

Fix: foreign alias collision button actually repairs state after Make unique —
sync destination from staging when older converted articles were renamed only
in staging; resolve owners live (empty Dest columns); invalidate stale Preflight.
Make unique now syncs same-ID destination aliases too.

## 0.3.1-rc86 — 2026-10-18

Policy: alias conflicts keep the newest article (by created) untouched; older
peers get automatic -2 / -3 suffixes. Foreign dest collisions: one-click suffix
on the older owner (staging + matching destination) — no typing a full alias.
Make unique uses the same newest-keep rule.

## 0.3.1-rc85 — 2026-10-18

UX: Preflight blockers show staging/destination titles and aliases (not bare
sample ids). Prep warning is explanation-first — no jump buttons that skip
understanding; Solutions remain the fix path.

## 0.3.1-rc84 — 2026-10-18

UX/fix: Prep no longer looks Convert-ready while Item Preflight blocks Convert.
Last Preflight blockers (sample ids + fix CTAs) appear on Preparation → Items and
override the hub Items card. Rename staging item alias for dest_alias_collision.
Alias-quality restore after Incremental requires exact staging size (no 95% revive).

## 0.3.1-rc83 — 2026-10-18

Fix: Incremental import no longer invalidates Item Preflight / locks Convert
items while tags stay unlocked — preflight validity ignores dump/source
fingerprint (matches product note). Re-preflight after full Convert no longer
flags dest_alias_collision / duplicate_alias only among already-converted items.

## 0.3.1-rc82 — 2026-10-18

UX: Convert Tags / Menus / K2 items rows show last successful conversion
(date/time + created/skipped/failed + job id). AJAX completion shows an
on-page success flash (HTTP finish had no Joomla message queue).

## 0.3.1-rc81 — 2026-10-18

Fix: Convert “Latest convert jobs” hub line updates live from AJAX (created/
status/expected) so install mid-run without full reload no longer leaves the
list stuck at created 0 while the progress bar advances.

## 0.3.1-rc80 — 2026-10-18

Fix: Convert items progress bar jumped to densify:prepare @ 99% after the first
bulk creates (~thousands of 227k) because rebuild_assets intent was treated as
active densify. Progress now uses created+skipped / expected until rebuild_state
exists; densify band is 90–99.9% (not 99+). Detail shows progress N/expected.

## 0.3.1-rc79 — 2026-10-18

Fix: Convert hub job display — show job id + sample-of-N (so sample 100 with
created 50/skipped 50 is not mistaken for the old 50 run); supersede prior
finished jobs on new start; soft heal invalidates ALL completed write jobs
(not only latest) so older samples cannot resurface.

## 0.3.1-rc78 — 2026-10-18

Fix: global Convert Rollback still called full pilot wipe (invalidate after
Clear) so soft heal only saved Preflight Step 1 — Steps 2–4 / Unlock were
lost. Rollback now soft-heals only and keeps pilot PASS + Unlock; incomplete
sample markers without PASS are cleared so Check is not offered on deleted rows.

## 0.3.1-rc77 — 2026-10-18

Fix: after Convert Rollback, empty-ledger heal no longer resets pilot, cancels
preflight, or forces hub “Checking staging…” — soft heal only hides finished
write jobs and re-pins preflight dest. Explicit Clear still full-resets.

## 0.3.1-rc76 — 2026-10-18

Fix: Convert tags failed with Duplicate entry com_tags.tag.* on idx_asset_name
after sample rollback left orphan assets (no #__tags row). Purge orphan tag
assets before bulk/singleton create and after tag Clear/delete.

## 0.3.1-rc75 — 2026-10-18

Fix: Prep Step 7 “one intro + one full” image mode failed to save and fell
back to K2 sizes — media paths were filtered/empty on POST. Read media RAW,
keep last saved fixed paths when re-selecting fixed, normalise joomlaImage URLs.

## 0.3.1-rc74 — 2026-10-17

Fix: Convert sample start was cancelled by empty-ledger heal (running job
before first CREATE) which also wiped preflight + pilot sample ids. Heal no
longer touches running Convert. Own Convert writes re-pin preflight destination
admission so Safety test Step 1 stays PASS after sample.

## 0.3.1-rc73 — 2026-10-17

Fix: “Why rows were skipped / matching destination” after Clear used leftover
Convert SKIP ledger rows and staging titles — looked like live #__tags even
when the destination was empty. Clear/heal purge skip receipts; hub paints
live skip breakdown only from non-cancelled Convert jobs.

## 0.3.1-rc72 — 2026-10-17

Fix: empty Convert ledger heal no longer treats a fresh item preflight as
“stale” and cancel it on every page load. Live preflight PASS overlays the hub
snapshot so Convert sample unlocks after Run item preflight.

## 0.3.1-rc71 — 2026-10-17

Convert checklist Safety-test row and pilot banner always use the live pilot
gate (not a frozen hub snapshot). Heal also invalidates a completed hub
snapshot that still embeds a prior unlock when the Convert ledger is empty.

## 0.3.1-rc70 — 2026-10-17

Convert Clear (global) resets Convert hub UI: invalidate prior preflight /
dry-run / convert jobs, clear pilot PASS/promote, and drop stale convert hub
snapshot. Opening Convert with an empty ledger auto-heals sites cleared before
this fix so old test runs no longer look “done”.

## 0.3.1-rc69 — 2026-10-17

Heal: username/email fixes applied before rc68 (Undo receipt present, no
override rows) are copied into durable overrides on Prep hub load so they
appear in the inventory and survive Full Import.

## 0.3.1-rc68 — 2026-10-17

Prep hub shows durable field corrections (item/category aliases + user
username/email) beside idmap remaps. User username/email fixes are now durable
and re-applied after Full Import. Step 5: large-site hub no longer uses list
limit 0; live ID-conflict overlay after Remap; Recheck restarts hub diagnosis;
trusted-clean alias count matches Dry-run; Undo alias styling when disabled.

## 0.3.1-rc67 — 2026-10-17

Convert clear was still far slower than Full import because ownership verified
article bodies (SQL SHA2 still reads LOBs). Clear now uses editorial fingerprint
only. CLI default chunk grows to 1000 rows when time-limit ≥ 15s; add `--batch`
and `--force-edited`.

## 0.3.1-rc66 — 2026-10-17

Soft P2 (safe subset): Check Convert ownership is idmap-aware; large sites get a
bounded orphan article-asset sample. Densify + eff-alias upserts use
SqlUpsertDialect; Category paths drop deprecated getError(). Make unique marks
trusted clean only with zero failures; Import/remap/undo remap clear the alias
quality snapshot. CLI hints are framed with Copy code + site-root instruction.
Skipped high-risk densify work-table ownership schema.

## 0.3.1-rc65 — 2026-10-17

Admin UI shows copy-paste CLI commands on Database, Media, Convert, Check, and
Rollback (from Joomla site root). CLI `--time-limit` accepts `30s` / `5000ms`
as well as bare milliseconds.

## 0.3.1-rc64 — 2026-10-17

Convert clear was far slower than Full import because ownership checks pulled
every article introtext/fulltext into PHP each tick. Body hash now runs in SQL
(SHA2). Added Cancel job on Rollback, fixed stale “deleted N” banner, and CLI
`--time-limit` default 30s for large clears.

## 0.3.1-rc63 — 2026-10-17

Fix Ops backup acknowledgement: after confirm, UI could still show “missing”
because status() read stale ComponentHelper `_system` cache. Status now loads
Extension params from the database; record() cleans `_system` cache. Rollback
page also has the ack form (stays on Rollback after submit).

## 0.3.1-rc62 — 2026-10-17

Rollback: Convert CTA renamed to Clear articles / tags / menus (with ledger
count note). New Clear Prep users / categories job deletes staging/idmap-matched
destination users and com_content categories (current login protected). Prep
Create now journals ownership for future clear.

## 0.3.1-rc61 — 2026-10-17

Convert admission pins identity on the first deferred tick (no silent re-pin).
Incremental import no longer globally purges orphan tag xref — only new articles
are filtered at ingest. Two consecutive memory pauses on the same SQL tuple
without progress become a permanent memory skip. Prep Users help clarifies
Destination only vs disabled users.

## 0.3.1-rc60 — 2026-10-17

Prep staging alias corrections (categories + Make unique) and orphan tag-xref
policy now survive Import (full). Same user ID with the same email counts as a
Match even when the destination username differs. Clear temporary tables still
wipes staging, idmap, and these durable Prep corrections.

## 0.3.1-rc59 — 2026-10-17

Database Clear / Import (full) / Incremental buttons show short on-page captions
stating exactly what each action does (staging, Prep ID rules, articles-only).

## 0.3.1-rc58 — 2026-10-17

Clear temporary tables now wipes staging and Prep ID rules (true start over).
Import (full) reloads staging data but keeps idmap remaps. Pilot/Preflight
fingerprints use structural staging (categories + tags) so Incremental article
imports do not force a new Sample when Preparation stays clean.

## 0.3.1-rc57 — 2026-10-17

After a full Database import, Incremental import brings only new K2 articles and
their tag links (`k2_items` + `k2_tags_xref`). It no longer refreshes users,
categories, menus, or other structural staging. The Database page states this
cutover contract: one large initial load, then article-only follow-ups until go-live.

## 0.3.1-rc56 — 2026-10-17

SQL ingest no longer permanently drops small rows when the current PHP request
is near memory_limit. Those rows pause and retry on the next batch unless even
a fresh request cannot afford them. The Database skipped list now shows table
and reason (oversize / memory / truncated).

## 0.3.1-rc55 — 2026-10-16

Preparation and Convert open immediately. Staging and destination diagnosis
runs in bounded browser batches and is kept for the next visit. Refresh
diagnosis when the data has changed.

## 0.3.1-rc54 — 2026-10-16

Check media existence now strips Joomla `#joomlaImage://` fragments so existing
files are not reported as missing. The Dashboard shows recommended PHP / FPM /
database settings for large migrations.

## 0.3.1-rc53 — 2026-10-16

The resumable Check shell now renders its status table immediately while report
batches run, preserving the administrator table contract without blocking page
load.

## 0.3.1-rc52 — 2026-10-16

Check now paints immediately and runs counts, media, menus, exact article
asset/ACL/workflow validation, pending-ledger validation, and editorial sync
through durable browser batches. Large reports resume from keyset checkpoints
instead of blocking page load. Convert skip samples now include titles and the
originating successful run, and the UI/docs publish recommended large-migration
PHP/FPM/database settings.

## 0.3.1-rc51 — 2026-10-16

Asset densify now completes through durable browser batches for Convert and
Rollback (`snapshot → repair → number → flush → verify`), including same-job
resume from legacy paused checkpoints. Convert progress reports the active
densify phase, and skipped items are split into previous-Convert versus
destination-match counts with sample IDs. CLI remains optional acceleration.

## 0.3.1-rc50 — 2026-10-16

Full Convert admission is split into resumable AJAX ticks: pilot gate,
preflight, asset capacity, and identity pinning complete separately before
writes begin.

## 0.3.1-rc49 — 2026-10-16

Full Convert tags and K2 items now return before processing: the progress UI
paints first and resumable AJAX ticks perform admission checks and writes.
CLI Convert remains synchronous.

## 0.3.1-rc48 — 2026-10-16

Pilot Unlock now reads persisted state directly instead of stale component
parameter cache; successful Unlock enables the full Convert actions immediately
and moves the operator to the next action.

## 0.3.1-rc47 — 2026-10-16

Pilot Unlock: paused historical Convert jobs no longer suppress the current
pilot fingerprint; a passing sample check can enable Unlock full Convert.

## 0.3.1-rc46 — 2026-10-16

Convert large-site start latency: request-local readiness/fingerprint memoization,
deferred Preflight/Convert identity pinning after the progress UI paints, and
`staging_imported` index for K2 tag xref fingerprint aggregates.

## 0.3.1-rc45 — 2026-10-16

Convert sample: replace the category N+1 selector with one grouped query;
HTTP start now creates deferred tag/item jobs and redirects immediately to
AJAX progress. Related tags run before articles; Cancel stops Convert jobs.

## 0.3.1-rc44 — 2026-10-16

Prep hub vs Items: heal alias quality snapshot from completed Make unique /
Verify jobs (fixes Needs review after cancelled auto-Verify cleared cache);
Verify start no longer wipes the snapshot.

## 0.3.1-rc43 — 2026-10-16

Prep Items: after Make unique completes, do not auto-start another Verify;
trusted_clean snapshot survives Recheck when raw_dup drifts after renames.

## 0.3.1-rc42 — 2026-10-16

Prep Items: Verify effective aliases and Make unique scan phase both use
chunked AJAX progress (spill table `#__devartk2migration_prep_eff_alias_scan`)
instead of one long page request — progress bar visible, Cloudflare-safe.

## 0.3.1-rc41 — 2026-10-16

Prep Items large-site: page load no longer runs the full effective-alias
scan (~90s); AJAX Verify starts automatically and caches the result.
Make unique Apply keeps AJAX progress bar; Dry-run uses the cached rename
count when available.

## 0.3.1-rc40 — 2026-10-16

Prep hub: do not show Items Convert-ready on large sites until an Items
drill-in verifies effective (stringURLSafe) aliases; cache that result for
the hub badge (Needs action when dups remain).

## 0.3.1-rc39 — 2026-10-16

Prep Items: Convert-equivalent effective alias scan on Items drill-in
(stringURLSafe + dest category); Make unique uses the same groups.
Prep Tags: Clean orphan tag xref (AJAX job + Undo) when staged
tags_xref points at missing k2_tags — hub Needs action until cleaned.

## 0.3.1-rc38 — 2026-10-16

Prep Items alias dedupe progress: show renamed/of (not deleted); clarify
groups vs rows renamed; hide Cancel when AJAX job completes.

## 0.3.1-rc37 — 2026-10-16

Prep Items: Make aliases unique Apply runs as AJAX job with progress bar
(plan+undo tick, then rename chunks) — no single long HTTP request / Cloudflare 524.

## 0.3.1-rc36 — 2026-10-16

Prep Items large-site: skip duplicate-alias sample SQL on page load (Cloudflare
524); load keep/peer samples via AJAX button; lighter peer lookup (no
GROUP_CONCAT); composite index idx_cat_alias.

## 0.3.1-rc35 — 2026-10-16

Prep Items: duplicate-alias accordion always shows sample rows (even when
capped) with category name, keep/peer ids, titles and created dates — so
operators can inspect whether clashes are old republishes. Convert Preflight
fail list shows sample ids.

## 0.3.1-rc34 — 2026-10-16

Prep Items: Make staging aliases unique (keep lowest id, rename others to
alias-{id}) with Dry-run + Undo — resolves Convert duplicate-alias blockers.

## 0.3.1-rc33 — 2026-10-16

Prep: large-site / empty-dest paths no longer skip Convert blockers — Items
surface staging duplicate aliases, empty titles, orphan categories; Tags
surface empty names and missing tag xref. Hub cards show Needs action instead
of false Convert-ready.

## 0.3.1-rc32 — 2026-10-16

Convert UX: Preflight progress runs inside Step 1 (auto-scroll); failed
Preflight lists blockers with “what to do” in the safety panel; checklist
shows cached Preflight result; remove duplicate Preflight/sample buttons
from Convert actions.

## 0.3.1-rc31 — 2026-10-16

Convert safety test: Item Preflight is Step 1; Convert sample is Step 2 and
stays disabled until Preflight PASS is current (no contradictory “sample
first” then Preflight wall).

## 0.3.1-rc30 — 2026-10-16

Convert: rewrite Pilot gate UX as clear 3-step safety test (sample → check →
unlock); human-readable checklist values instead of none+fp_stale.

## 0.3.1-rc29 — 2026-10-16

Prep hub: move Active Prep ID rules (idmap) below the step cards.

## 0.3.1-rc28 — 2026-10-16

Prep Menus hub: optional / never blocks Convert; note that old public URLs need
correct aliases — prefer rebuilding menus yourself with legacy aliases.

## 0.3.1-rc27 — 2026-10-16

Prep Menus: Remap all clears conflicts (idmap exclusion in diagnosis); Undo remap
button; staging Home excluded from Create/Convert (destination home never cleared).

## 0.3.1-rc26 — 2026-10-16

Prep hub Media step: treat Media files "manual" mode as satisfied (no longer
Needs action when no app copy job was recorded).

## 0.3.1-rc25 — 2026-10-16

Prep Convert images: accept Joomla media-field values with
`#joomlaImage://…` suffix when saving fixed intro/full paths.

## 0.3.1-rc24 — 2026-10-16

Prep Items: Cancel job stops bulk delete/remap completely (Pause only stopped
the browser loop; Recheck was resuming the still-running DB job).

## 0.3.1-rc23 — 2026-10-16

Prep Categories: fix destination math help (foreign-extension PK conflicts are
not part of Destination total). Split content vs foreign conflict counts.
Delete dropdown excludes foreign PK rows (Remap only).

## 0.3.1-rc22 — 2026-10-16

Prep Categories: bulk remap all ID conflicts (title mismatch or foreign-extension
PKs) to free destination IDs in one action, with dry-run and Undo — same pattern
as tags/menus. Single remap remains available.

## 0.3.1-rc21 — 2026-10-16

Prep Categories: IDs owned by other extensions in #__categories (com_contact,
com_users, …) are ID conflicts requiring Remap — not “Missing” that Create
cannot insert. Free-ID allocation uses max PK across all extensions.

## 0.3.1-rc20 — 2026-10-16

Prep Categories: after Create, same ID + same title counts as Match (not ID
conflict) when Joomla stringURLSafe transliterates Greek aliases. Create syncs
staging alias to the stored destination alias.

## 0.3.1-rc19 — 2026-10-16

Prep Categories: hide Delete destination when ID conflicts and alias clashes
are already 0 (destination-only leftovers after remap are optional). Undo
alias / delete / remap stay visible after conflicts clear.

## 0.3.1-rc18 — 2026-10-16

Prep Categories: change staging alias to resolve alias clashes (preferred;
keeps destination). Clarify Destination only vs Destination total math.

## 0.3.1-rc17 — 2026-10-16

Prep Categories Solutions: Delete dropdown includes alias-clash and
destination-only category IDs (not only ID conflicts). Clear help that
Remap does not free aliases.

## 0.3.1-rc16 — 2026-10-16

Prep Categories: alias clash diagnosis includes staging vs destination
(e.g. staging `statikes` vs destination ΣΤΑΤΙΚΕΣ). Create is blocked
until those clashes are resolved.

## 0.3.1-rc15 — 2026-10-16

Prep Categories diagnosis: after an idmap remap (e.g. 8→1000129), the old
destination category ID is listed under **Destination only** again (previously
hidden because staging still had the same source PK).

## 0.3.1-rc14 — 2026-10-15

Prep Categories: refuse duplicate aliases under the same parent (diagnosis +
Create blocked; no silent rename). Trashed destination categories can be
hard-deleted via Prep Delete (including duplicate-alias trees where Joomla
Empty Trash fails). Clearer delete errors for articles/children.

## 0.3.1-rc13 — 2026-10-15

Prep hub: **Active Prep ID rules (idmap)** inventory (categories/tags/items/
menus/users) plus **Clear all remaps** for reset-from-scratch. Category Create:
no Nested outer TX, unique aliases, error detail + partial keep. Diagnosis:
remapped+created categories count as matches (not Missing / Destination only).

## 0.3.1-rc12 — 2026-10-15

Prep Categories: remap is idmap-only (staging IDs stay; Convert maps article
catid; rules survive incremental Database import). Free destination ID is
auto-allocated (no manual pick). Scoped **Undo delete** and **Undo remap**.
Help text matched the real behaviour.

## 0.3.1-rc11 — 2026-10-15

Prep Users: **Undo fixes** stays visible after Apply clears uniqueness blocks
(previously the whole Fix section disappeared with the button).

## 0.3.1-rc10 — 2026-10-15

Prep Users Fix Apply: specific conflict messages (staging ID + username/email +
destination user ID). Collects all conflicts in one Apply instead of a generic
message.

## 0.3.1-rc9 — 2026-10-15

Prep Users Fix staging: only clashing username and/or email fields are editable
(yellow). Non-clashing values are read-only. Clash column mirrors diagnosis.

## 0.3.1-rc8 — 2026-10-15

Prep Undo is step-scoped only (no cross-step generic Undo toolbar). Access and
Author use delta snapshots (`prep_access_undo` / `prep_author_undo`) so Undo
works on large sites instead of a failing full-table copy. Apply updates only
rows that differ. Buttons: Undo access / Undo author (gray until Apply).

## 0.3.1-rc7 — 2026-10-15

Prep Create missing users: optional common password (checkbox reveals field;
hashed via UserHelper; requireReset=1) and Enabled/Disabled status for all new
destination users. Unchecked password keeps staging password hashes.

## 0.3.1-rc6 — 2026-10-15

Prep Users Fix staging: **Undo fixes** (scoped, gray until Apply) instead of a
misleading generic Undo beside identity fixes. Apply rejects username/email
values already present on destination (and duplicate values within the batch).

## 0.3.1-rc5 — 2026-10-15

Prep Items Undo UX: generic Prep Undo hidden on Items; **Undo remap** stays gray
until Apply remap succeeds, then unlocks. Large Delete never unlocks Undo remap.
Bulk Remap Dry-run/Apply (≤10000) use the same job + progress bar pattern as
Delete. Remap stays gray when conflicts exceed the limit.

## 0.3.1-rc4 — 2026-10-15

Prep item Dry-run and Apply both use job ticks + progress bar. Dry-run walks
conflict IDs in short batches (no single long wait / no blocking COUNT at start).
Uses diagnosis conflict_total as progress hint.

## 0.3.1-rc3 — 2026-10-15

Prep item Dry-run delete uses `COUNT(*)` (instant) instead of loading all conflict
IDs into PHP. Progress bar remains on Apply job ticks only — Dry-run is count-only
by design for shared hosting.

## 0.3.1-rc2 — 2026-10-15

Real-clone Prep UX: gray/disable bulk Remap when conflicts exceed 10000 (items/
tags/menus) with a strong delete warning. Prep item bulk delete Apply now runs
as a timeout-safe job with shared Convert/Media progress bar (shared-hosting
friendly); Dry-run stays count-only and synchronous.

## 0.3.1-rc1 — 2026-10-15

First 0.3.1 release candidate for real-clone / large-data rehearsal. GPT α30 P2:
media ledger journals destination `orig_mtime` (pre-overwrite clock), not source
publish mtime; Undo restore prefers backup mtime then journaled `orig_mtime`.
0.3.0 A–D does not transfer — re-run A–D on this SHA before a stable cut.

## 0.3.1-alpha30 — 2026-10-15

Opus hygiene close: `scripts/record-evidence.php` writes reproducible evidence
envelopes with `installed_version` and anonymized paths; `verify-evidence` checks
installed vs package version and takes harness path from `--harness` /
`DKM_QA_HARNESS_REPORT` (no hardcoded home path). qa-gate gains a generic
version-consistency contract so historical pins stop chasing the current cut.
Media Undo restore prefers atomic `rename` (unlink only as fallback). Docs:
run-record placeholders; `current-state` points at PROJECT_STATUS for verified;
quick-start separates densify seconds from full Convert wall time.

## 0.3.1-alpha29 — 2026-10-15

Sonnet P2: `docs/expected-assertions.integration.json` pins ConvertIntegrationHarness
assertion prefixes; `verify-evidence` requires ≥26 PASS lines, zero FAIL, and each
prefix present in `evidence.integration.assertions`.

## 0.3.1-alpha28 — 2026-10-15

GPT α26 group 2: `ExecutorWriterCoupledFixtureTest` runs real ConvertExecutor +
bulk writers (items/tags × commit-ACK loss / oversize singleton) via fixtures
under `tests/fixtures/` — not a local catch-chain mirror.

## 0.3.1-alpha27 — 2026-10-15

GPT α26 audit P1: media Undo verified temp-file restore + durable
`restore_inflight` (interrupted retry ≠ editor skip); backup cleanup retry after
orig digest match; backup preserves original mtime; Prep Undo restores editorial
stamp before Tags/UCM. Sonnet L1: evidence under `qa/evidence/` (tracked);
`verify-evidence` rejects empty `[]` and missing explicit `--harness`. Prep
Create Undo accepted as narrower product promise (not deferred).

## 0.3.1-alpha26 — 2026-10-15

Kimi P1 polish: verify-evidence harness compare now fails loudly on empty
scenario ID parse; dashboard quick-start hint uses `Text::sprintf` with file
links; README lists rehearsal checklist + CLI; run-record example refreshed to
0.3.1-alpha25 SHA. qa-gate alpha041 contract pins updated to match.

## 0.3.1-alpha25 — 2026-10-15

Sonnet K2: complete operator quick-start (host sizing, measured Convert rates,
exit/status tree, incremental cutover, SEO/aliases) + dashboard quick-start
hint. Includes Sonnet K1 evidence tooling shipped alongside
(`verify-evidence`, rehearsal/release docs).

## 0.3.1-alpha24 — 2026-10-15

GPT α19 audit #5: real writer→Executor catch fixtures (`ExecutorWriterCatchFixtureTest`)
with a `return [`→assignment mutation probe; docs fix promote/`preflight` placement
and a separate unchecked 0.3.1 residual A–D checklist (not transferred from 0.3.0).

## 0.3.1-alpha23 — 2026-10-15

GPT α19 audit #4: ambiguous media destinations pin durable `recovery_required`
on the ledger (no endless resume); Undo verifies restore digest, restores backup
mtime, and requires backup cleanup before receipt consume; copy-run pin scans
100 terminal jobs.

## 0.3.1-alpha22 — 2026-10-15

GPT α19 audit #3: Prep Delete Undo restores frontpage `ordering` with schedule
dates, and re-applies `modified`/`modified_by` after `Content::store()` so the
native store overwrite does not erase the snapshot editorial stamp.

## 0.3.1-alpha21 — 2026-10-15

GPT α19 audit #2: media listing `paused`/`requires_cli` resumes the same job on
CLI — CAS paused→running under lease with a larger listing budget; browser
returns `MEDIA_REQUIRES_CLI` and must not spawn a second run.

## 0.3.1-alpha20 — 2026-10-15

GPT α19 audit #1: pending tombstone reconcile refuses trash when the source
item reappeared in staging (`source_reappeared`) or the effective dest mapping
changed (`mapping_changed`) — eligibility conflict, zero stores, receipt kept.

## 0.3.1-alpha19 — 2026-10-15

Sonnet J3: operator quick-start for clone destinations — `docs/quick-start.md`
documents the install → import → media → prep → `--sample`/`--check`/`--promote`
→ full Convert path with flags verified against `ConvertCommand` /
`ImportCommand` (default sample N=200); README + ops-guide link the guide.

## 0.3.1-alpha18 — 2026-10-15

GPT α12 audit #6 (+ Sonnet J1 snapshot): FunctionalRegressionPathsTest asserts
tags **and** items Executor catch chains with a mutation probe (first-only
strpos blind spot closed); `docs/expected-assertions.harness.json` records the
current harness scenario ID set for same-SHA acceptance evidence.

## 0.3.1-alpha17 — 2026-10-15

GPT α12 audit #5: FS phase wall-clock — directory listing prep honours the tick
deadline (soft defer → `requires_cli` on stall); stale manifest purge is
checkpointed across ticks (`images_phase=purge`); empty-dir cleanup and
`directoryHasFiles` fence every traversed entry.

## 0.3.1-alpha16 — 2026-10-15

GPT α12 audit #4: Prep Delete Undo native side-effects — item snapshots capture
and restore tags, workflow stage, frontpage (+ schedule), and article asset
ACL; category Delete Undo restores captured category asset rules. Prep Create
Undo remains honestly Not delivered (Create is immediate destination state).

## 0.3.1-alpha15 — 2026-10-15

GPT α12 audit #3: media publish digest verify — after copy to temp, refuse
rename unless temp (and post-rename dest) SHA-256 matches the journaled
`pub_sha256`. Source drift after intent yields `publish_incomplete` +
`source_digest_mismatch` without landing mismatched bytes.

## 0.3.1-alpha14 — 2026-10-15

GPT α12 audit #2: pending tombstone ownership on retry — intent stores a
durable ownership fingerprint; reconcile re-checks live `#__content` before
trash (and on already-trashed finalize). Local edits after intent →
`ownership_conflict`, receipt kept, zero stores.

## 0.3.1-alpha13 — 2026-10-15

GPT α12 audit #1: first HTTP pilot sample bootstrap — readiness checklist
still surfaces pilot status, but pilot blockers no longer fold into `ready`
(that blocked Sample behind a completed pilot). Full Convert remains gated by
`ConvertPilotGate::assertFullConvertAllowed()` and hub `$fullAllowed`.

## 0.3.1-alpha12 — 2026-10-15

Sonnet I4: lighter production package — omit `ConvertIntegrationHarness` and
`IntegrationTestCommand` from installable ZIPs; package postflight purges
orphans; console plugin still optionally registers when classes exist;
`scripts/run-integration-convert.php` overlays repo sources for QA then
removes them; acceptance/ops/current-state document lean package.

## 0.3.1-alpha11 — 2026-10-15

GPT 0.3.0 audit #8: targeted functional regressions + synced A–D records —
`tests/FunctionalRegressionPathsTest.php` exercises writer→caller recovery/
packet vs fallback, media ACK matrix, tombstone pending gate, paused CLI
admission, Prep Undo consume gates; `run-record.example.json` keyed to
0.3.0 stable; acceptance/current-state declare 0.3.1 cuts do not inherit
0.3.0 A–D.

## 0.3.1-alpha10 — 2026-10-15

GPT 0.3.0 audit #7: exact identity / completeness bounds — destination
admission digests bounded owners (COUNT + MAX id/modified + SUM/XOR
CRC32 of `id:alias:catid:state`) so same-aggregate alias/catid edits
invalidate cached preflight; SQL dump fingerprints add ctime/inode and
remain explicitly sampled; `identityContract()` + ops-guide table declare
certainty; Not delivered lists exact whole-file dump hash.

## 0.3.1-alpha9 — 2026-10-15

GPT 0.3.0 audit #6: bounded generic filesystem + recovery accounting — cold
listing admits memory before collecting names (`opendir` count pass); refused
filenames increment `unsupported_paths`; CHILD_FIRST empty-dir cleanup no
longer uses a lexicographic cursor; batched manifest stale purge; ledger/
manifest multi-row writes share packet byte caps.

## 0.3.1-alpha8 — 2026-10-15

Sonnet I3: Honest status — PROJECT_STATUS / current-state “Not delivered”
(session password, SSL CA/verify, spill-only clean); convert-engine-design and
benchmarks docs synced to bulk OP_CREATE + OP_FINALIZE; `scripts/check-docs.php`
for version/docs consistency. No Convert execution-path changes.

## 0.3.1-alpha7 — 2026-10-15

Sonnet I2: Pilot gate surfaces in Convert readiness (blockers + checklist row);
explicit `convert_require_pilot=0` bypass records `convert_pilot_bypass_at` /
count; ops-guide pilot section; acceptance A–D requires pilot path + integration
harness evidence on the same installed package. Integration harness enables
CLI `setSyncPreflight(true)` (same as ConvertCommand); `DKM_CRASH_INJECTED`
rethrows without marking the job failed so in-process crash-resume works.

## 0.3.1-alpha6 — 2026-10-15

Sonnet I1: Convert safety ops — Site Offline + densify resume CLI documented;
Convert admin warns when the site is online and when a densify job is paused
(requires_cli); bulk destination assets exception noted in convert-engine-design
(+ AGENTS.md exception after operator approval).

## 0.3.1-alpha5 — 2026-10-15

GPT 0.3.0 audit #5: CLI CAS paused→running resumes the same densify/rollback
checkpoint (pins/cursor/rebuild_state); HTTP keeps paused + requires_cli;
createUnlessRunning treats paused as active so a second logical run cannot start.

## 0.3.1-alpha4 — 2026-10-15

GPT 0.3.0 audit #4: pending sync_tombstone intents are reconciled before
state-filtered candidates and terminal completion; Undo refuses subsequent
edits and consumes already-restored receipts.

## 0.3.1-alpha3 — 2026-10-15

GPT 0.3.0 audit #3: Media publish resume reconciles already-landed destination
bytes (pub hash) and reuses a valid same-operation before-image after a lost
publication ACK; edited/ambiguous destinations stay recovery-required instead
of infinite `materializeBackup` stall.

## 0.3.1-alpha2 — 2026-10-15

GPT 0.3.0 audit #2: Prep Undo verifies intro/fulltext + bound asset before
treating a destination as restored; incomplete owned rows (asset_id=0 after
store-false) are repaired via Table::store; categories use the same
ownership/description/asset gate (no bare-ID shortcut).

## 0.3.1-alpha1 — 2026-10-15

GPT 0.3.0 audit #1: `ConvertBulkTagWriter` matches article commit/packet
contract — admit every tag/asset singleton before PENDING/BEGIN; lost COMMIT
ACK → `ConvertRecoveryRequiredException`; typed packet refusal with no
Executor row fallback for tags.

## 0.3.0 — 2026-10-15

First stable release. Version cut on the alpha202 code line after operator
VPS A–D + fault acceptance on
`cd072e3cd4c91fc7c5f19d8a69968580fcd8ba29ae452e8d3864ef8082b09807`.
Update stream tag is `stable`. No product logic changes in this cut.

## 0.3.0-alpha202 — 2026-10-15

GPT α200 residual groups #1–#11 + #13 candidate: idmap pair_sum fingerprint;
source-scoped reject resolve; unverified expected COUNT fail-closed; Prep Undo
snapshot match; menu undo journal before commit; tombstone before-image before
trash; foreign asset ref guard; uncertain COMMIT → recovery; exact media ledger
path + required publish ACK; singleton admission before PENDING; typed packet
refusal (no bulk fallback); linear sibling asset verifier + HTTP→CLI pause;
unique media manifest generation column.

## 0.3.0-alpha201 — 2026-10-15

Upsert dialect pin (Sonnet #1 / GPT α200 #12): `SqlUpsertDialect` selects
alias vs legacy from `$db->getVersion()` (MariaDB → VALUES; MySQL ≥ 8.0.19
→ AS new only, never deprecated VALUES fallback), caches the pin in
component params, and keeps unpinned probe-once as a safety net.

## 0.3.0-rc4 — 2026-10-15

Release candidate after operator A–D PASS on alpha200. Version cut only
(no product code changes). Root `update.xml` / `changelog.xml` publish-path
metadata fixed (SHA + Joomla changelog schema). A–D of alpha200 transfers
when the packaged payload differs only by version strings — see
`docs/acceptance-a-d.md`.

## 0.3.0-alpha200 — 2026-10-15

Final GPT rc3 audit gate (#17): same-artifact A–D candidate. Package
includes α186–α199 corrections (#1–#16). No new product features —
operator acceptance A–D on this immutable SHA before any RC/stable cut.
See `docs/acceptance-a-d.md`.

## 0.3.0-alpha199 — 2026-10-15

Engine-specific upsert dialect (GPT rc3 audit #16): shared
`SqlUpsertDialect` prefers INSERT…AS row alias; deprecated `VALUES(col)`
(or select+update) only when the engine rejects alias syntax — never on
deadlock, lock wait, permission, or disconnect. Preflight, media
manifest, K2 staging, and idmap share the gate.

## 0.3.0-alpha198 — 2026-10-15

Packet / singleton bounds (GPT rc3 audit #15): `SqlPacketBudget` never
inflates above 70% of real `@@max_allowed_packet` (65 KiB no longer
becomes 256 KiB); item/tag/UCM packers refuse oversize singleton rows
before mutation (no silent execute / retry of impossible SQL).

## 0.3.0-alpha197 — 2026-10-15

Source-bound media inventory (GPT rc3 audit #14): manifest generation pin
+ stale-row purge after walk; `unsupported_paths` / `imageless` counters
initialized; listing cache v3 with UTF-8 refusal, canonical sort-key
validation, directory stamp beyond second mtime, and memory admission;
K2 empty-dir Undo cleanup is time-budgeted and resumable.

## 0.3.0-alpha196 — 2026-10-14

Whole-run asset admission (GPT rc3 audit #13): Convert start refuses when
projected `#__assets` (live + plan creates) cannot fit `memory_limit`;
CLI `rebuildTick` shares long PREPARE wall budget with `rebuildAll`
(`PHP_SAPI === 'cli'`); RAM refusal is permanent
(`AssetPreparePermanentRefusalException`) and terminal-fails Convert /
rollback densify instead of endless retry.

## 0.3.0-alpha195 — 2026-10-14

Native parity / asset verifier (GPT rc3 audit #12): UCM core_* values
follow installed `#__content_types.field_mappings` (core_body ← introtext);
bulk create resolves category parents via `categories.asset_id` with name
agreement and refuses invalid pointers before write; densify verifier
rejects missing parents, wrong levels, and crossing sibling ranges.

## 0.3.0-alpha194 — 2026-10-14

Import completeness / rejects (GPT rc3 audit #2): live import pins
independent source COUNT expected; shortfall no longer uses attempt
counts (duplicates are informational); fail-closed reject ledger with
locator + source fingerprint; resolveIdentity on successful upsert;
readiness gates unresolved rejects for the current source generation;
xref upsert+cleanup in one transaction with zero-tag cleanup queue.

## 0.3.0-alpha193 — 2026-10-14

Source identity + destination admission (GPT rc3 audit #1+#11): validate
Convert pins every tick; refuse unpinned legacy jobs; idmap fingerprint
includes BIT_XOR(CRC32(pair)); SQL dump sample adds mid-file chunk;
cached preflight PASS pins destination #__content revision; alias
collision probes use native collation (no CAST AS BINARY).

## 0.3.0-alpha192 — 2026-10-14

Tombstones retry / snapshot / Undo (GPT rc3 audit #6): re-validate pinned
staging fingerprint under lease each tick; store_failed holds cursor for
retry (no premature completed_with_errors); refuse empty ownership; body
fields in live match; journal before-image + undoTombstone restore.

## 0.3.0-alpha191 — 2026-10-14

Atomic remap delta + required history (GPT rc3 audit #3): replaceRemaps
runs deactivate/history/upsert in one transaction; single-row upsert
requires history inside a transaction; bulk can join an outer TX.

## 0.3.0-alpha190 — 2026-10-14

Conservative Convert rollback (GPT rc3 audit #7): refuse empty/malformed
ownership fingerprints; body/modified/version digest; bound leaf asset
delete; UCM-base purge errors retryable; failed row keeps ledger in range;
expired budget with no query keeps more=true.

## 0.3.0-alpha189 — 2026-10-14

Prep Menu Undo / remap / rebuild repair (GPT rc3 audit #5): remap cascade
rewrites type=alias Itemid/aliasoptions; create journals owned IDs + prior
home flags for Undo; nested-set rebuild after partial insert fault; durable
rebuild_pending resume on next create.

## 0.3.0-alpha188 — 2026-10-14

Prep Undo partial receipt (GPT rc3 audit #4): bulk delete Undo consumes a
receipt only after every snapshot restores; partial success rewrites the
remaining receipt; no prune of unfinished action-undo rows; idempotent
already-restored destinations.

## 0.3.0-alpha187 — 2026-10-14

Convert recovery-required (GPT rc3 audit #8): typed
`ConvertRecoveryRequiredException` on uncertain bulk transaction rollback;
Executor stops tags/items without per-row fallback or cursor advance.

## 0.3.0-alpha186 — 2026-10-14

Media Undo/publish hardening (GPT rc3 audit #9+#10): preserve relative
`images/` folder paths in ledger payloads; pin newest copy run including
cancelled/failed with receipts (fail-closed scope); durable
`published=false` journal + atomic temp publish + resume before cursor
advance; no clean completed while publish is incomplete.

## 0.3.0-rc3 — 2026-10-14

RC polish: remove duplicate `COM_DEVARTK2MIGRATION_PREP_MENU_ERROR_EMPTY`
language key (en-GB / el-GR); `validate.php` now fails on duplicate `.ini`
keys. Re-run acceptance A–D on this artifact before stable.

## 0.3.0-rc2 — 2026-10-14

Release candidate after Pre-RC acceptance A–D on alpha185
(`aca121950fd70a63d76dcdcb5b3300ec8ce44e01a3704ebe8cca852507263350`).
Feature freeze; no product code changes beyond version cut.

## 0.3.0-alpha185 — 2026-10-14

Same-artifact acceptance ops (GPT #20): acceptance A–D tied to one SHA;
ops-guide documents integer CLI flags, scoped `--convert-job` vs global
Convert rollback, SELECT-only source, and destination-asset vs densify
semantics; run-record JSON example; Convert/Rollback help strings clarified.

## 0.3.0-alpha184 — 2026-10-14

Platform / deprecation compliance (GPT #19): package `preflight` refuses
PHP &lt; 8.3 and Joomla &lt; 6 before writes; Preparation media form uses
FormFactory; Table store failures throw without `getError()`; Query
`setLimit` instead of deprecated `setQuery(query, offset, limit)`;
preflight alias work upsert prefers `INSERT … AS new`.

## 0.3.0-alpha183 — 2026-10-14

Atomic job lifecycle + single browser request (GPT #18): `JobRepository::update`
is CAS on expected status; dry-run await-preflight re-reads parent and does
not replace a cancelled child; `job-batch.js` uses immutable per-request
generation so stale abort/finally cannot reset busy or stack fetches.

## 0.3.0-alpha182 — 2026-10-14

Heavy-phase time/bytes budgets (GPT #17): preflight fast COUNTs are
phased one-per-tick; duplicate finalize uses COUNT + LIMIT samples;
CLI/sync Convert use durable `runToCompletion` (not full `scanItems`);
shared `SqlPacketBudget` for bulk/UCM inserts; Prep category/menu create
stops between chunks on HTTP wall budget (committed rows kept).

## 0.3.0-alpha181 — 2026-10-14

Native relations + assets hierarchy (GPT #16): densify reparents orphans
under the canonical ROOT (never parent_id=0); VERIFY requires single
root, unique lft/rgt, max(rgt)=2n, parent containment; bulk UCM resolves
`ucm_language_id` from `#__languages` and refuses missing core/tag rows;
pilot verify checks category asset pointer, UCM content/base, and article
asset parent.

## 0.3.0-alpha180 — 2026-10-14

Lost-ACK / uncertain SQL recovery (GPT #10): job `create()` only uses
`createUnlessRunning` (reconcile after gone-away; never blind
`insertObject` replay); bulk article/tag writers treat failed
`transactionRollback` as recovery-required (no mutation retry).

## 0.3.0-alpha179 — 2026-10-14

Media dry/inventory isolation (GPT #14 slice): dry-run never truncates or
writes the shared media_manifest (walks source directly); listing cache
is version-sealed with complete/count checks; unreadable dirs fail the
job (not empty EOF); paths longer than varchar(512) are refused.

## 0.3.0-alpha178 — 2026-10-14

Durable import rejects (GPT #3 residual): new
`#__devartk2migration_import_rejects` ledger records every oversize /
memory / truncated skip beyond job-JSON sample caps; completeness syncs
`rejects_durable` + sample; unresolved rejects keep
completed_with_errors.

## 0.3.0-alpha177 — 2026-10-14

Media source/outcome honesty (GPT #13): connector/query failures never
count as EOF; start refuses live-count failures; pin path + K2 backend
kind + connection dataset fingerprint; conflicts force
completed_with_errors; AJAX exposes missing/conflicts; sanitize
client last_error (detail stays server-side).

## 0.3.0-alpha176 — 2026-10-14

K2 media Undo ownership (GPT #11 slice): K2 XL uses the same preexisting
skip/backup policy as site images; journals normalized paths under
images/; Undo restores preexisting K2 backups; media Undo pins copy
run_id + ledger high-watermark so a later copy run is not reversed.

## 0.3.0-alpha175 — 2026-10-14

Media publish/restore integrity (GPT #12 slice): revalidate source/dest
immediately before each publish; journal plans backup path before
materialize (journal-before-backup); store pub_sha256 and preserve
edited destinations on Undo; recognize already-restored when backup is
gone but dest matches orig_sha256; K2 destination guard before mkdir;
flush respects batch deadline between files.

## 0.3.0-alpha174 — 2026-10-14

Prep Undo integrity (GPT #7): consume action-undo receipt only after
verified restore; article/category restore checks Table::store(); bulk
item delete always keeps before-image snapshots in memory-safe chunks;
category create rebuilds nested-set after a mid-run chunk failure.

## 0.3.0-alpha173 — 2026-10-14

Planner/dry-run integrity (GPT #6): finalize/conflict COUNTs use EXISTS so
duplicate pending receipts do not fan out; create+conflict+finalize must not
exceed staging total (no negative skip clamp); destination alias collision
examines every foreign owner id, not only min(id).

## 0.3.0-alpha172 — 2026-10-14

Idmap integrity (GPT #5): replaceRemaps keeps unchanged frozen rows and
only deactivates non-frozen deltas; bulk upsert validates duplicate
source/dest, runs in a transaction, and requires history; allocationFloor
stays sequential when Convert lifts native MAX into the reserved band;
tag ROOT clash uses effective idmap destination.

## 0.3.0-alpha171 — 2026-10-14

Import integrity (GPT #3): staging_imported IF runs before content
assignments (MariaDB left-to-right clock fix); completeness fails on any
upserted→staging shortfall (not half-empty); incremental tags_xref upserts
replacement rows before deleting stale watermark ids.

## 0.3.0-alpha170 — 2026-10-14

Tombstones require clean Full import `completed` only
(failed/completeness_failed=0); store_failed trash is not journaled
(retriable); Convert rollback refuses a different run_id scope (SCOPE_BUSY).

## 0.3.0-alpha169 — 2026-10-14

Integrity slice: JobRepository::create routes through createUnlessRunning
(lost-ACK cannot spawn a second running job); SQL-file fingerprint includes
mtime/size/head+tail sample; K2 media source query failures fail the job
instead of false EOF completion.


## 0.3.0-alpha168 — 2026-10-14

Media Undo: ledger paths normalized under `images/` (fixes K2
`images/images/...` resolve); K2 cleanup no longer blanket-deletes residual
files. Asset densify: preserve `restart` across PREPARE→FLUSH so VERIFY churn
can abort after >3.


## 0.3.0-alpha167 — 2026-10-14

Final-hardening slice: Convert `run_generation` no longer includes destination
content watermarks (own writes no longer self-invalidate). Prep/Convert menus:
unresolved K2 component links created unpublished; `template_style_id` forced
to 0; foreign-ID and alias-conflict reporting; HTTP create cap 2000; rebuild
warning; home-clear confirm. Ops guide: Prep create vs Convert Rollback.


## 0.3.0-alpha166 — 2026-10-14

Prep UX: Categories Create moved below A/B; bulk create missing menus in
Preparation after A/B (preserved IDs + source aliases, K2 link rewrite, nested
rebuild). Convert menus still skips existing rows.


## 0.3.0-rc1 — 2026-10-14

First release candidate after Pre-RC acceptance A–D on alpha165. Feature freeze;
operator docs (`docs/acceptance-a-d.md`, `docs/ops-guide.md`); no product code
changes beyond version cut.


## 0.3.0-alpha165 — 2026-10-14

Scale/polish: HTTP PREPARE wall-time admission (A156-05) with CLI bypass; bulk set-based idmap remaps; planner mb_strtolower; skip ledger recordMany; JobRepository::create alive-db retry; dry-run injected jobs; K2 media staging-source cache; CLI-only preflight comment.


## 0.3.0-alpha164 — 2026-10-14

Pins/generations: ConvertRunIdentity source + admission generation; dry-run same-generation as preflight (A156-03/04); N-tick pin revalidation; conditional staging_imported bump; items dry-run summarize sub-phases (one heavy COUNT per tick).


## 0.3.0-alpha163 — 2026-10-14

Sync integrity: tombstone ownership fingerprint check; Full-import staging snapshot gate (A156-02); windowed edit scan cursor.


## 0.3.0-alpha162 — 2026-10-14

Data-correct hardening: ledger (entity_type,action,entity_id) index; incremental stale k2_tags_xref cleanup; media/convert catch→failed; atomic createUnlessRunning job starts; preflight materialize/collisions loop-until-deadline.


## 0.3.0-alpha161 — 2026-10-14

Safe-perf hardening: media manifest cursor uses utf8mb4_bin PK without CAST; atomic walker listing cache; densify restart>3 terminal abort (no infinite loop); PREPARE overrun CLI hint.


## 0.3.0-alpha160 — 2026-10-14

Resumable time-budgeted media rollback jobs; ledger-driven K2 Undo; micro-batched write-ahead ledger INSERTs before publish.


## 0.3.0-alpha159 — 2026-10-14

Media journal fail-closed before publish; unique backup paths; Undo blocks older ops on failed path restore; incremental import watermarks k2_tags_xref.


## 0.3.0-alpha158 — 2026-10-14

Preflight scale: chunk-scoped idmap lookups; alias work varchar(255)+full index; single-pass duplicate scan; purge stale alias work.


## 0.3.0-alpha157 — 2026-10-14

HTTP Convert items admits via fingerprint-pinned completed batch preflight (no sync scan); exclusive continueConvert job_id routing; stale 30m cancel for preflight/dry-run.

## 0.3.0-alpha156 — 2026-10-13

Faster progress-bar paint (defer first heavy tick); dashboard step-by-step migration guide before collection promo.

## 0.3.0-alpha155 — 2026-10-13

Batched HTTP dry-run for tags/items (same checks as live, no writes) with progress bar — Cloudflare-safe.

## 0.3.0-alpha154 — 2026-10-13

Preflight UX: unique flash lines with counts; hide Pause/manual continue on batch done; sync Latest jobs line from AJAX.

## 0.3.0-alpha153 — 2026-10-13

Batched HTTP item preflight (anti-524) with progress UI; dashboard Options hub next to Rollback.

## 0.3.0-alpha152 — 2026-10-13

Convert Legacy K2 URLs: title (.htaccess); Apache paste note; add nginx rewrite snippet + copy.

## 0.3.0-alpha151 — 2026-10-13

Preparation: remove leftover Destination users (sample) accordion from hub and users step.

## 0.3.0-alpha150 — 2026-10-13

Dashboard: info/warnings below hubs; DevArt collection promo (en-GB/el-GR) + DevArt News image.

\n## 0.3.0-alpha149 — 2026-10-13

Dashboard hub cards (Article Tools pattern): colored icons, full-width action buttons, centered intro; status/preflight as secondary info cards.

\n## 0.3.0-alpha148 — 2026-10-13

Fix admin page headers: native width/height attrs + max-width per image (886/1200), Article Tools responsive pattern (width 100%, height auto, no upscale).

\n## 0.3.0-alpha147 — 2026-10-13

Admin chrome: centered responsive page headers (media images), centered footer, DevArt red active submenu (Article Tools pattern).

## 0.3.0-alpha146 — 2026-10-13

Agreed P1/P2 from α145 audits: Sync tombstone gate + OpsBackupAck; Media Undo LIFO/checksum/durable journal + manifest scale; densify whole-run PREPARE; alias candidate-only SQL; Import/Media fingerprint refuse; richer Convert media/idmap pins; qa-importer-matrix in qa-gate; joomla.asset.json sync.
## Template 1.1.0 — 2026-08-04

- Restored generic documentation placeholders for new product bootstrap.
- Added Joomla production-series and workspace awareness to `scripts/validate.php`.
- Added PHP minimum, compatibility targets, and extension checks to validation.
- Added stronger package and extension naming validation.
- Made `scripts/build.php` validation-first with a source integrity guard.
- Preserved the original simple template architecture.

Product change history for bootstrapped repositories begins below this template
section.

## 0.3.0-alpha145 — 2026-10-13

- F1: Convert jobs pin staging + prep-rules fingerprints; resume refuses on change.
- F2: Pilot verify exact tag id set + workflow stage; preflight empty effective
  alias and foreign destination alias collisions.
- Gate: importer pause/memory matrix; integration harness crash/resume
  (`DKM_CRASH_MODE=throw`).

## 0.3.0-alpha144 — 2026-10-13

- Asset densify PREPARE: yield without work-table writes when the tick
  budget is spent (before prepare / after load / after graph / mid-number);
  memory admission refuses oversized trees before mutation.
- qa-gate contracts for densify prepare yield + memory admission.

## 0.3.0-alpha143 — 2026-10-13

- Media: hash-aware preexisting decision (same size/mtime + different bytes
  no longer silent overwrite); ledger JSON payload with backup path +
  orig sha256; Undo restores from `images/k2migration-backup/` before
  clearing ledger.
- MediaPathGuard: shared symlink/parent containment for copy, backup and
  undo; K2 destination writes use the same safe-path check.
- Convert: `DKM_CRASH_AFTER_BATCHES=N` hard-exit after durable checkpoint
  (test/gate only).
- `scripts/qa-gate.php` release gate (unique temp dir, full unit suite +
  product contracts). Feature freeze: no new product features.

## 0.3.0-alpha142 — 2026-10-13

- Memory guard before tuple parse: estimate `tupleLen×5.0 + 12MB` so a 14MB
  row is refused under 64M instead of OOM fatal in MysqlStringDecoder
  (closes the ~57–71MB hole left by 2.5×+8MB).
- Media site-images: default preexisting policy `skip_conflict` (report
  conflict; do not overwrite differing clean J6 files). Optional
  `replace_with_backup` copies aside to `images/k2migration-backup/` first.
- Tests: Convert crash/idempotency journal contract; importer e2e spill/
  pause/memory contracts; memory-guard estimate; media preexisting policy.

## 0.3.0-alpha141 — 2026-10-13

- SQL ingest spill ceiling = maxTuple+1MB (items 25MB) so mid-tuple pauses
  no longer lossy-drop multi-MB k2_items rows under the ≥4MB scan guarantee.
- Memory guard before parsing large tuples: skip with `dropped_rows` /
  `skipped_memory` instead of an uncaught OOM fatal.
- PerformanceProfileService resolves once per request (no preflight storm
  on every upsertMany).
- Asset densify flush uses keyset `seq` ranges; VERIFY restarts PREPARE when
  `#__assets` count drifts mid-rebuild.
- Preflight warns when the site is online during densify-sensitive work.
- Rollback option: also delete edited articles (ignore ownership fingerprint)
  with explicit confirmation.

## 0.3.0-alpha140 — 2026-10-13

- Full Convert ownership fingerprint beyond alias/catid: title, created,
  created_by, language, access stored on PENDING/CREATE journal payloads.
- Rollback compares present fingerprint fields against live `#__content`
  before delete; older alias/catid-only journals stay compatible.
- Rollback UI shows journal vs live fingerprint summaries + mismatch fields.

## 0.3.0-alpha139 — 2026-10-13

- Resumable `#__assets` densify: number into work table
  `#__devartk2migration_asset_rebuild`, then chunked UPDATE with job
  checkpoint (`rebuild_state` / flush offset). Safe across HTTP time budgets.
- Convert items + Convert rollback resume densify ticks until verify PASS.
- `rebuildAll()` remains a blocking loop of ticks for CLI `--workers` parent.

## 0.3.0-alpha138 — 2026-10-13

- Editorial sync AJAX auto-loop (Check hub) + CLI `devartk2migration:sync`
  (`--phase=edits|tombstones`).
- Convert bulk tag writer (multi-row `#__tags` + assets) for OP_CREATE batches.
- Performance profiles (`auto` / `shared` / `vps` / `dedicated`): batch time
  budget, plan chunks, sync batch sizes, SQL scan bytes, upsert byte caps.
- Component config: performance profile + optional batch time budget override.
- Prep categories/tags: empty-destination and large-site SQL-side counts with
  capped conflict/orphan samples (no full in-memory ID maps).

## 0.3.0-alpha137 — 2026-10-13

- Workspace lease re-verify after DB reconnect (`IS_USED_LOCK` /
  `CONNECTION_ID`); abort tick when lease is lost.
- Cancelled Convert / Media / Menu / Sync / Import jobs no longer mutate
  after cancel (re-read status under lease).
- Convert cursor advances only through a contiguous durable plan prefix
  (prep failure + bulk/fallback cannot skip pending IDs).
- Staging `upsertMany` and UCM relations INSERT use byte budgets; bulk
  Convert retries MySQL deadlock / lock-wait (1213/1205).
- Live import selects staging columns only; k2_items fetch capped at 20/tick.
- CLI `--workers`: file logs (no pipe deadlock), lease heartbeat, terminate
  on spawn/lease failure, densify only after all shards exit 0.
- Environment preflight: GET_LOCK probe, disk float handling, blockers +
  shared/vps/dedicated profile hint.

## 0.3.0-alpha136 — 2026-10-13

- Dashboard host environment preflight (memory, max_execution_time,
  max_allowed_packet, sql_mode, free disk).
- Convert bulk INSERT packs by SQL byte budget (max_allowed_packet headroom),
  not a fixed 250-row chunk only.
- `AssetNestedRebuild` streams adjacency in pages into SplFixedArray (lower peak
  memory on large `#__assets` trees).
- SQL import EOF: staging COUNT vs ingest `by_table` completeness report;
  `failed_by_table` counters on row failures.
- Convert batch prefetch for tags / users / categories; multi-row journal inserts.
- CLI `devartk2migration:convert --workers=N` (items): parent holds workspace
  lease, children run MOD(id) shards with `--no-lease --skip-final-rebuild`,
  then one final asset densify.
- Repo stress unit test for multi-MB mid-tuple pause/resume.

## 0.3.0-alpha135 — 2026-10-13

- Rollback ownership policy (α): when live article alias/catid no longer matches
  the Convert journal, do not delete the article; keep a `foreign_skipped`
  ledger row and list it on the Rollback screen.
- SQL ingest: mid-tuple ticks guarantee ≥4MB scan progress before yielding to
  the soft time budget, so multi-MB base64 rows are not stall-skipped under
  tight host limits.

## 0.3.0-alpha134 — 2026-10-13

- Convert / Rollback: re-plan within the same HTTP tick until the wall-clock
  budget is spent (fewer admin bootstrap round-trips on large jobs).
- Convert AJAX progress uses `expected_total` from the job checkpoint (no
  per-tick `getReadiness()`).
- `job-batch.js`: consecutive error counter with exponential backoff; stop after
  ~8 failures instead of spinning on `ok:false, running:true`.
- AJAX error payloads strip filesystem-like paths before reaching the browser.
- Rollback: `AssetNestedRebuild` instead of recursive `Asset::rebuild`;
  ownership check (alias/catid) before deleting journalled article ids;
  time-boxed multi-chunk ticks.
- SQL ingest pause/resume unit test executes the real static policy extracted
  from `K2DatabaseImporter` (not a mirrored closure).

## 0.3.0-alpha133 — 2026-10-13

- Shared AJAX auto-continue for Convert (tags/items/menus), Media, and Rollback:
  `job-batch.js` + `continueAjax` endpoints with progress, Pause/Resume, and
  proxy/timeout retry (Database-style). Manual full-page Continue remains as
  fallback.
- Media batches are time-boxed (~2.8s) with a higher row ceiling instead of a
  fixed batch of 40 files/items per click.

## 0.3.0-alpha132 — 2026-10-13

- P0 fix: SQL ingest mid-tuple resume stores `scan_pos` and continues from that
  buffer offset (with depth/in_string). Alpha129 kept the buffer but reset `i`
  to 0, which re-parsed the prefix with the wrong string machine and dropped
  large stretches of VALUES rows under tight batch budgets.

## 0.3.0-alpha131 — 2026-10-13

- Release prep (US1): root `update.xml` + `changelog.xml` for GitHub-hosted
  Extension Manager updates; package manifest `updateservers` pointing at
  `devartgr/joomla-devart-k2migration`. Documented in `docs/update-server.md`.
  Alpha builds tagged `testing` until a stable public release.

## 0.3.0-alpha130 — 2026-10-13

- A126-05 pilot/bulk parity: sample Convert OP_CREATE uses the same
  `ConvertBulkItemWriter` path as full Convert (not per-row Content::store).
- Preflight `duplicate_alias` checks effective destination category (active
  category idmap) + `ApplicationHelper::stringURLSafe` final alias, so remaps
  and title→alias collisions block before bulk INSERT.

## 0.3.0-alpha129 — 2026-10-13

- SQL file ingest resume (P0-5): on batch timeout inside an open VALUES tuple,
  do not advance `consumed` or trim the parse buffer (fixes row loss under tight
  wall-clock budgets on shared hosting). Optional test hook
  `setSqlBatchMaxSecondsForTests()` for harness timing.

## 0.3.0-alpha128 — 2026-10-13

- A126-03 AssetNestedRebuild: dense first-child/next-sibling adjacency +
  iterative DFS (no recursive PHP, no full assigned map), missing parents and
  cycles reparented to `parent_id=0`, chunked UPDATEs, post-densify lft/rgt
  verify. Keeps alpha127 `rebuild_assets` retry contract.

## 0.3.0-alpha127 — 2026-10-13

- P0 bulk safety after alpha126 re-audit (A126-01..04 subset):
  - Asset titles truncated to 100 chars (Joomla `Table` parity) before bulk INSERT.
  - Bulk native writes + CREATE journal in one InnoDB transaction; PENDING
    write-ahead stays outside; no foreign planned-ID purge on failure.
  - Convert item cursor advances only for durable executed source IDs (sample +
    full); bulk fallback records `bulk_fallbacks` / `bulk_last_error`.
  - Densify failure keeps `rebuild_assets` and refuses terminal completion.
  - Convert rollback covers `pending` + `create` ledger rows; cancelled jobs
    refuse further destination mutations.
- Still deferred: set-based/low-memory rebuild, Database import resume under
  tight time budgets, pilot/bulk writer parity, ownership fingerprints.

## 0.3.0-alpha126 — 2026-10-12

- H6 bulk scale acceptance on `joomla6_tests`: 100k @ 281.64 arts/s;
  300k @ 255.21 arts/s (failed=0), densify ~6.3s, set-based rollback OK.
  Documented in `docs/convert-asset-benchmarks.md`.

## 0.3.0-alpha125 — 2026-10-12

- H5 bulk OP_CREATE Convert: `ConvertBulkItemWriter` multi-row `#__assets` +
  `#__content` (no per-row `Content::store`), batch workflow/frontpage/UCM/tags,
  `AssetNestedRebuild` densify at items job end (`rebuild_assets_ms`). Samples
  and finalize keep the row path; bulk failure purges partial ids then falls
  back. Default item plan chunk 200.

## 0.3.0-alpha124 — 2026-10-12

- H5 P0 fast Convert rollback: ledger cursor uses BATCH+1 lookahead (no per-tick
  `COUNT(*)`), article phase set-based side-effect purge + content/asset DELETE
  (BATCH=200) with row-at-a-time fallback, deferred `Asset::rebuild` after items
  (`rebuild_assets_ms` in job stats), ledger index
  `(run_id, entity_type, action, id)`.

## 0.3.0-alpha123 — 2026-10-12

- H4 Convert scale measurement: hardened `scripts/measure-convert-benchmarks.php`
  (test-DB safety, `--skip-seed`, owned `dkm-bench-` cleanup, toggles
  `behaviour/versionable` + `actionlog/joomla` with restore) +
  `scripts/seed-convert-benchmark.php` + unit coverage.
- Recorded 300k Convert on `joomla6_tests`: 14.49 arts/s, `store_avg` 57ms,
  failed=0. Plugin proof (200, history on): histΔ=0 / actionΔ=0 on Table path.
- Convert readiness / language: history checklist points at
  `plg_behaviour_versionable` (not legacy `content/history`).
- **Decision:** bulk OP_CREATE + set-based rollback is required for ~300k
  production targets; documented in `docs/convert-asset-benchmarks.md`.

## 0.3.0-alpha122 — 2026-10-12

- H3 pilot Promote gate: full tags/items/menus Convert requires pilot PASS +
  Promote with matching staging fingerprint (`convert_require_pilot`, default
  On). Admin Evaluate/Promote; CLI `--sample --check --promote`.
- Pilot orchestration fixes: scoped `startTagsSample` (not full staging tags),
  propagate tag failures, rollback both tag+item sample jobs, refuse mixing
  running full vs sample jobs. `relatedTagIds` includes remapped tag id=1
  candidates. Automatic sample verifies (asset/workflow/tags+UCM/images).

## 0.3.0-alpha121 — 2026-10-12

- H2 real-Joomla Convert integration harness: CLI
  `devartk2migration:integration-test` + launcher
  `scripts/run-integration-convert.php`. Safety gate requires DB suffix
  `_test`/`_tests` or `DEVART_K2MIGRATION_ALLOW_INTEGRATION=1`. Seeds synthetic
  staging, runs live ConvertExecutor (tags/items), asserts asset/workflow/
  tag_map+UCM/frontpage/dates, fault-injects missing asset and confirms
  finalize repair, then rolls back by convert job run_id.
- CLI tag convert: null-safe identity when no logged-in user (no PHP warning).

## 0.3.0-alpha120 — 2026-10-12

- H1 finalize asset repair: `applyItemFinalize` calls `Content::store(false)` when
  `asset_id < 1`, restores modified dates, and refuses journal CREATE until asset
  exists (needs_repair stays open on failure).
- Compensatory purge and Prep item bulk delete call
  `NativeArticleRelations::purgeArticleSideEffects` before `Content::delete`.
- K2 media copy uses staging source item id only (drops reverse idmap lookup that
  could copy the wrong XL under remapped dest collisions).

## 0.3.0-alpha119 — 2026-10-12

- JED hygiene: remove duplicate `COM_DEVARTK2MIGRATION_CHECK_MENUS_HELP` (en-GB /
  el-GR); admin submenu menus use `link` only (drop redundant `view` attribute).
  Update server / `update.xml` + root `changelog.xml` deferred to release
  (`joomla-devart-k2migration`, same pattern as other DevArt packages).

## 0.3.0-alpha118 — 2026-10-12

- Menus Prep alias diagnosis: keyset batches of 500 instead of loading every
  `type=alias` row into PHP (large-site memory safety). Hub/check menus paths
  already use COUNT + capped lists. Bulk-assets still deferred.

## 0.3.0-alpha117 — 2026-10-12

- G3 workspace lease: all mutating jobs share one `GET_LOCK` fence (import /
  media / convert / rollback / sync cannot run concurrently).
- G4 media symlink/undo: reject source symlinks and symlink path components;
  site-images rollback skips `preexisted` rows and never unlinks via symlink.
- G5 `idmap_history` table + history on remap changes; tag/item/menu remaps
  frozen after Convert CREATE for that source id.

## 0.3.0-alpha116 — 2026-10-12

- G1/G2 audit follow-ups: media manifest `rel_path` uses `utf8mb4_bin` + BINARY
  keyset compares (case-sensitive resume); Convert includes remapped source tag
  id=1 (R04); article `created_by`/`modified_by` and Prep user create resolve
  through user idmap (R03); preflight missing-author check honours remaps.

## 0.3.0-alpha115 — 2026-10-12

- F5 honest status: `PROJECT_STATUS.md` / `docs/current-state.md` list actually
  delivered F0–F5 vs deferred; CLI exit **3 = interrupt** (not 130) documented;
  Convert engine docs updated for pending/finalize durability. No git commit
  without explicit approval.

## 0.3.0-alpha114 — 2026-10-12

- F4 polish: menu home de-dup + Check warning; unrewritable K2 links → convert
  issue; dry-run `ok`/`can_apply` false when `would_conflict > 0`; stable idmap
  allocation floor (margin once, then sequential); source SSL param; CLI
  `--password-env`; Clear temporary also purges component tmp spill files.
  Tag `getError()` already included from F1.

## 0.3.0-alpha113 — 2026-10-12

- F3 Convert measurements: recorded 5k / 20k / 100k telemetry on joomla6_tests
  into `docs/convert-asset-benchmarks.md` (history on/off at 5k). Bulk-assets
  still deferred. Added `scripts/measure-convert-benchmarks.php`.

## 0.3.0-alpha112 — 2026-10-12

- F2 pilot sample: Convert sample of N articles (default 200, spread categories /
  tags / featured); CLI `--sample` / `--check` / `--rollback-run`; admin sample
  button; job-scoped Convert rollback via ledger `run_id`.

## 0.3.0-alpha111 — 2026-10-11

- F1 Convert recovery: write-ahead journal `pending` before Content::store;
  `OP_FINALIZE` repairs incomplete natives (no second insert); post-store failures
  open `needs_repair` issues; optional purge param default Off; removed pretend
  Nested transactions and write-path probes; Check reports pending/workflow gaps.

## 0.3.0-alpha110 — 2026-10-11

- F0 SQL EOF: residual dump buffer is drained statement-by-statement; leftover
  keeps the import job `running` (never ingest a multi-statement blob as one SQL).
- F0 large-site Prep: PK overlaps split into owned (alias match or journal CREATE)
  vs unknown conflicts; owned + new rows no longer block Convert readiness.

## 0.3.0-alpha109 — 2026-10-11

- E3 hygiene: removed legacy `SqlFileImporter` (SQL dumps use Database hub /
  `K2DatabaseImporter` only); package `postflight` deletes the orphan on update.
  Documented write-path dry-run probes as permanently closed (implicit-commit
  risk); dry-run remains planner + preflight.

## 0.3.0-alpha107 — 2026-10-11

- E2 low cleanups (Devin N5/N7/N9): K2 media ledger records `preexisted` before
  `copy()`; Check tombstone counts/samples filter `ct.state > -2` like apply;
  removed dead `applyStoredRemaps()` stubs from Prep*Actions.

## 0.3.0-alpha106 — 2026-10-11

- E1 Convert hub load: readiness no longer runs full-staging item preflight on
  every page open (deferred row + “Run item preflight” button). Convert start,
  dry-run, and CLI `preflight` still execute the full scan. Planner summarize
  uses the same hub-speed path.

## 0.3.0-alpha105 — 2026-10-11

- D6 CLI hygiene: `EXIT_BUSY=4` distinct from interrupt/fail; convert leases
  return busy (not hard fail); `--json` emits NDJSON progress per batch;
  new `devartk2migration:verify` (Check reports).
- Alias change logging: when `stringURLSafe` alters a K2 alias, journal stores
  `alias_source` / `alias_changed` and job stats count `alias_changed`.
- SqlGuard tokenization: keyword / side-channel scans ignore string and
  quoted-identifier literals (fewer false positives).
- Tests: NativeArticleRelations splitBody, SqlGuard tokenization, CliSupport
  busy/progress; removed tracked `.DS_Store` files.

## 0.3.0-alpha104 — 2026-10-11

- D5 stable ID floor: remap allocation uses `max(known ids) + 1 + 1_000_000`
  so live K2 growth is unlikely to collide with remap destinations during
  incremental migration.

## 0.3.0-alpha103 — 2026-10-11

- D4 full staging preflight + dry-run: duplicate alias is a blocker; added
  empty/long alias, access, dates, tag xref, tag-name checks; planner tallies
  create/skip/conflict across all staged rows; dry-run stays write-free
  (`full_plan` mode — Nested LOCK TABLES cannot be safely rolled back).

## 0.3.0-alpha102 — 2026-10-11

- D3 Convert prefetch: batch idmap / journal / destination lookups per plan
  tick; batch `convert_issues` resolve; `plan_ms_*` telemetry; menus use the
  same pattern. Measurement guide updated for 5k/20k/100k runs.

## 0.3.0-alpha101 — 2026-10-11

- D2 path/secret safety: admin `path_extra_roots` (one absolute directory per
  line) extends PathAllowlist beyond Joomla/parent/tmp; `SourceSecretCipher::seal`
  fails closed when the site secret is missing (no plaintext password storage).

## 0.3.0-alpha100 — 2026-10-11

- D1b data-loss fixes: SQL EOF drain no longer discards leftover buffer when the
  statement budget stops; missing SQL capture counts as failed; site images job
  correctly transitions `manifest` → `copy` (checkpoint merge order).

## 0.3.0-alpha99 — 2026-10-11

- D1 store + rollback + dry-run safety: Tag/Category `store()` return checked;
  tag/category rebuild failures surface; Convert rollback deletes ledger only
  after successful native delete or missing row; dry-run is planner/preflight
  only (no write-path probe — LOCK TABLES can implicit-commit).

## 0.3.0-alpha98 — 2026-10-11

- C12 security/hygiene: encrypt source DB password at rest (AES + site secret,
  com_users pattern); `PathAllowlist` for SQL/media/source roots; SqlGuard
  blocks INTO OUTFILE / LOAD_FILE / SLEEP / BENCHMARK; importer/tmp dirs 0700;
  archive checkpoints removed (`.gitkeep` only); git repository initialized.

## 0.3.0-alpha97 — 2026-10-11

- C11 full CLI: `devartk2migration:import`, `:media`, `:preflight`, `:convert`,
  `:rollback` via console plugin; shared `CliSupport` exit codes 0/1/2/3,
  `--json`, `--batch` / `--time-limit`, `--stop-on-fail`, SIGTERM/SIGINT;
  unit test for CLI exit mapping.

## 0.3.0-alpha96 — 2026-10-11

- C10 preflight + real dry-run: staging checks (empty titles, orphan catid,
  duplicate aliases, missing authors); Convert blocked on empty title /
  orphan category; dry-run runs aggregate plan plus transactional write-path
  probe (create then rollback) for tags/items.

## 0.3.0-alpha95 — 2026-10-11

- C9 Convert performance: time-boxed batches (plan 40 items / 80 tags / 40
  menus, HTTP 4.5s / CLI 30s budget); workflow default-stage cache per
  category; ops/budget telemetry; Convert checklist warns when history or
  actionlog plugins are on (operators disable for large runs — no auto-toggle).

## 0.3.0-alpha94 — 2026-10-11

- C8 media manifest walk: directories sort as `name/` so siblings `a.jpg` /
  `a-1.jpg` are never skipped after cursor `a/…`; binary resume index inside
  the current directory; extracted `MediaDirectoryWalker` + unit tests.

## 0.3.0-alpha93 — 2026-10-11

- C7 Convert reliability: `#__devartk2migration_convert_issues` with per-entity
  reason; one end-of-job retry for retryable failures; journal insert no longer
  swallows errors (rolls back create); planner `OP_CONFLICT` for dest mismatch
  instead of silent skip; idmap `dest_id_active` unique + allocation floor
  includes active remap dests; prep access mode defaults to unset (must choose
  Preserve or Override); `checked_out = NULL` on Convert/Prep creates; Convert
  hub lists open issues. Tests: access mode, idmap floor, conflict codes.

## 0.3.0-alpha92 — 2026-10-11

- C6 hygiene: `ReadOnlyDatabase::createQuery()` + bulk `getQuery(true)` →
  `createQuery()`; staging upsert uses MariaDB `AS new` (VALUES fallback);
  SqlGuard allows `SHOW GRANTS`; soft writable-grants warning on source test;
  docs/README sync with C1–C5 delivered state.

## 0.3.0-alpha91 — 2026-10-11

- C5 media manifest resume: sorted per-directory walk with lexicographic path
  cursor (prune dirs before cursor) — no O(n²) full-tree restart each HTTP
  batch. Copy phase keyset unchanged.

## 0.3.0-alpha90 — 2026-10-11

- C4 real CLI: package includes `plg_console_devartk2migration` (registers
  `devartk2migration:convert`); package script enables the plugin on install.
  Removed dead component `boot()` CLI registration (ConsoleApplication does
  not boot components).

## 0.3.0-alpha89 — 2026-10-11

- C3 asset/batch metrics only: Convert job stats accumulate batch / store
  (asset) / relations / rebuild milliseconds; shown on Convert hub + CLI.
  Write path unchanged. Docs: `convert-asset-benchmarks.md` live telemetry guide.

## 0.3.0-alpha88 — 2026-10-11

- C2 Convert hardening: job leases on tags/items/menus batches; aggregate
  dry-run/summarize without full-row `SELECT *`; menu `component_id` resolved
  from rewritten link option; `completed_with_errors` when failed>0 (Convert,
  sync, rollback); Prep access stats/warn + Convert checklist warn on force
  rewrite.

## 0.3.0-alpha87 — 2026-10-11

- Convert native correctness (C1): shared `NativeArticleRelations` for Convert +
  editorial sync — separate introtext/fulltext, restore source modified dates
  after `Content::store`, checked store failures, `#__workflow_associations`,
  `#__content_frontpage` for featured, UCM + tags via `TagsHelper::postStore`.
- Convert rollback: purge UCM / tag maps / workflow / frontpage before article
  delete.

## 0.3.0-alpha86 — 2026-10-11

- Fix schema check false positive: alpha63 `ADD INDEX IF NOT EXISTS` misparsed as
  index name `IF` — use plain `ADD INDEX … /** CAN FAIL **/`.
- Check: editorial sync apply — batched overwrite of edited articles + trash
  tombstone candidates (explicit confirm + Continue).

## 0.3.0-alpha85 — 2026-10-11

- Rollback: resumable Convert rollback (ledger create only) — menus → articles
  → tags via HTTP batches with Continue.

## 0.3.0-alpha83 — 2026-10-11

- Check performance on large sites (≥10k articles): skip full CONCAT/orphan
  article scans; sample wrong-name; ACL uses NOT IN viewlevels; editorial sync
  uses PK joins when no remaps and samples edited/tombstone counts.

## 0.3.0-alpha75 — 2026-10-11

- Convert: legacy K2 URL htaccess rule with Copy code (Admin Tools friendly).
- Check: assets-first integrity summaries (missing/wrong-name/orphan) plus
  planned count/media/menu/ACL reports list.

## 0.3.0-alpha74 — 2026-10-11

- Preparation Step 7: image mode K2 sizes or fixed intro/full for all articles.
- Convert hub: how-it-works note, final mapping checklist (authors/access/
  images/remaps), Convert tags / Convert items + dry-run (engine writes next).

## 0.3.0-alpha73 — 2026-10-11

- Migration Plan: clear ordered flow list (Database → Media → Preparation
  Steps 1–7 → Convert tags/items → Menus last → Repairs → Check), with
  explicit note that saved ID remaps / Prep rules must be honoured.

## 0.3.0-alpha72 — 2026-10-11

- Admin submenu/dashboard order: Dashboard → Plan → Database → Media →
  Preparation → Convert → Check → Rollback.
- Preparation Step 6 Menus Solutions: A delete destination conflicts, B remap
  staging menu IDs (parent_id + alias targets, persistent remaps), C orphans/
  broken-alias note; URL/htaccess caution; dry-run bulk actions.

## 0.3.0-alpha71 — 2026-10-11

- Preparation hub: larger step icons; Step 7 Media files (transfer status,
  last sync, link to Media submenu, Convert intro/full image size settings).
- Admin submenu/dashboard: Preparation directly above Convert; Media after.
- Dry-run (no writes) for media site/K2 copy and Preparation tags/items bulk
  delete/remap.

## 0.3.0-alpha70 — 2026-10-11

- Preparation drill-in loads only the active step diagnosis (hub still runs
  full status once); large Back-to-hub button top-left with arrow icon.

## 0.3.0-alpha69 — 2026-10-11

- Preparation hub UI: Steps 1–6 as status cards (OK / needs action /
  Convert-ready); drill-in keeps existing Solutions; Recheck/Apply return to
  the open step; Back to hub refreshes badges from diagnosis.

## 0.3.0-alpha68 — 2026-10-11

- Preparation tags/items: bulk-first UI when conflicts > 100; clearer delete
  buttons (delete existing destination articles/tags); large bulk delete skips
  full Undo snapshots; bulk remap persists remaps once and refuses >10000
  (prefer destination delete for ~100k conflicts).

## 0.3.0-alpha67 — 2026-10-11

- Preparation Step 5 Items Solutions: missing = Convert-ready; A single/bulk
  delete destination articles; B single/bulk staging remap with tags_xref+menu
  cascade and persistent import remaps; Undo / Recheck.

## 0.3.0-alpha66 — 2026-10-11

- Preparation Step 4 Tags Solutions: missing = Convert-ready (not a conflict);
  A single/bulk delete empty destination tags; B single/bulk staging remap with
  xref+menu cascade and persistent import remaps; Undo / Recheck.

## 0.3.0-alpha65 — 2026-10-11

- Preparation Step 3 Solutions: A delete empty destination com_content category
  (with Undo snapshot); B remap staging category ID with cascade + persistent
  import remaps; Recheck after Apply. Convert remains conversion-only.

## 0.3.0-alpha64 — 2026-10-11

- Preparation large-site fast path (≥2000 staging or destination articles): one
  PK overlap COUNT instead of full match/alias JOIN scans (demo / extreme sites).
- Menus diagnosis via SQL aggregates (no full-row hydrate); alias targets checked
  with targeted IN lookups.
- UI notes for large-site and empty-destination item fast paths.

## 0.3.0-alpha63 — 2026-10-11

- Fix Preparation timeout (Cloudflare 524): items diagnosis fast-path when
  destination #__content is empty (no heavy JOINs on large staging).
- Drop LOWER() joins on item alias checks; add staging `idx_alias` (IF NOT EXISTS).

## 0.3.0-alpha60 — 2026-10-11

- Preparation Steps 3–5 diagnosis: categories / tags / items ID buckets.
- Tags ROOT (id=1) clash warning; items alias-clash check (SQL aggregates).
- Modules / templates remain deferred.

## 0.3.0-alpha60 — 2026-10-11

- Preparation Steps 3–5 diagnosis: categories / tags / items ID buckets.
- Tags ROOT (id=1) clash warning; items alias-clash check (SQL aggregates).
- Modules / templates remain deferred.

## 0.3.0-alpha59 — 2026-10-11

- Preparation step order: Categories → Tags → Items → Menus (menus after
  content identity they usually link to).

## 0.3.0-alpha58 — 2026-10-11

- Admin submenu order: Database before Preparation; new Convert before Check.
- Preparation Steps 3–5 skeletons (categories, tags, items conflict prep) and
  Step 6 menus (later renumbered in alpha59).
- Convert view skeleton (image/content options placeholders).
- Migration Plan display order aligned with workflow (C → A → B → D → F → G).

## 0.3.0-alpha57 — 2026-10-11

- Preparation Step 3 Menus diagnosis (IDs, menutype/type/link family, alias
  targets, orphan parents vs destination).
- Recheck + Undo actions beside Apply buttons (Undo only when staging snapshot
  exists; Create-users shows Recheck only).

## 0.3.0-alpha56 — 2026-10-11

- Removed Audit and Source Inventory from admin submenu / dashboard cards.
- Migration Plan rewritten as clear sections A–G (Preparation, Media, Database
  staging, article conversion, repairs, final checks) with deep links.
- Note for a future DevArt recommendations submenu.

## 0.3.0-alpha55 — 2026-10-11

- Step 1 Solutions: Author mapping first (default keep source). Keep shows
  Create missing users + usergroup; Force shows user dropdown + Apply author only.

## 0.3.0-alpha54 — 2026-10-10

- Preparation speed: drop full DestinationInspector + access GROUP BY on page load.
- Per-step Apply (author / access); no global Save.
- Step 1 Solutions: staging username/email fixes, ID remap, create missing
  users under chosen usergroup; uniqueness diagnosis is read-only Source/Destination pairs.

## 0.3.0-alpha53 — 2026-10-10

- Preparation Step 1: author mapping controls moved into Solutions (bottom).
- Step 2: keep source access vs override-for-all (override default) plus
  lightweight staging access distribution (COUNT/GROUP BY).

## 0.3.0-alpha52 — 2026-10-10

- Uninstall drops prep undo tables (same features as alpha51).

## 0.3.0-alpha51 — 2026-10-10

- Preparation: clearer username/email clash table (staging vs destination);
  green Solutions boxes per step; force-author UI only when selected (red warn);
  staging username/email fixes; one-level Undo for staging item/user rewrites.

## 0.3.0-alpha50 — 2026-10-10

- Preparation compact UI; closed detail sections with ✕ / ! problem flags.
- Step 2 Access level (default Public 1) + Step 1 optional force-all-articles
  author; settings persist in component params and rewrite staging k2_items
  on save and on every Database import.

## 0.3.0-alpha49 — 2026-10-10

- Preparation simplified to users only (K2 article authors / created_by).
  Usergroups UI removed; access levels are a Public (ID 1) policy note.

## 0.3.0-alpha48 — 2026-10-10

- Preparation: always show overview + destination users sample (`id`/`name`/`username`)
  even when staging diagnosis cannot run yet.

## 0.3.0-alpha47 — 2026-10-10

- Preparation Step 1: always-visible overview table (entity totals / matches /
  mismatches / missing) so the page stays scannable when detail lists are collapsed.

## 0.3.0-alpha46 — 2026-10-10

- Preparation Step 1: diagnose users / usergroups / viewlevels (match, mismatch,
  missing, uniqueness blocks) with per-section conclusion and recommended action.
- Stop Check from loading full ID maps (was hanging the admin on large dumps);
  point operators to Preparation for access diagnosis.
- IdCheckService loads only requested staging maps (never pulls k2_items for access prep).

## 0.3.0-alpha45 — 2026-10-10

- Replace regex-based inline base64 stripping with byte-scan
  (`InlineBase64Stripper`) so multi-MB `<img src="data:…;base64,…">` and
  CSS `url(data:…)` payloads are removed without PCRE backtrack/JIT limits.
- Add `tests/StripInlineBase64Test.php` for large and edge-case payloads.

## 0.3.0-alpha44 — 2026-10-10

- Database Import UI: permanent note that inline base64 images in K2 article HTML
  are removed automatically during import.

## 0.3.0-alpha43 — 2026-10-10

- k2_items import: strip inline data-URI base64 (and related img tags) before staging;
  raise per-row parse ceiling to 24 MB for items so articles like 197738 are imported
  without the embedded images.

## 0.3.0-alpha42 — 2026-10-10

- SQL capture ingest: strip leading whitespace/comments before parsing INSERT
  headers and correct values_offset. Fixes silent skip of statements >~512KB
  (k2_items / k2_tags_xref in large dumps) when capture files start with "\nINSERT".
- Harden mid-tuple ingest resume (trim scanned buffer; quote-safe skip-large compact).

## 0.3.0-alpha41 — 2026-10-10

- k2_tags_xref: accept NULL / omitted AUTO_INCREMENT ids and tagId/itemId aliases;
  synthesize stable BIGINT keys so xref rows are staged (was empty when dump omitted id).

## 0.3.0-alpha40 — 2026-10-10

- After Save: clear Joomla `_system` component-params cache and refresh
  ComponentHelper in-memory params (fixes path reverting to onlyusers on reload).
- Import Full/Incremental uses posted form config after save (not stale cache).
- Job UI shows dump basename + size; warn when dump is under 1 MB.

## 0.3.0-alpha39 — 2026-10-10

- SQL import: keep draining the in-memory buffer after the final fread (feof no
  longer drops the last chunk). Fixes empty staging for small dumps (e.g.
  onlyusers.sql) and missing trailing users/groups/viewlevels on large dumps.
- EOF flush now processes leftover SQL statement-by-statement instead of as one blob.

## 0.3.0-alpha38 — 2026-10-10

- Import Full/Incremental now submit adminForm so the SQL path and prefix on screen
  are saved before the job starts (fixes path appearing to "revert").
- Detect prefix also recognises users, usergroups, user_usergroup_map, viewlevels
  and menu (users-only dumps no longer fail with "No K2 tables").

## 0.3.0-alpha37 — 2026-10-10

- SQL import: stronger Plesk/mysqldump table-name parsing (`db`.`table`, `db.table`) and `_binary` values.
- Added scripts/diagnose-sql-allowlist.php to verify dump contains users/groups/viewlevels INSERTs.

## 0.3.0-alpha36 — 2026-10-10

- Database import records oversized skipped rows (id/title) in the job UI.
- Added scripts/list-oversized-k2-items.php to list huge k2_items from a dump offline.

## 0.3.0-alpha35 — 2026-10-10

- Database SQL import: recognise INSERT IGNORE / REPLACE INTO and `database`.`table`
  qualified names so users/usergroups/viewlevels are not skipped while menu/K2 import.

## 0.3.0-alpha34 — 2026-10-10

- Database staging expanded: mirror tables for users, usergroups, user_usergroup_map,
  viewlevels and menu (plus existing K2 four). Modules removed from migration scope.
- Import (full) / Clear reload all staging mirrors; Check/Preparation read users & menus
  from the new mirrors after re-import.

## 0.3.0-alpha33 — 2026-10-10

- Check → Users: preparation buckets (ready to create with source ID, ID conflicts,
  username/email blocks, destination only). Read-only — no create action yet.

## 0.3.0-alpha13 — 2026-10-10

- Database import: Cancel import stops a running job and keeps staging tables.
- Clear temporary tables also cancels stuck running jobs (Continue disappears).
- Auto-continue AJAX batches with dump MB progress bar; Pause / Resume auto.
- Job JSON uses UTF-8-safe encoding and base64 SQL buffer checkpoints.
- Preparation menu + Check ID conflict tables before Media.
- SQL dump path accepts `.sql`, `.zip`, and `.sql.zip`.

## 0.3.0-alpha3 — 2026-10-09

- Database: removed Source Joomla root path from this screen (stays on Media).
- Clarified source table prefix purpose and Import (full) wipe behaviour.

## 0.3.0-alpha2 — 2026-10-09

- Database hub language/completeness fix for incomplete source error.

## 0.3.0-alpha1 — 2026-10-09

- Added Database hub (under Media): K2 source A/B, four mirror staging tables, full/incremental import.
- Staging mirrors: k2_items, k2_categories, k2_tags, k2_tags_xref (SELECT-only from live K2 or SQL file).
- Source Connection menu redirects to Database; media copy prefers the new k2_items staging table.

## 0.2.4-alpha1 — 2026-10-09

- Compact Manual copy control (checkbox next to label).

## 0.2.3-alpha1 — 2026-10-09

- Media files: Manual copy under action buttons; Save settings per block; removed top toolbar Save/Close/Continue.
- Clearer source-folder hints based on Source connection root path.

## 0.2.2-alpha1 — 2026-10-09

- Renamed submenu to Media files.
- Cleaner Media UI: two stacked blocks (Site images / K2 Media) with manual-copy checkboxes that gray app controls.

## 0.2.1-alpha1 — 2026-10-09

- Reorganised Media into a hub: folder settings, path/size info, app vs manual modes.
- Media settings keep K2 sizes to copy only; intro/full mapping deferred to articles.
- Added resumable/incremental copy for site images and selected K2 cache sizes.
- Added ledger-based rollback for app-copied site images and k2migration tree rollback from Media.

## 0.2.0-alpha1 — 2026-10-09

- Added source mode selection: live SELECT-only database or absolute-path SQL dump.
- Added resumable SQL staging import into component staging tables.
- Added ID + uniqueness audit gate with READY / WARN / BLOCK statuses.
- Added Migration Plan, Media Settings and controlled Rollback admin pages.
- Added selectable K2 image sizes and intro/full mapping preferences.
- Added job / ledger / audit / staging schema for Phase 2 operations.

## 0.1.0-alpha1 — 2026-10-09

- Bootstrapped `pkg_devartk2migration` / `com_devartk2migration` for Joomla 6 / PHP 8.3.
- Added read-only source database connection configuration and connection test.
- Added source inventory, destination inventory and informational ID collision reporting.
- Documented media copy / incremental batch architecture for later phases.
- Added English and Greek administrator language files.
- Added Phase 1 helper tests via `scripts/test-phase1.php`.

Verification status belongs in `PROJECT_STATUS.md` and must record only
confirmed Joomla testing.
