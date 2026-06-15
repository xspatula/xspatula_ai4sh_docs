---
title: "Manage Observation Utilities"
layout: single
sidebar:
  nav: "utility"
  nav2: "loading_data"
excerpt: "Inserting the 20 observation utility catalogue tables into the AI4SH database from the JSON process files produced by the translate step."
permalink: /utility/manage_observation_utilities/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

The fourth utility step inserts all non-inheritance observation utility catalogues into the database. The 20 tables are processed in the order defined by the pilot file, which respects foreign key dependencies. Tables linking to other tables with a foreign key use an __ notation in parameter keys to resolve foreign keys by name rather than ID.


## Prerequisite

Run [Translate observation utilities] first to generate the manage process files.

## Notebook cell

In `load_ai4sh_utility_data.ipynb`, the **Manage observation utilities** cell runs:

```python
job_file = 'import_data/utility/job_manage_observation_utility.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/utility/job_manage_observation_utility.json`

```json
{
  "process": {
    "job_folder": "import_data/utility/observation",
    "process_sub_folder": "manage_process",
    "pilot_file": "manage_observation_utility.txt"
  }
}
```

## Pilot file

**Path**: `./ai4sh/import_data/utility/observation/manage_observation_utility.txt`

```
manage_analysis_method.json
manage_apparatus.json
manage_order.json
manage_license.json
manage_location_method.json
manage_method_tier.json
manage_preparation.json
manage_preservation.json
manage_provider.json
manage_quantity.json
manage_setting_system.json
manage_spatial_reference.json
manage_storage.json
manage_transportation.json
manage_unit.json
manage_family.json
manage_genus.json
manage_indicator.json
manage_juxtaposition.json
manage_profiling.json
```

The order mirrors the translate pilot file: independent tables first (lines 1–15), then FK-dependent tables (lines 16–20).

## Manage process file structure

Each manage process file contains one JSON object per row from the source Excel file. Example from `manage_analysis_method.json` (first entry):

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_analysis_method",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "name": "reflectance",
        "display_name": "Reflectance spectroscopy",
        "standard_schema": "none",
        "standard_code": "none",
        "url": "none"
      }
    }
  ]
}
```

The `parameters` keys map directly to database column names in the target table. The full file contains one object per row in `analysis_method.xlsx`.

## Flags

| Flag | Default | Effect |
|---|---|---|
| `delete` | `false` | Removes the record if `true` |
| `overwrite` | `false` | Updates an existing record with matching primary key if `true` |

These flags are set during the translate step from corresponding columns in the Excel file, or default to `false` if those columns are absent.

## What gets inserted

| Manage file | Database table |
|---|---|
| `manage_analysis_method.json` | `observation_utility.analysis_method` |
| `manage_apparatus.json` | `observation_utility.apparatus` |
| `manage_order.json` | `observation_utility.classification_order` |
| `manage_license.json` | `observation_utility.license` |
| `manage_location_method.json` | `observation_utility.location_method` |
| `manage_method_tier.json` | `observation_utility.method_tier` |
| `manage_preparation.json` | `observation_utility.preparation` |
| `manage_preservation.json` | `observation_utility.preservation` |
| `manage_provider.json` | `observation_utility.provider` |
| `manage_quantity.json` | `observation_utility.quantity` |
| `manage_setting_system.json` | `observation_utility.setting_system` |
| `manage_spatial_reference.json` | `observation_utility.spatial_reference` |
| `manage_storage.json` | `observation_utility.storage` |
| `manage_transportation.json` | `observation_utility.transportation` |
| `manage_unit.json` | `observation_utility.unit` |
| `manage_family.json` | `observation_utility.classification_family` |
| `manage_genus.json` | `observation_utility.classification_genus` |
| `manage_indicator.json` | `observation_utility.indicator` |
| `manage_juxtaposition.json` | `observation_utility.juxtaposition` |
| `manage_profiling.json` | `observation_utility.profiling` |

## Next step

Proceed to [Translate observation utilities with inheritance] to handle `provision`, `provision_indicator`, and `provision_serial_nr`.

[Translate observation utilities]: /utility/translate_observation_utilities/
[Translate observation utilities with inheritance]: /utility/translate_observation_utilities_inherit/
