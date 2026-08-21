---
title: "Community Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The community schema manages all organisations and users in the AI4SH database. It is a default Xspatula framework schema. Every user logging into the system must be registered here."
permalink: /setup_db/community/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The `community` schema manages all organisations and users in the AI4SH database. It is one of the default Xspatula framework schemas. Every user logging into the system must have a record in `community.user`, and the login process checks that table for credentials and access level.

![Community schema](/assets/media/xspatula_db_community_schema.png)

## Two ways to add a user

There are two distinct routes into `community.user`, depending on whether the user already
exists to log in with:

- **Bootstrap, during database setup** — hand-editing `user_records_v10_sql.json` and running
  `setup_db.ipynb`. This is the *only* option for the very first user, since every other route
  requires an already-logged-in user to run. See "Default records" below and
  [Bootstrap user][setup_community_bootstrap_user] for the full walkthrough.
- **Excel/CSV intake, for every user after that** — filling in a spreadsheet, reviewed and run
  through the `manage_organisation`/`manage_user` processes. No plaintext password is ever
  handled this way — one is generated and emailed automatically. See
  [Setup community][setup_community_introduction] for the full collection.

## Process files

Six process files create and populate the community schema:

| File | Purpose |
|---|---|
| `community/user_categories_v10_sql.json` | Creates the `user_categories` table |
| `community/user_categories_records_v10_sql.json` | Inserts default user category records |
| `community/organisation_v10_sql.json` | Creates the `organisation` table |
| `community/organisation_records_v10_sql.json` | **Edit before running** — inserts default organisation(s) |
| `community/user_v10_sql.json` | Creates the `user`, `user_media` and `user_activity` tables |
| `community/user_records_v10_sql.json` | **Edit before running** — inserts default user(s) |

## Tables

### user_categories

Defines the access strata (0–5) that control which `.env` credentials a user logs in with. Stratum 0 has the least privilege, stratum 5 the most.

### organisation

Holds registered organisations. At least one organisation must exist before any user can be inserted, because `community.user` references `community.organisation`.

Key columns: `id`, `organisation_name`, `abbreviation`, `territory_id`, `url`, `email`, `status_code`.

### user

The central authentication table. At login, the framework checks `user_name`, `password`, and `stratum_code` against this table.

Key columns:

| Column | Type | Description |
|---|---|---|
| `id` | SERIAL | Primary key |
| `organisation_id` | INTEGER | FK to `community.organisation` |
| `user_name` | VARCHAR UNIQUE | Login name |
| `password` | VARCHAR | Hash crypted login password |
| `stratum_code` | SMALLINT | FK to `user_categories`; determines database privilege level |
| `first_name` | VARCHAR | — |
| `last_name` | VARCHAR | — |
| `email` | VARCHAR | — |
| `territory_id` | INTEGER | FK to `utility.territory` |
| `status_code` | INTEGER | Account status (default 10 = active) |

### user_media

Optional social/web links for a user. One row per user, linked by `user_id`.

### user_activity

Tracks cumulative session time and last activity date per user.

## Default records — edit before running

The files `organisation_records_v10_sql.json` and `user_records_v10_sql.json` insert default records directly into the database during setup. You must edit these files to include at least one real organisation and one real user before running the notebook.

The `password` value in `user_records_v10_sql.json` must be a **bcrypt hash**, not plaintext — `community.user.password` is never stored or compared as plaintext. Generate the hash with the standalone CLI `setup/hash_password.py` and paste its output into the JSON file; see [Bootstrap user][setup_community_bootstrap_user] for the full walkthrough, including how to apply a new hash to a live database without rebuilding. Without a correctly hashed password here, the bootstrap user cannot log in and no further users can be registered — the Excel-intake registration workflow itself requires an already-logged-in user.

The `user_name` in this file must match the `user_name` in the `user_project` credentials of any non-setup scheme file you later use to operate the database.

**Note**, the scheme file's `password`, however, stays **plaintext** — it's the password you type at login, verified against the stored hash. The two values are never the same string; the scheme file holds the plaintext that hashes to the value stored in `user_records_v10_sql.json` upon login.

For the full framework treatment of the community schema and its role in authentication, see the [core framework documentation][setup_core_db_docs_schemas].

[setup_core_db_docs_schemas]: https://xspatula.github.io/setup_core_db_docs/setup_db/schemas_tables/
[setup_community_bootstrap_user]: /setup_community/bootstrap_user/
[setup_community_introduction]: /setup_community/
