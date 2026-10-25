# Disclaimer

**DevArt K2 Migration** (`pkg_devartk2migration`) is a free tool written primarily
for the author's own migrations and shared with others who are stuck on
Joomla 3 and K2. It is provided as is.

## Read this before you run it

A site migration is a risky operation. This tool writes a large amount of data
into your Joomla 6 database and into its `images/` folder, and some of those
changes are hard to undo. Use it only if you understand what it does and you are
prepared to restore from your own backup.

## No warranty

This software is released under the GNU General Public License version 3 or
later. It comes with **no warranty of any kind**, express or implied, including
fitness for a particular purpose, merchantability, completeness or accuracy of
the migrated data. The author is not liable for data loss, downtime, broken
URLs, lost search rankings, corrupted databases or any other damage arising from
its use. You use it entirely at your own risk.

## What the tool changes

- It **reads** the legacy Joomla 3 / K2 database. Source access is intended to
  be SELECT-only. Use a database user that has **only SELECT rights**, so the
  guarantee does not depend on this tool alone.
- It **writes** to the Joomla 6 destination: articles, categories, tags, users,
  menu items, workflow and tag-map rows, ID mapping and ledger tables.
- It **copies files** into the destination `images/` folder, including
  `images/k2migration/`. It does not modify the source files.
- It preserves source IDs for articles, categories, tags, users, groups,
  viewlevels and menu items. If an ID is already taken on the destination, the
  tool stops and asks you to resolve it. It does not rewrite it silently.

## Before you start

1. Migrate into a **clean Joomla 6 destination**, ideally a disposable clone or
   a fresh install. Do not migrate into a live production site.
2. Take a **full backup** (database and files) of the destination, and keep a
   backup of the legacy site. Test that you can restore it.
3. Keep the destination site **offline** during Convert and until the nested-set
   rebuild (asset densify) has finished.
4. Rehearse on a copy first. Check your own data, not only the tool's report.
5. Do not run it against the legacy production site as a write target.

## Limits of Undo, rollback and checks

The tool records its work and offers rollback and Check reports. These are
safety aids, not guarantees. Rollback removes what the tool recorded; it cannot
restore data you changed by hand, other extensions' data, or anything that was
not recorded. Check reports compare what the tool can see and may miss problems.
Always verify the result yourself.

## What is not migrated

By design, the following are out of scope and are not migrated:

- K2 comments, attachments, galleries and video
- K2 extra fields and extra field groups
- Modules and templates
- Smart Search / Finder indexes (rebuild them yourself)
- Native Joomla `com_content` articles (this tool migrates K2 items only)
- SEO redirects beyond the optional `.htaccess` helper for `/item/ID-alias`
  URLs. Plan your own redirects, and expect URLs and rankings to change.

Editing an article on the legacy site after it has been imported is not carried
over by the daily incremental import, which brings only new items. Use a full
import and the Check editorial sync if you need to propagate edits and deletions.

## Compatibility

Targets Joomla 6 and PHP 8.3 or later only. There is no compatibility layer for
older Joomla or PHP versions. It has been tested on one large real-world site
(about 230,000 articles and 90 GB of images). That does not mean it will work
on yours. Third-party extensions, plugins, custom K2 fields and unusual data can
behave differently.

## Responsibility and privacy

You are responsible for having the right to copy the content and user data you
migrate, and for complying with data-protection law (for example GDPR) that
applies to it. The tool copies user accounts and password hashes as they exist
in the source. The source database password is stored encrypted in the
component's parameters; remove it when you finish.

## Trademarks

Joomla! is a registered trademark of Open Source Matters, Inc. K2 is a product
of its respective authors. This project is independent and is not affiliated
with, endorsed by or sponsored by Joomla, Open Source Matters or the K2 project.

## Support

There is no guaranteed support. Issues and pull requests are welcome, but the
author may not respond. Reports with the package version, the job stats and the
error message are the most useful.

Copyright (C) 2026 Kostas Stathopoulos - DevArt, https://devart.gr
