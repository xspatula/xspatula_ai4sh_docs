---
title: "Setup AI4SH Database"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "Seeding the AI4SoilHealth postgreSQL database using the Xspatula framework. This guide covers running the setup notebook, editing the scheme file, and the full order of JSON process files."
permalink: /setup_db/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

Seeding the AI4SH database uses the same Xspatula JSON-driven workflow described in the [Xspatula python and DB model environment documentation][setup_core_db_docs]. What this site adds is the full set of AI4SH-specific process files that define the schemas, tables and processes for a comprehensive soil data repository.

## Prerequisites

Before running the AI4SH database setup you need:

- A running postgreSQL installation where you are superuser — see [PostgreSQL setup][postgres]
- The Xspatula framework installed and a working Anaconda environment — see [Anaconda setup][anaconda]
- The `load_ai4sh_db` repository cloned to your machine:

```bash
git clone https://github.com/xspatula/load_ai4sh_db
```

## The setup notebook

The database setup is driven by the Jupyter notebook:

```
./setup/setup_db.ipynb
```

Open this notebook in VS Code or Jupyter Lab. It has an imports cell you can run as-is, followed by three cells that matter:

1. **Scheme file** — set the path to your [scheme file][scheme_file]
2. **Setup database** — runs `Initiate_database`, pointed at the [job file][job_file] `job_setup_db.json`. This creates every schema and table, and — as a side effect — also (re)writes the audit config files described under [The pilot file](#the-pilot-file) below. No audit trigger exists in the database yet after this cell alone.
3. **Apply audit triggers** *(optional)* — runs `Initiate_audit`, pointed at `job_setup_audit.json`. This is what actually creates the audit log table and every table's trigger. Skip it and the database simply has no auditing; run it any time later (or re-run it after adding a new table's `"audit"` key) — it always re-applies the complete current config. See [Auditing setup][auditing_setup] for the full walkthrough.

## Edit the scheme file

The scheme file for setting up the AI4SH database is at:

```
./setup/zzz/scheme_ai4sh_local_setup.json
```

You must edit this file before running the notebook. At minimum change the postgreSQL superuser credentials and the database name:

```json
{
  "project_path": "./ai4sh",
  "postgresdb": {
    "host": "localhost",
    "port": 5432,
    "db": "ai4sh",
    "user_name": "your_postgres_superuser",
    "password": "your_postgres_superuser_password",
    "db_users": [
      {
        "user_id": "community_admin",
        "password": "guessing-rubble-garden-opera",
        "role": "community_admin"
      },
      {
        "user_id": "login_evaluation",
        "password": "hippodrome-bicycle-concert-shuttle",
        "role": "login_evaluation"
      },
      {
        "user_id": "user_cat_0",
        "password": "tablecloth-summerleaf-riverbasin-vacuumcleaner",
        "role": "user_cat_0"
      },
      {
        "user_id": "user_cat_1",
        "password": "secret-parsimony-archipelago-hedgehog",
        "role": "user_cat_1"
      },
      {
        "user_id": "user_cat_2",
        "password": "sailing-courageous-upsidedown-castle",
        "role": "user_cat_2"
      },
      {
        "user_id": "user_cat_3",
        "password": "rollerscates-forever-skyline-coconut",
        "role": "user_cat_3"
      },
      {
        "user_id": "user_cat_4",
        "password": "superfluid-altruistic-guitarplayer-climatechange",
        "role": "user_cat_4"
      },
      {
        "user_id": "user_cat_5",
        "password": "fireplace-olympicgames-grassroot-luminescence",
        "role": "user_cat_5"
      }
    ]
  },
  "process": [
    {
      "execute": true,
      "verbose": 1,
      "overwrite": false,
      "delete": false
    }
  ]
}
```

You can alternatively use a `.netrc` file for credentials — replace `user_name` and `password` with `"host_netrc_id": "your_netrc_machine_code"`. See [.netrc setup][netrc] in the core documentation.

