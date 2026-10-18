---
title: "Insert Digit Soil Data"
layout: single
sidebar:
  nav: "observation"
  nav2: "loading_data"
excerpt: "Cell 3.2.6 of insert_ai4sh_data.ipynb inserts the Digit Soil enzymatic activity observation log and the activity rates of five soil enzymes per sample, translate and insert together in one cell."
permalink: /observation/insert_digitsoil/
author_profile: false
date: 2026-10-17 08:00:00 +0200
last_modified_at: 2026-10-17 08:00:00 +0200
---

Cell **3.2.6 Load Digit Soil enzymatic observation data** of `./ai4sh/import_data/insert_ai4sh_data.ipynb` (see [Load all data][all_data]) loads the soil enzymatic activity measured with the [Digit Soil](https://www.digit-soil.com) Soil Enzymatic Activity Reader (SEAR), model BOB. The reader measures the activity rate of five extracellular soil enzymes, which reflect the microbial turnover of carbon, nitrogen and phosphorus.

The cell follows the common pattern of block 3.2 — an observation log, then one row of indicator values per sample. See [Load observation data][observation].

## Prerequisites

- [Load sample data] (cell 3.1) must be complete.
- The provision `digit soil enzymatic activity reader bob` (alias `digit soil sear bob`, tag `digit-soil-bob`) and its five provision indicators must exist from [Load utility data] (block 1).

## Notebook cell

In `insert_ai4sh_data.ipynb`, block **3.2**, cell **3.2.6** runs:

```python
process_file = 'import_data/digit_soil/insert_process/insert_digitsoil_observation_log+observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `./ai4sh/import_data/digit_soil/insert_process/insert_digitsoil_observation_log+observation.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation_log",
        "tabular_data_path": "import_data/digit_soil/excel/AI4SH_observation_log_digitsoil.xlsx",
        "dst_path": "import_data/digit_soil/insert_process/staging"
      }
    },
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation",
        "tabular_data_path": "import_data/digit_soil/excel/AI4SH_SEAR_tg.xlsx",
        "dst_path": "import_data/digit_soil/insert_process/staging"
      }
    }
  ]
}
```

The observation log is inserted first, as the observations reference it by name.

## Source files

Source directory: `./ai4sh/import_data/digit_soil/excel/`

**`AI4SH_observation_log_digitsoil.xlsx`** — 13 rows, one per sampling log analysed (all pilot sites and sampling years). Columns:

| Column | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: the sampling log (name or tag) |
| `provision_id__provision_name` | FK lookup: the provision (name, alias or tag), here `digit soil sear bob` |
| `contact_name`, `contact_email` | The person responsible for the observations |
| `name` | `auto` composes `<sampling log>@<provision>`, see [Automatic naming and tags][auto_naming] |
| `abstract` | Optional free text |
| `preparation_id__preparation_name`, `preservation_id__preservation_name`, `storage_id__storage_name`, `transportation_id__transportation_name` | FK lookups: sample handling; `none` if not applicable |
| `laboratory` | Method tier flag, `true` |

The samples are kept and transported cold (`cold`) and analysed without further preparation (`none`).

**`AI4SH_SEAR_tg.xlsx`** — 268 rows, one per sample, spread over 9 observation logs. Columns:

| Column | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: the observation log, e.g. `ai4sh-se-loennstorp 2024@digit soil enzymatic activity reader bob` |
| `sample_id__sample_name` | FK lookup: the sample, e.g. `4-a@0-20` (looked up within the sampling log of the observation log) |
| `observed_at` | Date of the analysis (`YYYYMMDD`) |
| `@leu activity rate` … `@xyl activity rate` | Indicator values, see below |

## Indicators

| Excel column | Indicator | Unit | Analysis method |
|---|---|---|---|
| `@leu activity rate` | `leu activity rate` | pmol·min⁻¹ | `digitsoil soil enzymatic activity reader` |
| `@gla activity rate` | `gla activity rate` | pmol·min⁻¹ | same |
| `@gls activity rate` | `gls activity rate` | pmol·min⁻¹ | same |
| `@pho activity rate` | `pho activity rate` | pmol·min⁻¹ | same |
| `@xyl activity rate` | `xyl activity rate` | pmol·min⁻¹ | same |

The indicator, unit and analysis method of each `@` column come from `provision_indicator.xlsx` (block 1); the Excel file only carries the values. Each sample becomes one row in `observation.observation` and five rows in `observation.observation_measurement`.

## Two-step route

The two-step route (translate, then manage) works as described for Agrolab on [Translate Agrolab data][translate_agrolab], with the folder `import_data/digit_soil/` and the two Excel files above.


[observation]: {{ site.baseurl }}/observation/
[insert_agrolab]: {{ site.baseurl }}/observation/insert_agrolab/
[translate_agrolab]: {{ site.baseurl }}/observation/translate_agrolab/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[auto_naming]: {{ site.baseurl }}/setup_process/auto_naming/
[Load sample data]: {{ site.baseurl }}/sample/
[Load utility data]: {{ site.baseurl }}/utility/
[all_data]: {{ site.baseurl }}/all_data/
