---
title: "Insert Microbiometer Data"
layout: single
sidebar:
  nav: "observation"
  nav2: "loading_data"
excerpt: "Cell 3.2.8 of insert_ai4sh_data.ipynb inserts the Microbiometer observation logs and the microbial carbon, nitrogen and fungi/bacteria shares per sample, from the classic and the pro test kit."
permalink: /observation/insert_microbiometer/
author_profile: false
date: 2026-10-17 08:00:00 +0200
last_modified_at: 2026-10-17 08:00:00 +0200
---

Cell **3.2.8 Load Microbiometer observation data** of `./ai4sh/import_data/insert_ai4sh_data.ipynb` (see [Load all data][all_data]) loads results from the [Microbiometer](https://microbiometer.com) test kit, a smartphone-read colour test of soil microbial biomass. Two kit versions were used: **classic** (all sites) and **pro** (Zazari 2025), which also reports microbial nitrogen.

The cell follows the common pattern of block 3.2, with one difference: there is one observation log file but **two** observation files, one per kit. See [Load observation data][observation].

## Prerequisites

- [Load sample data] (cell 3.1) must be complete.
- The provisions `microbiometer-classic` and `microbiometer-pro` and their provision indicators must exist from [Load utility data] (block 1).

## Notebook cell

In `insert_ai4sh_data.ipynb`, block **3.2**, cell **3.2.8** runs:

```python
process_file = 'import_data/microbiometer/insert_process/insert_microbiometer_observation_log+observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `./ai4sh/import_data/microbiometer/insert_process/insert_microbiometer_observation_log+observation.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "execute": true,
      "overwrite": true,
      "parameters": {
        "process": "manage_observation_log",
        "tabular_data_path": "import_data/microbiometer/excel/AI4SH_observation_log_microbiometer.xlsx",
        "dst_path": "import_data/microbiometer/insert_process/staging"
      }
    },
    {
      "process": "insert_tabular_data",
      "execute": true,
      "overwrite": true,
      "parameters": {
        "process": "manage_observation",
        "tabular_data_path": "import_data/microbiometer/excel/AI4SH_microbiometer-classic_tg_cleaned.xlsx",
        "dst_path": "import_data/microbiometer/insert_process/staging"
      }
    },
    {
      "process": "insert_tabular_data",
      "execute": true,
      "overwrite": true,
      "parameters": {
        "process": "manage_observation",
        "tabular_data_path": "import_data/microbiometer/excel/AI4SH_microbiometer-pro_tg_cleaned.xlsx",
        "dst_path": "import_data/microbiometer/insert_process/staging"
      }
    }
  ]
}
```

The three entries run in order: observation log, classic observations, pro observations. All three write their staging JSON to the same folder, so `staging/manage_observation.json` only keeps the last (pro) file; the staging files are a debug trail, not an input.

## Source files

Source directory: `./ai4sh/import_data/microbiometer/excel/`

**`AI4SH_observation_log_microbiometer.xlsx`** — 11 rows: 10 sampling logs with `microbiometer-classic` and one (`ai4sh-gr-zazari 2025`) with `microbiometer-pro`. Columns:

| Column | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: the sampling log (name or tag) |
| `provision_id__provision_name` | FK lookup: the provision (name, alias or tag), here `microbiometer-classic` or `microbiometer-pro` |
| `contact_name`, `contact_email` | The person responsible for the observations |
| `name` | `auto` composes `<sampling log>@<provision>`, see [Automatic naming and tags][auto_naming] |
| `abstract` | Optional free text |
| `preparation_id__preparation_name`, `preservation_id__preservation_name`, `storage_id__storage_name`, `transportation_id__transportation_name` | FK lookups: sample handling; `none` if not applicable |
| `field` | Method tier flag, `true` |

**`AI4SH_microbiometer-classic_tg_cleaned.xlsx`** (114 rows) and **`AI4SH_microbiometer-pro_tg_cleaned.xlsx`** (19 rows) — one row per sample. Columns:

| Column | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: the observation log, e.g. `ai4sh-se-loennstorp 2024@microbiometer-classic testkit for soil fungi and bacteria` |
| `sample_id__sample_name` | FK lookup: the sample, e.g. `1-sand@0-20` |
| `@microbial carbon`, `@microbial nitrogen` (pro only), `@fungi(%)`, `@bacteria(%)` | Indicator values, see below |

There is no `observed_at` column, so the observation date is set to the date of loading.

## Indicators

| Excel column | Indicator | Unit | Classic | Pro |
|---|---|---|---|---|
| `@microbial carbon` | `microbial carbon` | µg·g⁻¹ | yes | yes |
| `@microbial nitrogen` | `microbial nitrogen` | µg·g⁻¹ | — | yes |
| `@fungi(%)` | `fungi(%)` | percent | yes | yes |
| `@bacteria(%)` | `bacteria(%)` | percent | yes | yes |

The analysis method is `microbiometer classic` or `microbiometer pro`. The pro provision also registers `microbial biomass`, which is not in the current file.

## Two-step route

The two-step route (translate, then manage) works as described for Agrolab on [Translate Agrolab data][translate_agrolab], with the folder `import_data/microbiometer/` and the three Excel files above.


[observation]: {{ site.baseurl }}/observation/
[insert_agrolab]: {{ site.baseurl }}/observation/insert_agrolab/
[translate_agrolab]: {{ site.baseurl }}/observation/translate_agrolab/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[auto_naming]: {{ site.baseurl }}/setup_process/auto_naming/
[Load sample data]: {{ site.baseurl }}/sample/
[Load utility data]: {{ site.baseurl }}/utility/
[all_data]: {{ site.baseurl }}/all_data/
