---
title: "Translate Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Translate processes convert tabular data (CSV or Excel spreadsheets) into xSpatula JSON process files. Unlike data management processes, translate processes do not write to the database — they prepare data for subsequent process calls."
permalink: /setup_process/translate/
author_profile: false
date: 2026-04-07 08:00:00 +0200
last_modified_at: 2026-04-07 08:00:00 +0200
---

Translate processes belong to the `translate_data` root process family. They convert externally held tabular data — CSV files or Excel spreadsheets — into the JSON process file format that the xSpatula framework uses for all data management operations. The generated JSON files can then be executed through the normal notebook workflow to insert or update records in the database.

This is distinct from the `manage_table_data` root: translate processes write JSON files to disk, not records to the database.

## Process file

**File**: `./setup/zzz/ai4sh/setup_processes/json_ai4sh/translate/translate_tabular_data_v10_sql.json`

## Registered process

### translate_tabular_data

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "translate_data",
        "process": "translate_tabular_data",
        "min_user_stratum": 3,
        "title": "Translate tabular data from spreadsheet to json process",
        "label": "Translate tabular data recorded as csv or excel files to xSpatula json command process files."
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "process",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "The target process."
            },
            {
              "parameter": "tabular_data_path",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "Absolute or relative path of the spreadsheet source data"
            },
            {
              "parameter": "dst_path",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "Absolute or relative destination path for writing the json process file"
            }
          ]
        }
      ]
    }
  ]
}
```

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `process` | text | yes | The target process that the generated JSON will call |
| `tabular_data_path` | text | yes | Path to the source CSV or Excel file |
| `dst_path` | text | yes | Destination path for the generated JSON process file |

## How it works

When `translate_tabular_data` is called, the framework:

1. Reads the tabular source file (CSV or Excel) at `tabular_data_path`
2. Maps each row to the parameter structure of the named `process`
3. Writes a JSON file to `dst_path` containing one process call per row

The resulting JSON file follows the standard xSpatula process file format and can be executed directly via the `setup_processes.ipynb` notebook or added to the pilot file for batch execution.

## When to use

Use `translate_tabular_data` when you have a large number of records to insert and maintaining them as a spreadsheet is more practical than writing individual JSON process calls. Typical use cases include:

- Loading a curated unit library from a reference spreadsheet
- Bulk-inserting apparatus, provider, or method definitions from a catalogue maintained outside the framework
- Converting field data collected in spreadsheet form into observation process files

## Access level

Minimum user stratum: 3
