---
title: "Setup AI4SH Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "The second stage of seeding the AI4SH database registers all operational processes into the process schema. This enables the framework to handle data insertion, update and deletion for every schema in the database."
permalink: /setup_process/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

After the database schemas and tables are created (see [Setup DB][setup_db]), the second stage registers all operational processes into the `process` schema. These process definitions tell the framework what operations are available, what parameters they accept, and what permissions apply.

Without this stage, the framework has a working database structure but no knowledge of what it can do with it. Process registration is what connects the database tables to callable Python functions.

## The setup processes notebook

The process setup is driven by the Jupyter notebook:

```
./setup/setup_processes.ipynb
```

This notebook uses a separate scheme file from the database setup:

```
./setup/zzz/scheme_ai4sh_local_use.json
```

This scheme file uses ordinary user credentials (not the postgreSQL superuser) to connect to the already-created database.

## The pilot file

The job file for process setup points to:

```
./setup/zzz/ai4sh/setup_processes/ai4sh_setup_processes.txt
```

This pilot file lists the process JSON files to execute, grouped by schema. The execution order is:

1. `root_process/` — define the process families
2. `translate/` — translate tabular data process
3. `utility/` — foreign key and territory management processes
4. `observation_utility/` — 27 processes for managing observation utility catalogues
5. `observation/` — processes for managing observation data
6. `select/` — processes for selecting spectral data
7. `plot/` — processes for plotting indicators and spectra
8. `machine_learning/` — 12 processes for spectral preprocessing, outlier detection, and regression
9. `landscape/` — landscape processes (files defined, not yet in pilot)
10. `edna/` — eDNA workflow processes (files defined, not yet in pilot)

## Process file structure

Every process definition JSON file follows the same structure. A process is registered using the `manage_process` action with a `parameters` block describing the operation, and a `nodes` block describing each parameter it accepts:

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "manage_table_data",
        "process": "manage_apparatus",
        "min_user_stratum": 4,
        "title": "Manage apparatus type",
        "label": "Insert, update or delete a generic apparatus type used for observing properties"
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "name",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "Generic name of the apparatus",
              "schema_table": {
                "schema": "observation_utility",
                "table": "apparatus",
                "write": true
              },
              "permission": {
                "update": false,
                "delete": false
              }
            }
          ]
        }
      ]
    }
  ]
}
```

Key fields:

| Field | Description |
|---|---|
| `root_process` | The process family this operation belongs to |
| `process` | Unique process identifier |
| `min_user_stratum` | Minimum user access level required to run this process |
| `title` | Short human-readable name |
| `label` | Longer description |
| `parameter` | Parameter name (matches the column name in the target table) |
| `parameter_type` | Data type: `text`, `integer`, `real`, `boolean` |
| `required` | Whether the parameter must be supplied |
| `schema_table` | The target database table this parameter writes to |
| `permission.update` | Whether the value can be changed after initial insertion |
| `permission.delete` | Whether the record can be deleted |


[setup_db]: /setup_db/
