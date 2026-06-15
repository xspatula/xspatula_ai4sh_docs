---
title: "Process Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The process schema stores all process definitions and their parameter specifications. It is a default Xspatula framework schema. Every operation the framework can execute must be registered here."
permalink: /setup_db/process/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The `process` schema stores all process definitions and their parameter specifications. It is one of the three default Xspatula framework schemas. Every operation the framework can execute — including the operations used to insert records into the `process` schema itself — must be registered here before it can be called from a process file.

## Process files — edit before running

Three process files create and seed the process schema:

| File | Purpose |
|---|---|
| `process/processes_v10_sql.json` | Creates the 8 process tables |
| `process/root_process_records_v10_sql.json` | Inserts the root process group definitions |
| `process/processes_records_v10_sql.json` | Inserts the bootstrap processes needed to register all other processes |

You must edit `processes_records_v10_sql.json` to have a `creator` that exists in the database; if not the script will report an error and stop. Typically, the user defined in the [Default community records][default_community_record]

## Tables

The process schema defines 8 tables that together fully describe every operation the framework can run:

| Table | Purpose |
|---|---|
| `root_process` | Groups related processes into families (e.g. `manage_table_data`) |
| `process` | Each executable operation, linked to a `root_process` and a Python function |
| `process_parameter` | All parameters expected by a specific process |
| `process_parameter_set_value` | Enumerated allowed values for a text parameter |
| `process_parameter_minmax` | Minimum and maximum allowed values for a numeric parameter |
| `process_parameter_schema_table` | The target `schema.table` that a parameter writes to |
| `process_parameter_permission` | Whether a parameter value can be updated or deleted after insertion |
| `process_parameter_default` | Default parameter values drawn automatically from another table |

## Bootstrap processes

`processes_records_v10_sql.json` inserts the minimal set of processes needed to register all other processes. Without these bootstrap records, no subsequent process file that tries to register a new process can execute — the framework would find no matching process definition for `add_process`.

As outlined above you must edit the `creator` for all process records to be inserted, but otherwise
these bootstrap records are fixed and should not be edited.

## AI4SH processes

After the community and process schemas are seeded, the [setup processes][setup_processes] stage registers all AI4SH-specific processes (for managing campaigns, samples, observations, etc.) into the process schema. This is done via the separate `setup_processes.ipynb` notebook.

For the full technical description of the process schema tables and how process files reference them, see the [core framework documentation][setup_core_db_docs_schemas].


[setup_processes]: /setup_process/
[setup_core_db_docs_schemas]: https://xspatula.github.io/setup_core_db_docs/setup_db/schemas_tables/
[default_community_record]: ../index.html#default-community-records
