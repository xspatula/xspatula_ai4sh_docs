---
title: "Insert Sample Data (Single Step)"
layout: single
sidebar:
  nav: "sample"
  nav2: "loading_data"
excerpt: "insert_ai4sh_sample_data.ipynb inserts geolocation and geolocated profile sample data in one combined process file, translate and insert together — no separate manage step."
permalink: /sample/insert_sample_data/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-08-21 08:00:00 +0200
---

`insert_ai4sh_sample_data.ipynb` is the single-step alternative to [Translate geotag sample][translate_geotag_sample]
plus its two manage pages. See [Insert process][setup_process_insert] for the mechanism.

## Which route to use

Use this notebook when you just want the two spreadsheets' rows in the database and don't
need to `UPDATE` an existing record or hand-inspect the generated JSON. Use [Translate geotag sample][translate_geotag_sample]
plus the manage pages when you do. See [Single-step vs dual-step][insert_vs_translate] for the
full comparison.

## Preliminary cell: re-apply sampling logs

Sample geolocation depends on `sampling_log` already existing in the database (see
[Insert dataset metadata][insert_dataset_meta]). The first content cell in this notebook
re-applies `import_data/dataset/insert_process/sampling_log.json` directly, as a safety net in
case this notebook is run on its own:

```python
process_file = 'import_data/dataset/insert_process/sampling_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

This is the exact same file the dataset-metadata single-step route uses — re-running it here
is safe precisely because `insert_tabular_data` is INSERT-only: it never duplicates or
overwrites sampling logs that already exist.

## Insert sample geotag data

One cell translates and inserts both sample tables, in one combined process file (same
`process_file`-not-`job_file` pattern the existing 2-step [Translate geotag sample][translate_geotag_sample]
page already uses):

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

[translate_geotag_sample]: /sample/translate_geotag_sample/
[insert_dataset_meta]: /dataset_meta/insert_dataset_meta/
[setup_process_insert]: /setup_process/insert/
[insert_vs_translate]: /insert_vs_translate/
