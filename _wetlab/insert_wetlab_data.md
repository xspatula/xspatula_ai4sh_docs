---
title: "Insert Wetlab Data (Single Step)"
layout: single
sidebar:
  nav: "wetlab"
  nav2: "loading_data"
excerpt: "insert_ai4sh_wetlab_data.ipynb inserts the Agrolab observation log and observation values in 2 cells instead of 3 — translate and insert happen together per table, no separate manage step."
permalink: /wetlab/insert_wetlab_data/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-08-21 08:00:00 +0200
---

`insert_ai4sh_wetlab_data.ipynb` is the single-step alternative to [Translate Agrolab wetlab data][translate_ai4sh_wetlab_data]
plus its two manage pages. Unlike that page's combined `process_file` (one file, two entries),
this route uses two separate cells, one process file each — see [Insert process][setup_process_insert]
for the mechanism.

## Which route to use

Use this notebook when you just want the two spreadsheets' rows in the database and don't
need to `UPDATE` an existing record or hand-inspect the generated JSON. Use [Translate Agrolab wetlab data][translate_ai4sh_wetlab_data]
plus the manage pages when you do. See [Single-step vs dual-step][insert_vs_translate] for the
full comparison.

## Prerequisites

Same as the 2-step route: [Load sample data] must be complete, and the observation log's
`provision` record (`ai4sh-agrolab`) must exist from [Load utility data].

## Notebook cells

In `insert_ai4sh_wetlab_data.ipynb`, after the shared imports and scheme-file cells, two cells
each translate and insert one table:

```python
process_file = 'import_data/wetlab/agrolab/insert_process/insert_agrolab_observation_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

The second cell is identical in shape, pointing at `insert_agrolab_observation.json`. Run the
observation-log cell first — the observation cell's rows reference it by name.

## Process files

**Path**: `./ai4sh/import_data/wetlab/agrolab/insert_process/insert_agrolab_observation_log.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation_log",
        "tabular_data_path": "import_data/wetlab/agrolab/excel/AI4SH_observation_log_agrolab.xlsx",
        "dst_path": "import_data/wetlab/agrolab/insert_process/staging"
      }
    }
  ]
}
```

`insert_agrolab_observation.json` follows the same pattern, calling `manage_observation`
against `AI4SH_observation_agrolab.xlsx` instead. Both source files, columns, and the `@`-prefix
observation-value convention are exactly as described on [Translate Agrolab wetlab data][translate_ai4sh_wetlab_data] —
only the loading mechanism changes, not the source data format.

There is deliberately no `insert_process/manage_process` step for this route; the
`insert_process/staging/*.json` files it produces are a debug trail, not something you're meant
to edit.

## A note on the all-in-one notebook

The aggregate `insert_ai4sh_data.ipynb` (single-step counterpart to `load_ai4sh_data.ipynb`)
uses a third variant, `insert_agrolab_observation_log+observation.json`, that combines both
tables into one process file — functionally the same two `insert_tabular_data` calls as above,
just bundled for a one-cell run through the whole pipeline.

[translate_ai4sh_wetlab_data]: /wetlab/translate_ai4sh_wetlab_data/
[Load sample data]: /sample/
[Load utility data]: /utility/
[setup_process_insert]: /setup_process/insert/
[insert_vs_translate]: /insert_vs_translate/
