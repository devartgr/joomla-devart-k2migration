# DevArt K2 Migration

**Migrate a Joomla 3 + K2 site to native Joomla 6 articles.** Move K2 items,
categories, tags, users, menus and images into a clean Joomla 6 site, from the
browser, with resumable batches, incremental sync and rollback.

Proven on a real site with **230,000+ articles and 90 GB of images**, run
entirely from the web interface.

![Joomla](https://img.shields.io/badge/Joomla-6.x-blue)
![PHP](https://img.shields.io/badge/PHP-8.3%2B-green)
![Release](https://img.shields.io/badge/Version-1.0.0-orange)
![License](https://img.shields.io/badge/License-GPLv3-red)

> ## ⚠️ Warning: read before you run
>
> A migration is a risky operation. This tool writes a large amount of data to
> your Joomla 6 database and `images/` folder.
>
> - Migrate into a **clean, disposable Joomla 6 destination**, never into a
>   live production site.
> - Take a **full backup** of the destination and keep one of the legacy site.
>   Test the restore.
> - Connect to the legacy database with a user that has **SELECT rights only**.
> - Rehearse on a copy first and verify the result yourself.
>
> This software is provided **as is, without warranty**, and you use it at your
> own risk. See [DISCLAIMER.md](DISCLAIMER.md).

## Why this exists

K2 is no longer maintained for modern Joomla, and many sites are stuck on
Joomla 3 because they have thousands of K2 articles and no practical way out.
This tool was written to move such a site to Joomla 6 and to keep working while
the new site is built: it re-imports only the new articles of the live K2 site
on demand.

## What it migrates

- **K2 items → Joomla articles**, with intro/full text, dates, featured flag,
  state, authors and workflow association
- **K2 categories → Joomla categories**
- **K2 tags → Joomla tags**, with tag maps on the articles
- **Users and user groups**, plus view levels
- **Menu items**
- **Site `images/` folder**, copied incrementally
- **K2 article images** (cache sizes you choose, `XL` by default), copied to
  `images/k2migration/YYYY/MM/` and keeping the original K2 file names
- **Original IDs** of articles, categories, tags, users, groups, view levels
  and menu items are preserved. ID collisions are reported for you to resolve,
  never rewritten silently

## Main features

- **Works from the browser.** Small time-boxed batches that survive page
  closes and Cloudflare timeouts. A CLI is available but optional.
- **Resumable and incremental.** Stop and continue any phase. Daily
  incremental imports bring only new articles and their images while the old
  site stays live.
- **Read-only source.** The legacy database is only read, and legacy files are
  never changed.
- **Fast on large sites.** Bulk article creation and an asset/nested-set
  rebuild that is resumable.
- **Preparation stage.** Detects ID, alias and user conflicts before anything is
  converted, and lets you remap or fix them.
- **Pilot first.** Convert a sample, evaluate it, and promote it before the full
  run.
- **Check and rollback.** Verify assets, counts, media and menus, and undo the
  recorded work of a Convert or a media copy.
- **Languages.** English and Greek administrator interface.

## Requirements

- Joomla **6** and PHP **8.3** or later on the destination
- MySQL / MariaDB
- Read access to the legacy K2 database, either a live connection or an SQL
  dump, and to its files (`images/` and `media/k2/items/cache/`)
- A destination with enough disk space for all copied images

There is no compatibility layer for Joomla 3, 4, 5 or PHP below 8.3 on the
destination.

## Installation

1. Download `pkg_devartk2migration_vX.Y.Z.zip` from the
   [Releases](../../releases) page.
2. In the Joomla 6 administrator go to **System → Install → Extensions** and
   upload the ZIP, or run
   `php cli/joomla.php extension:install --path=/path/to/package.zip`.
3. Open **Components → DevArt K2 Migration**.

## How it works

The administrator flow is:

1. **Database.** Connect to the K2 database (or load an SQL dump) and import
   into staging tables.
2. **Media.** Copy the site `images/` folder and the K2 article images.
3. **Preparation.** Review and resolve users, categories, tags, items and
   menus.
4. **Convert.** Run the pilot sample, then tags, items and menus.
5. **Check.** Verify the result before going live.
6. **Rollback.** Undo what the tool recorded, if you must.

Keep the destination site **offline** during Convert and until the asset
rebuild is finished. For the exact steps and the optional CLI, read
[docs/quick-start.md](docs/quick-start.md) and
[docs/ops-guide.md](docs/ops-guide.md).

### Staying in sync while you build the new site

After the first full import, run **Incremental** import and **Media
incremental** whenever you like. They bring only new K2 items, their tag links
and their images. Edits to articles that were already imported are not
propagated by the incremental import. Use a full import and the **Check**
editorial sync for that.

## What it does not do

By design, these are out of scope:

- K2 comments, attachments, galleries, video and extra fields
- Modules and templates
- Smart Search / Finder indexing (rebuild it yourself)
- Native Joomla 3 `com_content` articles (this tool is for K2 items only)
- Automatic SEO redirects. A helper generates `.htaccess` rules for
  `/item/ID-alias` URLs, but you must plan and test your own redirects

## Documentation

- [Quick start](docs/quick-start.md)
- [Operations guide](docs/ops-guide.md)
- [Architecture](docs/architecture.md)
- [Convert design](docs/convert-engine-design.md)
- [Media and batch architecture](docs/media-and-batch-architecture.md)
- [Disclaimer](DISCLAIMER.md)

## Support

This is a free tool, written mainly for the author's own migrations and shared
with others. There is no guaranteed support. Issues are welcome. Please include
the package version, the job stats and the error message.

## License

GNU General Public License version 3 or later.

Copyright (C) 2026 [Kostas Stathopoulos - DevArt](https://devart.gr)

*Joomla! is a registered trademark of Open Source Matters, Inc. This project is
independent and is not affiliated with Joomla or the K2 project.*

<!-- Keywords: K2 to Joomla 6 migration, migrate K2 to Joomla 4, 5, 6, Joomla 3 to Joomla 6, convert K2 items to Joomla articles, K2 migration tool -->
