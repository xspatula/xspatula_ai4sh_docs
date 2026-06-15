---
title: "Manage Observation Utilities with Inheritance"
layout: single
sidebar:
  nav: "utility"
  nav2: "loading_data"
excerpt: "Inserting provision, provision_indicator, and provision_serial_nr into the AI4SH database. Parameter keys use __ notation to resolve foreign keys by name at insert time."
permalink: /utility/manage_observation_utilities_inherit/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

The sixth and final utility step inserts the inheritance-dependent observation utility records: `provision`, `provision_indicator`, and `provision_serial_nr`. After this step, the full controlled vocabulary is in place and dataset metadata loading can begin.

## Prerequisite

Run [Translate observation utilities with inheritance] first. The `apparatus`, `provider`, `method_tier`, `indicator`, `analysis_method`, and `unit` tables must already be populated (from [Manage observation utilities]).

## Notebook cell

In `load_ai4sh_utility_data.ipynb`, the **Manage observation utilities with inheritance** cell runs:

```python
job_file = 'import_data/utility/job_manage_observation_utility_inherit.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/utility/job_manage_observation_utility_inherit.json`

```json
{
  "process": {
    "job_folder": "import_data/utility/observation",
    "process_sub_folder": "manage_process",
    "pilot_file": "manage_observation_utility_inherit.txt"
  }
}
```

## Pilot file

**Path**: `./ai4sh/import_data/utility/observation/manage_observation_utility_inherit.txt`

```
manage_provision.json
manage_provision_indicator.json
manage_provision_serial_nr.json
```

## Manage process file structure

### manage_provision.json (first entry)

A provision combines one apparatus, one provider, and one method tier into a named instrument–lab–professionality combination:

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_provision",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "provider_id__provider_name": "metrohm",
        "apparatus_id__apparatus_name": "visnir",
        "method_tier_id__method_tier_name": "laboratory",
        "name": "nirs xds rapid content analyzer",
        "alias": "metrohm-nirs-xds",
        "display_name": "NIRS XDS Rapid Content Analyzer",
        "abstract": "VISNIR laboratory grade spectrometer",
        "laboratory": 1.0
      }
    }
  ]
}
```

### manage_provision_indicator.json (first entry)

A provision indicator links a provision to the specific indicators it delivers, with the analysis method and unit for each:

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_provision_indicator",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "provision_id__provision_name": "metrohm-nirs-xds",
        "indicator_id__indicator_name": "reflectance",
        "analysis_method_id__analysis_method_name": "reflectance",
        "unit_id__unit_name": "unitless"
      }
    }
  ]
}
```

## The `__` notation

Parameter keys containing `__` instruct the framework to resolve a foreign key by name rather than by ID:

```
"provider_id__provider_name": "metrohm"
```

This means: look up the `provider` record where `provider_name = "metrohm"`, then use its `provider_id` for the insert. The referenced record must already exist. If it does not, the insert fails with a foreign key error — re-check that the prerequisite manage steps completed successfully.

## What gets inserted

| File | Database table |
|---|---|
| `manage_provision.json` | `observation_utility.provision` |
| `manage_provision_indicator.json` | `observation_utility.provision_indicator` |
| `manage_provision_serial_nr.json` | `observation_utility.provision_serial_nr` |

## Next step

All utility data is now loaded. Proceed to [Load dataset metadata] to register data sources, persons, datasets, campaigns, and sampling logs.

[Translate observation utilities with inheritance]: /utility/translate_observation_utilities_inherit/
[Manage observation utilities]: /utility/manage_observation_utilities/
[Load dataset metadata]: /dataset_meta/
