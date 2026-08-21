---
title: "Root Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Root processes are named families that group related processes together. The AI4SH database registers two root processes: manage_table_data for all data management operations, and translate_data for data format translation — with one deliberate, named exception that does write to the database."
permalink: /setup_process/root_process/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

Root processes are named families that group related processes. Every process in the `process` schema must belong to a root process. Root processes themselves are stored in `process.root_process` and registered before any other processes in the setup_processes workflow.

## Process file

**File**: `./setup/zzz/ai4sh/setup_processes/json_ai4sh/root_process/root_processes_v10_sql.json`

## Registered root processes

The AI4SH database registers two root processes:

### manage_table_data

```json
{
  "process": "add_root_process",
  "parameters": {
    "root_process": "manage_table_data",
    "title": "Manage table data",
    "label": "Root for processes for inserting, updating and deleting table data"
  }
}
```

All data management operations — inserting, updating, and deleting records in any schema — belong to this root. Every process registered in the subsequent community, utility, observation_utility, observation, and landscape process files will reference `manage_table_data` as its root (the eDNA `manage_metabarcoding_*` processes are part of `observation_utility` and do the same).

### translate_data

```json
{
  "process": "add_root_process",
  "parameters": {
    "root_process": "translate_data",
    "title": "Translate data",
    "label": "Process for translating data between different formats — note that translate data does not affect the database, it prepares data in different formats to be handled by other processes."
  }
}
```

**One deliberate exception**: [`insert_tabular_data`][setup_process_insert] is registered under this same `translate_data` root, purely for dispatch convenience — `Run_process` routes anything under `translate_data` to the same handler class (`Process_import_JSON`), and that class's own internal dispatch is keyed off the process **name's prefix** (`translate_`, `insert_`, `manage_`), not the root it's registered under. So despite the root's label above, `insert_tabular_data` does write to the database — it reads a spreadsheet and applies it immediately, INSERT-only. `translate_tabular_data` itself is unaffected by this and still never touches the database.

Translation processes convert data between formats (e.g. from a tabular spreadsheet to the JSON process file format expected by the framework). As a rule, translation processes do not write to the database directly — they prepare data for subsequent processes that do — with the one `insert_tabular_data` exception noted above.

## Why root processes matter

The root process is part of the primary key structure for process lookup. When the framework receives a process call it uses both the `root_process_id` and the `process_id` to find the correct Python function to execute. This two-level lookup means you can have the same process name in different root families without collision.

Root processes also serve as a grouping mechanism in the user interface — all processes under `manage_table_data` can be presented together as the "data management" operations for a given schema.

[setup_process_insert]: /setup_process/insert/
