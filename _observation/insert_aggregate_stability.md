---
title: "Insert Aggregate Stability Data"
layout: single
sidebar:
  nav: "observation"
  nav2: "loading_data"
excerpt: "Cell 3.2.9 of insert_ai4sh_data.ipynb inserts the aggregate stability observation log and the aggregate stability index per sample from the Slakes smartphone app."
permalink: /observation/insert_aggregate_stability/
author_profile: false
date: 2026-10-17 08:00:00 +0200
last_modified_at: 2026-10-17 08:00:00 +0200
---

Cell **3.2.9 Load aggregate stability data** of `./ai4sh/import_data/insert_ai4sh_data.ipynb` (see [Load all data][all_data]) loads soil aggregate stability measured with the smartphone app [Slakes](https://soilhealthinstitute.org/app/uploads/2023/12/SHI-Slakes_App_One-Pager-v2.pdf). The app photographs dry soil aggregates submerged in water and computes a stability index from how fast they slake (fall apart).

The cell follows the common pattern of block 3.2 — an observation log, then one row of indicator values per sample. See [Load observation data][observation].

**Slakes only:** the laboratory wet sieving results (provision `aggstab wet sieving`, Arkaute 2025) are prepared in `AI4SH_observation_log_aggstab.xlsx`, but their observation file is not ready and they are not loaded by this cell.
{: .notice--info}

## Prerequisites

- [Load sample data] (cell 3.1) must be complete.
- The provision `slakes aggregate stability app` (alias and tag `slakes`) and its provision indicator `aggregate stability` must exist from [Load utility data] (block 1).

## Notebook cell

In `insert_ai4sh_data.ipynb`, block **3.2**, cell **3.2.9** runs:

```python
process_file = 'import_data/aggstab/insert_process/insert_aggstab_observation_log+observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `./ai4sh/import_data/aggstab/insert_process/insert_aggstab_observation_log+observation.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation_log",
        "tabular_data_path": "import_data/aggstab/excel/AI4SH_observation_log_slakes.xlsx",
        "dst_path": "import_data/aggstab/insert_process/staging"
      }
    },
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation",
        "tabular_data_path": "import_data/aggstab/excel/AI4SH_Slakes_tg.xlsx",
        "dst_path": "import_data/aggstab/insert_process/staging"
      }
    }
  ]
}
```

The observation log is inserted first, as the observations reference it by name.

## Source files

Source directory: `./ai4sh/import_data/aggstab/excel/`

**`AI4SH_observation_log_slakes.xlsx`** — 3 rows, one per sampling log measured with Slakes. Columns:

| Column | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: the sampling log (name or tag) |
| `provision_id__provision_name` | FK lookup: the provision (name, alias or tag), here `slakes` |
| `contact_name`, `contact_email` | The person responsible for the observations |
| `name` | `auto` composes `<sampling log>@<provision>`, see [Automatic naming and tags][auto_naming] |
| `abstract` | Optional free text |
| `preparation_id__preparation_name`, `preservation_id__preservation_name`, `storage_id__storage_name`, `transportation_id__transportation_name` | FK lookups: sample handling; `none` if not applicable |
| `home` | Method tier flag, `true` |

The samples are prepared by aggregate selection (`agg`), preserved by drying (`d`) and stored at ambient temperature (`amb`) before the test. The current observation file holds results for Foulum and Lönnstorp 2024; the Arkaute 2025 log has no Slakes observations yet.

**`AI4SH_Slakes_tg.xlsx`** — 21 rows, one per aggregate test. Columns:

| Column | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: the observation log, e.g. `ai4sh-dk-foulum 2024@slakes aggregate stability app` |
| `sample_id__sample_name` | FK lookup: the sample, e.g. `201-c5@0-20` |
| `subsample` | `a`, `b`: repeated tests on aggregates from the same sample |
| `@aggregate stability` | The Slakes stability index (unit `index`, analysis method `slakes soil aggregate stability`) |

There is no `observed_at` column, so the observation date is set to the date of loading.

The app-specific process `manage_app_aggregate_stability_observation` (see [Automatic naming and tags][auto_naming]) is not used here: the stability index is an ordinary indicator value loaded with `manage_observation`.

## Two-step route

The two-step route (translate, then manage) works as described for Agrolab on [Translate Agrolab data][translate_agrolab], with the folder `import_data/aggstab/` and the two Excel files above.


[observation]: {{ site.baseurl }}/observation/
[insert_agrolab]: {{ site.baseurl }}/observation/insert_agrolab/
[translate_agrolab]: {{ site.baseurl }}/observation/translate_agrolab/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[auto_naming]: {{ site.baseurl }}/setup_process/auto_naming/
[Load sample data]: {{ site.baseurl }}/sample/
[Load utility data]: {{ site.baseurl }}/utility/
[all_data]: {{ site.baseurl }}/all_data/
