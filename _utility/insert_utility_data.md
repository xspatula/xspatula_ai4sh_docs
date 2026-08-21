---
title: "Insert Utility Data (Single Step)"
layout: single
sidebar:
  nav: "utility"
  nav2: "loading_data"
excerpt: "insert_ai4sh_utility_data.ipynb loads all three utility groups in 3 cells instead of 6 — translate and insert happen together, with no manual pilot-file hand-off. INSERT-only, safe to re-run."
permalink: /utility/insert_utility_data/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-08-21 08:00:00 +0200
---

`insert_ai4sh_utility_data.ipynb` is the single-step alternative to the 6-cell
translate-then-manage walkthrough covered by [Translate general utilities][translate_general_utilities],
[Manage general utilities][manage_general_utilities], and the observation-utility page pairs.
Each of its 3 cells does in one step what previously took a translate cell and a manage cell —
see [Insert process][setup_process_insert] for the mechanism this relies on.

## Which route to use

Use this notebook when you just want a spreadsheet's rows in the database and don't need to
inspect the generated JSON or `UPDATE` an existing record. Use the existing translate/manage
route when you need to `UPDATE`, or want to hand-inspect/edit the generated JSON before it's
applied. See [Synopsis][utility_synopsis] for the fuller "which one should I use" guidance,
and [Single-step vs dual-step][insert_vs_translate] for the site-wide comparison.

## Notebook cells

In `insert_ai4sh_utility_data.ipynb`, after the shared imports and scheme-file cells, three
cells each translate and insert one utility group in a single call:

```python
job_file = 'import_data/utility/job_insert_general_utility.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

The other two cells are identical in shape, pointing at
`job_insert_observation_utility.json` and `job_insert_observation_utility_inherit.json`
respectively. Order matters — general utilities first, then observation utilities, then the
inheritance-dependent ones — same dependency order as the existing translate/manage route.

## Job files

Each `job_insert_*.json` mirrors the existing `job_translate_*.json` shape, but points at a
parallel `insert_process/` folder instead of `process/`:

```json
{
  "process": {
    "job_folder": "import_data/utility/general",
    "process_sub_folder": "insert_process",
    "pilot_file": "insert_general_utility.txt"
  }
}
```

The pilot files (`insert_general_utility.txt`, `insert_observation_utility.txt`,
`insert_observation_utility_inherit.txt`) list the same tables, in the same dependency order,
as the existing `translate_*.txt` pilot files — just re-headed for the insert route.

## Process files

Each table gets one process file under `insert_process/`, same shape as the existing
`process/<table>.json` translate files, but calling `insert_tabular_data` instead of
`translate_tabular_data`, and writing its staging output to `insert_process/staging` instead
of `manage_process`:

```json
{
  "process": [
    {
      "overwrite": true,
      "process": "insert_tabular_data",
      "parameters": {
        "process": "manage_territory",
        "tabular_data_path": "import_data/utility/general/excel/territory.xlsx",
        "dst_path": "import_data/utility/general/insert_process/staging"
      }
    }
  ]
}
```

The observation-utility group works the same way, e.g. `unit.json`:

```json
{
  "process": [
    {
      "overwrite": true,
      "process": "insert_tabular_data",
      "parameters": {
        "process": "manage_unit",
        "tabular_data_path": "import_data/utility/observation/excel/unit.xlsx",
        "dst_path": "import_data/utility/observation/insert_process/staging"
      }
    }
  ]
}
```

There is deliberately **no** `insert_process/manage_process` step and no separate "manage"
job/pilot files for this route — that whole hand-off is what got automated away. The
`insert_process/staging/*.json` files that appear after running these cells are a debug trail,
not something you're meant to edit.

[translate_general_utilities]: /utility/translate_general_utilities/
[manage_general_utilities]: /utility/manage_general_utilities/
[utility_synopsis]: /utility/
[setup_process_insert]: /setup_process/insert/
[insert_vs_translate]: /insert_vs_translate/
