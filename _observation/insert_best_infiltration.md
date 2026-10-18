---
title: "Insert BEST Infiltration Data"
layout: single
sidebar:
  nav: "observation"
  nav2: "loading_data"
excerpt: "Cell 3.2.7 of insert_ai4sh_data.ipynb inserts the BEST (Beerkan Estimation of Soil Transfer parameters) field infiltration observation log and ten soil hydraulic properties per sample."
permalink: /observation/insert_best_infiltration/
author_profile: false
date: 2026-10-17 08:00:00 +0200
last_modified_at: 2026-10-17 08:00:00 +0200
---

Cell **3.2.7 Load BEST infiltration observation data** of `./ai4sh/import_data/insert_ai4sh_data.ipynb` (see [Load all data][all_data]) loads field infiltration results. Water infiltration is measured in the field with the Beerkan ring method, and the BEST algorithm (Beerkan Estimation of Soil Transfer parameters) derives the soil hydraulic properties from the infiltration curve and the soil texture. Only the derived properties are loaded, not the raw infiltration times.

The cell follows the common pattern of block 3.2 — an observation log, then one row of indicator values per sample. See [Load observation data][observation].

**Not yet run:** cell 3.2.7 is switched off in the notebook (`%%script false`). The Excel column `@porosity` has no matching provision indicator; the catalogue names it `porspace`. One of them must be renamed before the cell can run.
{: .notice--warning}

## Prerequisites

- [Load sample data] (cell 3.1) must be complete.
- The provision `beerkan estimation of soil hydraulic properties` (alias `best infiltration`, tag `best-infiltration`) and its ten provision indicators must exist from [Load utility data] (block 1).

## Notebook cell

In `insert_ai4sh_data.ipynb`, block **3.2**, cell **3.2.7** runs:

```python
%%script false --no-raise-error
process_file = 'import_data/infiltration/insert_process/insert_infiltration_observation_log+observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Process file

**Path**: `./ai4sh/import_data/infiltration/insert_process/insert_infiltration_observation_log+observation.json`

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation_log",
        "tabular_data_path": "import_data/infiltration/excel/AI4SH_observation_log_infiltration.xlsx",
        "dst_path": "import_data/infiltration/insert_process/staging"
      }
    },
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation",
        "tabular_data_path": "import_data/infiltration/excel/AI4SH_infiltration_tg_cleaned.xlsx",
        "dst_path": "import_data/infiltration/insert_process/staging"
      }
    }
  ]
}
```

The observation log is inserted first, as the observations reference it by name.

## Source files

Source directory: `./ai4sh/import_data/infiltration/excel/`

**`AI4SH_observation_log_infiltration.xlsx`** — 12 rows, one per sampling log. Columns:

| Column | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: the sampling log (name or tag) |
| `provision_id__provision_name` | FK lookup: the provision (name, alias or tag), here `best infiltration` |
| `contact_name`, `contact_email` | The person responsible for the observations |
| `name` | `auto` composes `<sampling log>@<provision>`, see [Automatic naming and tags][auto_naming] |
| `abstract` | Optional free text |
| `preparation_id__preparation_name`, `preservation_id__preservation_name`, `storage_id__storage_name`, `transportation_id__transportation_name` | FK lookups: sample handling; `none` if not applicable |
| `field` | Method tier flag, `true` |

The sampling logs are given by their tags (e.g. `ai4sh_dk_foulum_2024`); a tag resolves like a name.

**`AI4SH_Infiltration_tg_cleaned.xlsx`** — 98 rows, one per infiltration measurement. Columns:

| Column | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: the observation log, given as `<sampling log tag>@<provision alias>`, e.g. `ai4sh_es_arkaute_2024@best infiltration` |
| `sample_id__sample_name` | FK lookup: the sample, e.g. `10-1@0-20` |
| `subsample` | `a` to `h`: repeated measurements at the same sample (up to `e` in the current file) |
| `@porosity` … `@m shape parameter of retention curve` | Indicator values, see below |

There is no `observed_at` column, so the observation date is set to the date of loading.

## Indicators

| Excel column | Indicator | Unit |
|---|---|---|
| `@porosity` | `porspace` (name mismatch, see above) | vol·vol⁻¹ |
| `@final infiltration rate` | `final infiltration rate` | m·day⁻¹ |
| `@sorptivity` | `sorptivity` | m·day⁻¹ᐟ² |
| `@saturated hydraulic conductivity` | `saturated hydraulic conductivity` | m·day⁻¹ |
| `@field capacity` | `field capacity` | vol·vol⁻¹ |
| `@plant available water` | `plant available water` | vol·vol⁻¹ |
| `@saturated water content` | `saturated water content` | vol·vol⁻¹ |
| `@hg water pressure head scale parameter` | `hg water pressure head scale parameter` | m |
| `@n shape parameter of retention curve` | `n shape parameter of retention curve` | unitless |
| `@m shape parameter of retention curve` | `m shape parameter of retention curve` | unitless |

All indicators share the analysis method `beerkan estimation of soil transfer parameters`.

The observation process `manage_beerkan_observation` (see [Automatic naming and tags][auto_naming]) is not used here: the derived properties are ordinary indicator values loaded with `manage_observation`.

## Two-step route

The two-step route (translate, then manage) works as described for Agrolab on [Translate Agrolab data][translate_agrolab], with the folder `import_data/infiltration/` and the two Excel files above.


[observation]: {{ site.baseurl }}/observation/
[insert_agrolab]: {{ site.baseurl }}/observation/insert_agrolab/
[translate_agrolab]: {{ site.baseurl }}/observation/translate_agrolab/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[auto_naming]: {{ site.baseurl }}/setup_process/auto_naming/
[Load sample data]: {{ site.baseurl }}/sample/
[Load utility data]: {{ site.baseurl }}/utility/
[all_data]: {{ site.baseurl }}/all_data/
