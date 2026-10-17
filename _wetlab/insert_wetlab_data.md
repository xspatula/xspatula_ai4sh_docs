---
title: "Insert Wetlab Data (Single Step)"
layout: single
sidebar:
  nav: "wetlab"
  nav2: "loading_data"
excerpt: "Cell 3.2.1 of insert_ai4sh_data.ipynb inserts the Agrolab observation log and observation values in one cell — translate and insert happen together per table, no separate manage step."
permalink: /wetlab/insert_wetlab_data/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-10-16 08:00:00 +0200
---

Cell **3.2.1 Load Agrolab (wetlab) observation data** of `./ai4sh/import_data/insert_ai4sh_data.ipynb`
(see [Load all data][all_data]) is the single-step alternative to [Translate Agrolab wetlab data][translate_ai4sh_wetlab_data]
plus its two manage pages. One process file holds two `insert_tabular_data` entries, observation
log first, then observations — see [Insert process][setup_process_insert] for the mechanism.

## Which route to use

Use cell 3.2.1 when you just want the two spreadsheets' rows in the database and don't need to
`UPDATE` an existing record or hand-inspect the generated JSON. Use [Translate Agrolab wetlab data][translate_ai4sh_wetlab_data]
plus the manage pages (`more_notebooks/load_ai4sh_wetlab_data.ipynb`) when you do. See
[Single-step vs dual-step][insert_vs_translate] for the full comparison.

## Prerequisites

Same as the 2-step route: [Load sample data] (cell 3.1) must be complete, and the observation
log's `provision` record (`ai4sh-agrolab`) must exist from [Load utility data] (block 1).

## Notebook cell

In `insert_ai4sh_data.ipynb`, block **3.2**, cell **3.2.1** runs:

```python
process_file = 'import_data/wetlab/agrolab/insert_process/insert_agrolab_observation_log+observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `./ai4sh/import_data/wetlab/agrolab/insert_process/insert_agrolab_observation_log+observation.json`

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
    },
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation",
        "tabular_data_path": "import_data/wetlab/agrolab/excel/AI4SH_observation_agrolab.xlsx",
        "dst_path": "import_data/wetlab/agrolab/insert_process/staging"
      }
    }
  ]
}
```

The order matters — the observation rows reference the observation log by name. Both source
files, columns, and the `@`-prefix observation-value convention are exactly as described on
[Translate Agrolab wetlab data][translate_ai4sh_wetlab_data] — only the loading mechanism
changes, not the source data format.

The same folder also holds the two halves as separate files, `insert_agrolab_observation_log.json`
and `insert_agrolab_observation.json`, used by `more_notebooks/insert_ai4sh_wetlab_data.ipynb`.

There is deliberately no `insert_process/manage_process` step for this route; the
`insert_process/staging/*.json` files it produces are a debug trail, not something you're meant
to edit.

## Other cells in block 3.2

Block 3.2 also holds the other non-spectral AI4SH observations. They follow the same
one-cell pattern (one `..._observation_log+observation.json` process file each) but are not yet
documented on this site:

- 3.2.2–3.2.5 Veltia, iBEC, SLU and Neiker wetlab data — not yet completed in the notebook
- 3.2.6 Digit Soil enzymatic activity
- 3.2.7 BEST infiltration
- 3.2.8 Microbiometer
- 3.2.9 Aggregate stability

[translate_ai4sh_wetlab_data]: {{ site.baseurl }}/wetlab/translate_ai4sh_wetlab_data/
[all_data]: {{ site.baseurl }}/all_data/
[Load sample data]: {{ site.baseurl }}/sample/
[Load utility data]: {{ site.baseurl }}/utility/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
