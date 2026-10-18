---
title: "Load Dataset Metadata"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Cell 2.1 of insert_ai4sh_data.ipynb loads data source, persons, dataset, campaign, and sampling log in a single job — translate and insert happen together per table, with no separate manage step."
permalink: /dataset_meta/insert_dataset_meta/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-10-16 08:00:00 +0200
---

Block **2. Dataset metadata** of `./ai4sh/import_data/insert_ai4sh_data.ipynb` (see [Load all data][all_data])
is the single-step alternative to [Translate dataset][translate_dataset] plus its five manage
pages. One cell, **2.1 Load dataset metadata**, calls a `job_file` whose pilot file inserts the
five tables in dependency order — see [Insert process][setup_process_insert] for the mechanism.

## Which route to use

Use block 2 when you just want the five spreadsheets' rows in the database and don't need to
`UPDATE` an existing record or hand-inspect the generated JSON. Use [Translate dataset][translate_dataset]
plus the manage pages (`more_notebooks/load_ai4sh_dataset_meta.ipynb`) when you do. See
[Single-step vs dual-step][insert_vs_translate] for the full comparison.

## Notebook cell

Block 1 (utilities) must have run first. In `insert_ai4sh_data.ipynb`, cell **2.1 Load dataset
metadata** runs:

```python
job_file = 'import_data/dataset_metadata/job_insert_dataset_metadata.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/dataset_metadata/job_insert_dataset_metadata.json`

```json
{
  "process": {
    "job_folder": "import_data/dataset_metadata",
    "process_sub_folder": "insert_process",
    "pilot_file": "insert_dataset_metadata.txt"
  }
}
```

## Pilot file

**Path**: `./ai4sh/import_data/dataset_metadata/insert_dataset_metadata.txt`

```
data_source.json
person.json
dataset.json
campaign.json
sampling_log.json
```

The order is mandatory: data source and person before dataset, dataset before campaign, campaign
before sampling log. Comment lines (`#`) are left out above.

## Process files

Each table gets one process file under `./ai4sh/import_data/dataset_metadata/insert_process/`,
same shape as the `process/<table>.json` translate files, but calling `insert_tabular_data`
instead of `translate_tabular_data`, and writing its staging output to `insert_process/staging`
instead of `manage_process`. Example, `data_source.json`:

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_data_source",
        "tabular_data_path": "import_data/dataset_metadata/excel/data_source.xlsx",
        "dst_path": "import_data/dataset_metadata/insert_process/staging"
      }
    }
  ]
}
```

The other four tables follow the same pattern, one process file each. There is deliberately no
`insert_process/manage_process` step for this route — the `insert_process/staging/*.json` files
that appear after running the cell are a debug trail, not something you're meant to edit.

## Sampling logs belong to the metadata

In AI4SH the sampling logs are entered as part of the dataset metadata, not together with the
samples or observations. This keeps the sampling logs consistent across all observation types
(wetlab, spectra, eDNA etc.), without duplicates under different names. Cell 3.1 (samples) and
all later cells therefore require cell 2.1 to have run.

[translate_dataset]: {{ site.baseurl }}/dataset_meta/translate_dataset/
[all_data]: {{ site.baseurl }}/all_data/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
