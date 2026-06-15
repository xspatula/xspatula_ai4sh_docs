---
title: "Translate Wetlab Data"
layout: single
sidebar:
  nav: "wetlab"
  nav2: "loading_data"
excerpt: "Translating both Agrolab Excel files to JSON process files in a single process_file call. Covers the observation log structure, the provision link, and the '@'-prefixed value column format."
permalink: /wetlab/translate_ai4sh_wetlab_data/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Both Agrolab Excel files are translated in a single notebook cell. Like the [sample translate step], this cell calls a `process_file` directly — the translate process file contains two entries, one per table.

## Prerequisites

[Load sample data] must be complete. The observation log links to a `provision` record (`ai4sh-agrolab`) that must exist from [Load utility data].

## Notebook cell

In `load_ai4sh_wetlab_data.ipynb`, the **Translate Agrolab (wetlab) Observation log** cell runs:

```python
process_file = 'import_data/wetlab/agrolab/process/translate_agrolab_observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

Note: the cell uses `process_file`, not `job_file`.

## Translate process file

**Path**: `./ai4sh/import_data/wetlab/agrolab/process/translate_agrolab_observation.json`

```json
{
  "process": [
    {
      "process": "translate_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation_log",
        "tabular_data_path": "../../../wetlab/agrolab/excel/AI4SH_observation_log_agrolab.xlsx",
        "dst_path": "../../../wetlab/agrolab/manage_process"
      }
    },
    {
      "process": "translate_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation",
        "tabular_data_path": "../../../wetlab/agrolab/excel/AI4SH_observation_agrolab.xlsx",
        "dst_path": "../../../wetlab/agrolab/manage_process"
      }
    }
  ]
}
```

The two entries are executed in sequence. Each reads an Excel source file and writes one manage process file to `./ai4sh/import_data/wetlab/agrolab/manage_process/`.

## Source files

**`AI4SH_observation_log_agrolab.xlsx`** — one row per sampling log / provision combination. Required columns:

| Column | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: parent sampling log |
| `provision_id__provision_name` | FK lookup: the Agrolab provision record (e.g. `ai4sh-agrolab`) |
| `contact_name` | Name of the responsible person |
| `contact_email` | Email of the responsible person |
| `name` | Observation log name — `auto` generates `<sampling_log>@<provision>` |
| `preparation_id__preparation_name` | FK lookup: sample preparation method |
| `preservation_id__preservation_name` | FK lookup: sample preservation method |
| `storage_id__storage_name` | FK lookup: sample storage condition |
| `transportation_id__transportation_name` | FK lookup: transportation method |
| `laboratory` | Boolean — `true` for laboratory-based observations |

**`AI4SH_observation_agrolab.xlsx`** — one row per sample. Columns beyond the identifier fields carry the measured values and use the `@` prefix convention described below.

Required identifier columns:

| Column | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: parent observation log |
| `sample_id__sample_name` | FK lookup: sample record |
| `observation_date` | Date of analysis as integer (`YYYYMMDD`) |

## The `@` prefix for observation values

Columns whose names begin with `@` in the Excel file are observation value columns. The `@` is a signal to the framework that the column maps to a `provision_indicator` in the database — not a regular field or FK.

For example, `@ph-h2o` in the spreadsheet maps to the `ph-h2o` provision indicator defined in the `ai4sh-agrolab` provision. The value is stored as an observation record for that indicator.

Values can be:
- **Numeric**: stored directly (e.g. `5.9`, `32`, `2.18`)
- **String with qualifier**: below-detection values such as `"<0.10"` are stored as strings

## Output

Two manage process files are written:

| File | Source |
|---|---|
| `manage_observation_log.json` | `AI4SH_observation_log_agrolab.xlsx` |
| `manage_observation.json` | `AI4SH_observation_agrolab.xlsx` |

## Next step

Proceed to [Manage Agrolab observation log].

[sample translate step]: /sample/translate_geotag_sample/
[Load sample data]: /sample/
[Load utility data]: /utility/
[Manage Agrolab observation log]: /wetlab/manage_ai4sh_agrolab_obs_log/
