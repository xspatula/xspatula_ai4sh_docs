---
title: "Insert Sample Data"
layout: single
sidebar:
  nav: "sample"
  nav2: "loading_data"
excerpt: "Cell 3.1 of insert_ai4sh_data.ipynb inserts geolocation and geolocated profile sample data in one combined process file, translate and insert together — no separate manage step."
permalink: /sample/insert_sample_data/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-10-16 08:00:00 +0200
---

Cell **3.1 Load AI4SH geolocations** of `./ai4sh/import_data/insert_ai4sh_data.ipynb` (see
[Load all data][all_data]) is the single-step alternative to [Translate geotag sample][translate_geotag_sample]
plus its two manage pages. See [Insert process][setup_process_insert] for the mechanism.

## Which route to use

Use cell 3.1 when you just want the two spreadsheets' rows in the database and don't need to
`UPDATE` an existing record or hand-inspect the generated JSON. Use [Translate geotag sample][translate_geotag_sample]
plus the manage pages (`more_notebooks/load_ai4sh_sample_data.ipynb`) when you do. See
[Single-step vs dual-step][insert_vs_translate] for the full comparison.

## Prerequisites

Sample geolocation depends on `sampling_log` already existing in the database. Sampling logs are
inserted with the dataset metadata in cell 2.1 (see [Insert dataset metadata][insert_dataset_meta]),
so blocks 1 and 2 must have run first.

Cell 3.1 must run before any other cell in block 3: observations are only accepted for samples
registered with an existing geolocation, which keeps the sample naming consistent across all
observation types.

## Notebook cell

One cell translates and inserts both sample tables, in one combined process file (same
`process_file`-not-`job_file` pattern the 2-step [Translate geotag sample][translate_geotag_sample]
page uses):

```python
process_file = 'import_data/sample/insert_process/insert_geotag_sample.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `./ai4sh/import_data/sample/insert_process/insert_geotag_sample.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_geolocation",
        "tabular_data_path": "import_data/sample/excel/AI4SH_geolocation.xlsx",
        "dst_path": "import_data/sample/insert_process/staging"
      }
    },
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_geolocated_profile_sample",
        "tabular_data_path": "import_data/sample/excel/AI4SH_samples_geotag.xlsx",
        "dst_path": "import_data/sample/insert_process/staging"
      }
    }
  ]
}
```

Same two entries, same order, as the 2-step route's `translate_geotag_sample.json` — geolocation
first, then geolocated profile sample. There is deliberately no `insert_process/manage_process`
step for this route; the `insert_process/staging/*.json` files it produces are a debug trail,
not something you're meant to edit.

[translate_geotag_sample]: {{ site.baseurl }}/sample/translate_geotag_sample/
[all_data]: {{ site.baseurl }}/all_data/
[insert_dataset_meta]: {{ site.baseurl }}/dataset_meta/insert_dataset_meta/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
