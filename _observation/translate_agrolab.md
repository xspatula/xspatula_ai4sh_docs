---
title: "Translate Agrolab Data"
layout: single
sidebar:
  nav: "observation"
  nav2: "loading_data"
excerpt: "Translating both Agrolab Excel files to JSON process files in a single process_file call. Covers the observation log structure, the provision link, and the '@'-prefixed value column format."
permalink: /observation/translate_agrolab/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-10-19 08:00:00 +0200
---

Both Agrolab Excel files are translated in a single notebook cell. Like the [sample translate step], this cell calls a `process_file` directly — the translate process file contains two entries, one per table.

## Prerequisites

[Load sample data] must be complete. The observation log links to a `provision` record (`ai4sh-agrolab`) that must exist from [Load utility data].

## Notebook cell

In the 2-step notebook `./ai4sh/import_data/more_notebooks/load_ai4sh_wetlab_data.ipynb`, the **Translate Agrolab (wetlab) Observation log** cell runs:

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
        "tabular_data_path": "import_data/wetlab/agrolab/excel/AI4SH_observation_log_agrolab.xlsx",
        "dst_path": "import_data/wetlab/agrolab/manage_process"
      }
    },
    {
      "process": "translate_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation",
        "tabular_data_path": "import_data/wetlab/agrolab/excel/AI4SH_observation_agrolab.xlsx",
        "dst_path": "import_data/wetlab/agrolab/manage_process"
      }
    }
  ]
}
```

`tabular_data_path` and `dst_path` are resolved relative to the project root, not to this process file's own directory — see [Path resolution][setup_process_path_resolution]. The two entries are executed in sequence. Each reads an Excel source file and writes one manage process file to `./ai4sh/import_data/wetlab/agrolab/manage_process/`.

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

The text after `@` can be the indicator's name or alias. An `@` column that is not registered for the provision **stops the import** with `No indicator found for name/alias: <key>`.

| Value in the cell | Stored |
|---|---|
| number, or a numeric string (e.g. `5.9`, `32`, `"2.18"`) | as a number |
| below detection, starting with `<` (e.g. `"<0.10"`) | half the limit, `0.05` (a number, not a string) |
| other non-numeric string | nothing; warning `⚠️ Value for indicator … is not a number` |
| empty | nothing for that indicator |

The unit is not in the spreadsheet; it comes from the provision's `provision_indicator` row.

The texture columns (`@clay`, `@fine silt`, `@coarse silt`, `@fine sand`, `@coarse sand`) also give derived USDA and IUSS texture classes, stored in `observation.observation_classification` — see [Derived soil texture classes][obs_texture].

## Output

Two manage process files are written:

| File | Source |
|---|---|
| `manage_observation_log.json` | `AI4SH_observation_log_agrolab.xlsx` |
| `manage_observation.json` | `AI4SH_observation_agrolab.xlsx` |

## Next step

Proceed to [Manage Agrolab observation log].

**Or, do it in one step instead**: [Insert Agrolab data][insert_wetlab_data] covers both tables via the single-step `insert_tabular_data` route, with no separate manage step.

[sample translate step]: {{ site.baseurl }}/sample/translate_geotag_sample/
[Load sample data]: {{ site.baseurl }}/sample/
[Load utility data]: {{ site.baseurl }}/utility/
[Manage Agrolab observation log]: {{ site.baseurl }}/observation/manage_agrolab_obs_log/
[setup_process_path_resolution]: {{ site.baseurl }}/setup_process/path_resolution/
[insert_wetlab_data]: {{ site.baseurl }}/observation/insert_agrolab/
[obs_texture]: {{ site.baseurl }}/observation/#derived-soil-texture-classes
