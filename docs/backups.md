# Backups

> **Status: planned.** Automated backups are not set up yet. This page describes what the plan covers.

## What needs to be backed up

| Data                            | Why                                                                 |
| ------------------------------- | ------------------------------------------------------------------- |
| Nextcloud data folder           | User files                                                          |
| MariaDB database                | Nextcloud metadata; the files are not usable without it             |
| Vaultwarden data                | The encrypted password vault                                        |
| Docker compose and `.env` files | Needed to rebuild the stack (the `.env` files are kept outside Git) |

## Plan

- Automate the backups with `cron`, using a script in [`scripts/`](../scripts).
- Dump the database with a consistent snapshot rather than copying live database files.
- Keep at least one copy on a separate disk, and ideally one off-site.
- Test a restore, since a backup that was never restored is not a proven backup.

## Open decisions

- Where the backups will be stored (external drive, second machine, cloud).
- How long to keep them.
- Whether the backups are encrypted.
