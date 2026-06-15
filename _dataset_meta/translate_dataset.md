---
title: "Translate Dataset"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Translating all five dataset metadata Excel files to JSON process files in a single notebook cell. Covers data_source, person, dataset, campaign, and sampling_log."
permalink: /dataset_meta/translate_dataset/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

All five dataset metadata tables are translated in a single notebook cell. The job file points to a pilot file that lists the five translate process files in dependency order.

## Prerequisite

[Load utility data] must be complete. The translate step itself does not touch the database, but the manage steps that follow require utility records (`territory`, `license`, `spatial_reference`, `provision`, etc.) to already exist.

## Notebook cell

In `load_ai4sh_dataset_meta.ipynb`, the **Translate dataset** cell runs:

```python
job_file = 'import_data/dataset/job_translate_dataset.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/dataset/job_translate_dataset.json`

```json
{
  "process": {
    "job_folder": "import_data/dataset",
    "process_sub_folder": "process",
    "pilot_file": "translate_dataset.txt"
  }
}
```

## Pilot file

**Path**: `./ai4sh/import_data/dataset/translate_dataset.txt`

```
data_source.json
person.json
dataset.json
campaign.json
sampling_log.json
```

The order follows the loading dependency: data source and person before dataset, dataset before campaign, campaign before sampling log.

## Translate process file structure

All five translate process files share the same structure. Example for `data_source.json`:

**Path**: `./ai4sh/import_data/dataset/process/data_source.json`

```json
{
  "process": [
    {
      "process": "translate_tabular_data",
      "overwrite": false,
      "parameters": {
        "process": "manage_data_source",
        "tabular_data_path": "../../dataset/excel/data_source.xlsx",
        "dst_path": "../../dataset/manage_process"
      }
    }
  ]
}
```

Each translate process reads its Excel source file and writes the resulting manage process file to `./ai4sh/import_data/dataset/manage_process/`.

## Output

Running this cell produces five manage process files:

| File | Source |
|---|---|
| `manage_data_source.json` | `data_source.xlsx` |
| `manage_person.json` | `person.xlsx` |
| `manage_dataset.json` | `dataset.xlsx` |
| `manage_campaign.json` | `campaign.xlsx` |
| `manage_sampling_log.json` | `sampling_log.xlsx` |

## Next step

Proceed to [Manage data source] to begin inserting records into the database.

[Load utility data]: /utility/
[Manage data source]: /dataset_meta/manage_data_source/
