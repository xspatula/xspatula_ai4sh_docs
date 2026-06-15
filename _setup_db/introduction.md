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

Open this notebook in VS Code or Jupyter Lab. The notebook has two key code blocks to edit before running:

1. **Scheme file** — set the path to your [scheme file][scheme_file]
2. **Job file** — set the name of the [job file][job_file] (default: `job_setup_db.json`)

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

The job file points to the pilot file:

```
./setup/zzz/ai4sh/setup_db/db_xspatula_ai4sh_setup.txt
```

This text file lists all process JSON files in the order they must be executed. The order matters because schemas must exist before tables, and reference tables before tables that reference them.

The full execution order is:

1. `schema/schema_v10_sql.json` — create all 9 schemas
2. `utility/utility_v10_sql.json` — utility tables
3. `utility/territory_v10_sql.json` — territory reference data
4. `community/` — user categories, organisations and users
5. `process/` — process and process parameter tables
6. `observation_utility/` — 37 JSON files defining reference catalogue tables (independent first, then dependent)
7. `observation/` — dataset, campaign, sample and observation tables
8. `landscape/` — landscape utility and observation tables
9. `edna/` — eDNA utility and observation tables

Each section is described in detail in the following pages.

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
