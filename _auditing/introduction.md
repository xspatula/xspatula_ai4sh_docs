---
title: "Auditing"
layout: single
sidebar:
  nav: "auditing"
excerpt: "Every write to the AI4SH database is logged automatically by a native PostgreSQL trigger system — not by Python. This page explains the mechanism, what is audited, and how to change it."
permalink: /auditing/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

A natural first question: is auditing a Python thing, or a database thing? It's a **database thing**. Python touches it exactly once, at setup time, to create two Postgres objects. After that, Python is out of the picture entirely — every `INSERT`, `UPDATE`, or `DELETE` on an audited table is intercepted by Postgres itself, synchronously, inside the same transaction as the write that triggered it, no matter what client made that write: a notebook, `psql`, DBeaver, or any future application.

Think of it like a doorbell wired directly into a house's electrical system, not a security-camera app running on someone's phone. The app only has to be open once, to do the wiring — after that, the doorbell rings no matter who's home or what device is nearby, because the mechanism lives in the house itself.

If the audit insert itself fails for any reason, the whole triggering write rolls back with it — you can't accidentally end up with a change that wasn't logged.

## The two Postgres objects

1. **`audit.if_modified_func()`** — a single PL/pgSQL trigger function, shared by every audited table in the database. It reads the special variables Postgres hands to any trigger function (`TG_OP`, `TG_TABLE_SCHEMA`, `TG_TABLE_NAME`, `OLD`, `NEW`), serializes the row to JSONB, and inserts one row into `audit.logged_actions`. There is no per-table column list to maintain anywhere — the function is generic, so it can't drift out of sync when a table's schema changes.
2. **One `CREATE TRIGGER` per audited table** — named `<table>_audit`, firing `AFTER` the relevant events, `FOR EACH ROW`, calling `audit.if_modified_func()`.

## The audit.logged_actions table

Every logged change is one row here:

| Column | Meaning |
|---|---|
| `id` | Row identity |
| `schema_name`, `table_name` | Which table changed — always filter on both; table names repeat across schemas (e.g. `unit` exists in more than one) |
| `op` | `I` / `U` / `D` |
| `old_data`, `new_data` | Full row image as JSONB (`to_jsonb`) — `old_data` is `NULL` on an `INSERT`, `new_data` is `NULL` on a `DELETE` |
| `changed_by` | The connecting Postgres *role* (`current_user`) — shared per user stratum (e.g. `user_cat_2`), not a specific person |
| `changed_by_user_id` | The individual `community.user.id`, when known — see below |
| `changed_at` | Wall-clock time the trigger fired (`clock_timestamp()`) |
| `txid` | The Postgres transaction ID — group every row from the same transaction together |

### Individual attribution: changed_by vs. changed_by_user_id

`changed_by` only ever identifies the shared Postgres role a connection used — everyone logged in at the same access stratum connects as the same role, so `changed_by` alone can't tell you *which person* made a change.

To fix that, `PG_session` (in `src/postgres/pg_session.py`) accepts an `app_user_id` parameter — the `community.user.id` of the individual who just logged in — and sets it as a session-level Postgres setting (`audit.app_user_id`) right after authentication. `audit.if_modified_func()` reads that setting on every trigger firing and stores it as `changed_by_user_id`.

This column is `NULL` whenever a write happens outside a real login flow — most notably the raw-superuser `setup_db.ipynb`/`delete_db.ipynb` path. That's not a bug to explain away: a `NULL` here is a genuine, useful signal that a change came from infrastructure/admin setup rather than a specific person's action.

## Which tables are audited, and for which events

Coverage differs by **table role**, not just by schema:

| Schema | Audited tables | `INSERT` audited? |
|---|---|---|
| `community` | 5 | yes |
| `process` | 9 | yes |
| `utility` | 2 | yes |
| `observation_utility` | 63 | yes |
| `landscape_utility` | 17 | yes |
| `observation` | 40 | no (`UPDATE`/`DELETE` only) |
| `landscape` | 6 | no (`UPDATE`/`DELETE` only) |
| `audit` (self-audit) | 1 | no, and never can be — see below |

The rule in one sentence per group:

- **Full coverage** (`community`, `process`, `utility`, `observation_utility`, `landscape_utility`) — admin/config data, or catalogue/reference tables (`indicator`, `taxa`, `apparatus`, `unit`, `land_use_order`, etc.) that get added one row at a time, usually by hand. Knowing when a new catalogue entry was created has real audit value.
- **`UPDATE`/`DELETE` only** (`observation`, `landscape`) — bulk pipeline-written data: samples, measurements, spectra. Auditing every `INSERT` here would double the write volume on an import batch of thousands of rows, for no real audit gain — the row's own existence already proves it was created.

When adding a new table to the schema: a new catalogue/reference table follows the full-coverage group; a new bulk pipeline-written table follows the `UPDATE`/`DELETE`-only group.

## The self-audit gotcha

`audit.logged_actions` audits *itself* — but only for `UPDATE`/`DELETE`, **never** `INSERT`. This is deliberate, not an oversight: `if_modified_func()` responds to every trigger event by inserting a row into `audit.logged_actions`. An `INSERT` trigger on that same table would make that internal insert re-fire itself, recursing forever. This is the one table in the database that can never appear in its own log with `op = 'I'`.

## Read access

The `login_evaluation` role has read access to the audit log, alongside its existing read access to `community`:

```sql
GRANT USAGE ON SCHEMA audit TO login_evaluation;
GRANT SELECT ON ALL TABLES IN SCHEMA audit TO login_evaluation;
```

This is almost certainly the role you want to connect DBeaver as for the queries on the next page, unless you're already using a role with broader database access.

## Config file map

If you ever need to change what's audited, everything lives under `setup/zzz/ai4sh/setup_db/json_ai4sh/audit/`:

| File | Defines |
|---|---|
| `audit_table_v10_sql.json` | The `audit.logged_actions` table |
| `audit_function_v10_sql.json` | The `audit.if_modified_func()` trigger function |
| `audit_triggers_<schema-group>_v10_sql.json` | One `CREATE TRIGGER` block per audited table, grouped by schema |

**To audit a new table**: add one `create_trigger` block (`schema`, `table`, `trigger`, `timing`, `events`, `function: "audit.if_modified_func"`) to the relevant `audit_triggers_*` file — full-coverage events for a catalogue/admin table, `UPDATE`/`DELETE`-only for a bulk pipeline-written table — and make sure the pilot-list entry for that trigger file comes *after* both the target table and `audit_function_v10_sql.json` have already been created.

See [Auditing queries][auditing_queries] for how to actually read this log day-to-day, and [Setup DB][setup_db] for how the schemas and tables it audits are created in the first place.

[auditing_queries]: /auditing/queries/
[setup_db]: /setup_db/
