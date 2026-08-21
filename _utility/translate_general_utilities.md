---
title: "Translate General Utilities"
layout: single
sidebar:
  nav: "utility"
  nav2: "loading_data"
excerpt: "Translating the territory Excel file to a JSON process file using the load_ai4sh_utility_data notebook. This is the first of six utility loading steps."
permalink: /utility/translate_general_utilities/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

The first utility step translates the general utility Excel file (`territory.xlsx`) into a JSON process file. General utilities contain reference data used across the whole framework — for AI4SH this is a single table of ISO-coded countries and regions.

## Notebook cell

In `load_ai4sh_utility_data.ipynb`, the **Translate general utilities** cell runs:

```python
job_file = 'import_data/utility/job_translate_general_utility.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/utility/job_translate_general_utility.json`

```json
{
  "process": {
    "job_folder": "import_data/utility/general",
    "process_sub_folder": "process",
    "pilot_file": "translate_general_utility.txt"
  }
}
```

The job file points to the pilot file that lists which translate process files to run.

## Pilot file

**Path**: `./ai4sh/import_data/utility/general/translate_general_utility.txt`

```
territory.json
```

Only one process file is listed. Comments (lines beginning with `#`) are ignored by the framework.

## Translate process file

**Path**: `./ai4sh/import_data/utility/general/process/territory.json`

```json
{
  "process": [
    {
      "process": "translate_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_territory",
        "tabular_data_path": "import_data/utility/general/excel/territory.xlsx",
        "dst_path": "import_data/utility/general/manage_process"
      }
    }
  ]
}
```

`tabular_data_path` and `dst_path` are resolved relative to the project root, not to this process file's own directory — see [Path resolution][setup_process_path_resolution]. The translate process reads `territory.xlsx` and writes the resulting manage process file to:

```
./ai4sh/import_data/utility/general/manage_process/manage_territory.json
```

## Source file

**Path**: `./ai4sh/import_data/utility/general/excel/territory.xlsx`

The Excel file contains one row per territory. The framework maps each column header directly to a database column name in `utility.territory` (only possible for tables that have no foreign keys). Required columns:

| Column | Description |
|---|---|
| `name` | Lowercase identifier (e.g. `sweden`) |
| `display_name` | Display label (e.g. `Sweden`) |
| `iso_code_a2` | ISO 3166-1 alpha-2 code (e.g. `se`) |
| `iso_code_a2_ext` | Extended alpha-2 code (same as `iso_code_a2` in most cases) |

## Next step

After running this cell, proceed to [Manage general utilities] to insert the territory data into the database.

[Manage general utilities]: /utility/manage_general_utilities/
[setup_process_path_resolution]: /setup_process/path_resolution/