Change the default passwords for all `db_users` before deploying to any non-local environment.

## The pilot file

There are two pilot files, used by two different cells above:

| Pilot file | Used by | Maintained |
|---|---|---|
| `db_xspatula_ai4sh_setup.txt` | "Setup database" cell (`job_setup_db.json`) | Hand-maintained |
| `db_xspatula_ai4sh_audit.txt` | "Apply audit triggers" cell (`job_setup_audit.json`) | Auto-generated — its own header says "do not hand-edit" |

### The core database pilot file

```
./setup/zzz/ai4sh/setup_db/db_xspatula_ai4sh_setup.txt
```

This text file lists all process JSON files in the order they must be executed. The order matters because schemas must exist before tables, and reference tables before tables that reference them.

The full execution order is:

1. `schema/schema_v10_sql.json` — create all 8 schemas
2. `utility/utility_v10_sql.json` — utility tables
3. `utility/territory_v10_sql.json` — territory reference data
4. `community/` — user categories, organisations and users
5. `process/` — process and process parameter tables
6. `observation_utility/` — 38 JSON files defining reference catalogue tables (independent first, then dependent), plus 9 eDNA metabarcoding catalogue files
7. `observation/` — dataset, campaign, sample and observation tables, including eDNA observation tables
8. `landscape/` — landscape utility and observation tables

Each section is described in detail in the following pages.

### The audit pilot file — generated, not hand-edited

```
./setup/zzz/ai4sh/setup_db/db_xspatula_ai4sh_audit.txt
```

You never write this file yourself. Every table's own `create_table` definition may carry an
inline `"audit": {"INSERT": bool, "UPDATE": bool, "DELETE": bool}` key — running the "Setup
database" cell above scans every table's key across the whole core pilot list and (re)writes
this pilot file, plus one `audit_triggers_<schema>_v10_sql.json` per schema, to match. This is
pure file assembly — no database connection is used and no audit objects exist yet at this
point.

Running the "Apply audit triggers" cell then executes this generated pilot file against the
database, creating the actual triggers. See [Auditing setup][auditing_setup] for the full
mechanism, including how to add auditing to a new table.

## Default community records

Before running the notebook, also edit the default organisation and user records:

```
./setup/zzz/ai4sh/setup_db/json_ai4sh/community/organisation_records_v10_sql.json
./setup/zzz/ai4sh/setup_db/json_ai4sh/community/user_records_v10_sql.json
```

These files insert at least one default organisation and user into the database. The inserted user name and password must match the `user_project` credentials in subsequent (non-setup) scheme files.

You must also edit the records for inserting the initial processes -- used for defining all other processes:

```
./setup/zzz/ai4sh/setup_db/json_ai4sh/process/processes_records_v10_sql.json
```

You must change the name of the `creator` to the name of a user you just defined in all records to be inserted. If the user is not found in the database when running this command, the script will return an error.

## Run the notebook

With the scheme file edited and the notebook pointing to it, run all cells in `setup_db.ipynb`. The framework will connect to your postgreSQL cluster, create the database, and execute all process files in pilot file order.

To delete and rebuild the database, use:

```
./setup/delete_db.ipynb
```

with a scheme file where `"delete": true`.


[setup_core_db_docs]: https://xspatula.github.io/setup_core_db_docs/
[postgres]: https://xspatula.github.io/setup_core_db_docs/setup_db/postgres/
[anaconda]: https://xspatula.github.io/setup_core_db_docs/setup_db/anaconda/
[netrc]: https://xspatula.github.io/setup_core_db_docs/setup_db/netrc/
[scheme_file]:https://xspatula.github.io/setup_core_db_docs/framework/scheme_file/
[job_file]:https://xspatula.github.io/setup_core_db_docs/framework/job_file//
[auditing_setup]: /auditing/setup/
