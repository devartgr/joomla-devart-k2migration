# Disclaimer / Limitation of Liability

This software is provided **as is**, without warranty of any kind, express or
implied. DevArt and Kostas Stathopoulos shall not be held liable for any
damages, data loss, downtime, security issues, migration failure, or other
problems resulting from the use or misuse of this software.

Users are responsible for:

- Testing in their own environment before any production cutover
- Maintaining full backups of both the destination and the legacy site, and
  verifying that restores work
- Using a destination that is disposable or offline during Convert
- Connecting to the legacy database with **SELECT-only** privileges

Always rehearse on a copy first. Migration writes large amounts of data to the
Joomla 6 database and `images/` folder — treat it as a high-risk operation.

Copyright (C) 2026 Kostas Stathopoulos - DevArt  
https://devart.gr
