---
title: "Insert Process"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "insert_tabular_data collapses the translate-then-manage 2-step route into one notebook cell: it reads a spreadsheet and applies it to the database immediately, INSERT-only by construction."
permalink: /setup_process/insert/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-08-21 08:00:00 +0200
---

`insert_tabular_data` is a single-step alternative to the [translate][setup_process_translate] process: it reads a spreadsheet and inserts its rows into the database in one notebook cell, with no manual pilot-file hand-off. It's registered under the same `translate_data` root as `translate_tabular_data`, and shares the same 3 parameters — but where `translate_tabular_data` only ever writes a JSON file, `insert_tabular_data` writes that file and then immediately applies it to the database.

## Process file

**File**: `./setup/zzz/ai4sh/setup_processes/json_ai4sh/translate/insert_tabular_data_v10_sql.json`

## Registered process

### insert_tabular_data

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "translate_data",
        "process": "insert_tabular_data",
        "min_user_stratum": 3,
        "title": "Insert tabular data from spreadsheet directly into the database",
        "label": "Translate tabular data recorded as csv or excel files to xSpatula json command process files, and immediately apply them to the database in one step."
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
              "hint": "Absolute or relative destination path for writing the staging json process file"
            }
          ]
        }
      ]
    }
  ]
}
```

Registered right after `translate_tabular_data_v10_sql.json` in the pilot list, under a
`### IMPORT DATA ###` section header.

## Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `process` | text | yes | The target process that the row data is applied through (e.g. `manage_territory`) |
| `tabular_data_path` | text | yes | Path to the source CSV or Excel file — resolved relative to the project root, see [Path resolution][setup_process_path_resolution] |
| `dst_path` | text | yes | Destination path for the generated **staging** JSON file — resolved relative to the project root, see [Path resolution][setup_process_path_resolution] |

Identical shape to `translate_tabular_data`'s parameters — same names, same types, same resolution rules.

## How it works

When `insert_tabular_data` is called, the framework:

1. Reads the tabular source file (CSV or Excel) at `tabular_data_path` — identical to `translate_tabular_data`.
2. Writes a staging JSON file to `dst_path`, one process call per row — identical in shape to `translate_tabular_data`'s output, but this file is a **debug/audit trail, not something you're meant to hand-edit**. That's the key difference from the 2-step route: there, the generated JSON is the thing you're expected to inspect or fix by hand before it's applied; here, it's applied immediately, before you'd ever get the chance.
3. **Immediately** loads and applies that staging file to the database, row by row, calling the same underlying insert logic the 2-step route's `manage_<table>` processes use — no separate cell, no pilot-file hand-off.
4. Every row is generated with `overwrite: false, delete: false` — same as the 2-step route's generated JSON always is. But because this route applies the file immediately rather than waiting for a human to possibly hand-edit it, **INSERT-only is structural, not a checkbox you could accidentally flip**: if a row already exists (matched on its natural key, the same lookup the manage step uses), it's silently left alone — never updated, never duplicated. Re-running the same insert cell twice is safe.

## When to use

Use `insert_tabular_data` when you just want a spreadsheet's rows in the database and don't need to inspect or edit the generated JSON, and don't need to `UPDATE` an existing record via this route.

Use `translate_tabular_data` plus a separate manage step (the existing 2-step route, see [Translate][setup_process_translate]) when you need to:

- `UPDATE` existing records (`insert_tabular_data` can't — it's INSERT-only by construction)
- Load from a JSON source that has no spreadsheet at all
- Manually inspect or edit the generated JSON before it's applied — e.g. to fix a bad row without touching the source spreadsheet

## Access level

Minimum user stratum: 3

[setup_process_translate]: /setup_process/translate/
[setup_process_path_resolution]: /setup_process/path_resolution/
