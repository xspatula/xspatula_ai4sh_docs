---
title: "Manage Dataset"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Inserting dataset records into the AI4SH database. A dataset is the top-level grouping for all data from a single source, usually with a distinct thematic content."
permalink: /dataset_meta/manage_dataset/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

A dataset is the top-level grouping for data contributed by a single data source, typically with a coherent thematic scope (e.g. LUCAS topsoil, a national soil survey, a project-specific campaign series). Every campaign must belong to a dataset.

## Prerequisites

- [Manage data source] must be complete.
- Utility records referenced by datasets must be populated (from [Load utility data]): `license`, `spatial_reference`, `profiling`.

## Notebook cell

In `load_ai4sh_dataset_meta.ipynb`, the **Manage dataset** cell runs:

```python
process_file = 'import_data/dataset/manage_process/manage_dataset.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/dataset/manage_process/manage_dataset.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_dataset",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "name": "ai4soilhealth",
        "alias": "ai4sh",
        "display_name": "AI4SoilHealth",
        "data_source_id__data_source_name": "ai4soilhealth",
        "contact_name": "stockholm university",
        "contact_email": "thomas.gumbricht@natgeo.su.se",
        "begun_at": 20230101,
        "ended_at": 20261231,
        "species_id__species_name": "soil",
        "profiling_id__profiling_name": "depth_cm",
        "setting_system_id__setting_system_name_array": "agricultural field",
        "license_id__license_name": "mit",
        "substance_array": "soil",
        "keyword_array": "topsoil, soil health, soil health indicator, europe",
        "url": "https://ai4soilhealth.eu",
        "abstract": "AI4SoilHealth…",
        "field": 1.0,
        "home": 1.0,
        "laboratory": 1,
        "satellite": 1.0,
        "territory_id__territory_name": "eu",
        "spatial_reference_id__spatial_reference_name": "geographic"
      }
    }
  ]
}
```

## Parameters

| Parameter | Description |
|---|---|
| `name` | Full name of the dataset (lowercase) |
| `alias` | Short identifier used for FK lookup in campaign manage files |
| `display_name` | Display label |
| `data_source_id__data_source_name` | FK lookup: data source by alias |
| `contact_name` | Dataset contact name |
| `contact_email` | Dataset contact email |
| `begun_at` | Start date as integer (`YYYYMMDD`) |
| `ended_at` | End date as integer (`YYYYMMDD`) |
| `species_id__species_name` | FK lookup: biological classification (typically `soil`) |
| `profiling_id__profiling_name` | FK lookup: z-dimension profiling method (e.g. `depth_cm`) |
| `setting_system_id__setting_system_name_array` | Comma-separated setting systems (e.g. `agricultural field`) |
| `license_id__license_name` | FK lookup: license by name |
| `substance_array` | Comma-separated substance keywords |
| `keyword_array` | Comma-separated search keywords |
| `url` | Dataset landing page |
| `abstract` | Free-text description |
| `field` | Flag: 1 if field observations are included |
| `home` | Flag: 1 if home (e.g. slaking app) observations are included |
| `laboratory` | Flag: 1 if laboratory analyses are included, 0 otherwise |
| `satellite` | Flag: 1 if satellite-derived data are included |
| `territory_id__territory_name` | FK lookup: geographic territory |
| `spatial_reference_id__spatial_reference_name` | FK lookup: coordinate reference system |

## Next step

Proceed to [Manage campaign].

[Manage data source]: /dataset_meta/manage_data_source/
[Load utility data]: /utility/
[Manage campaign]: /dataset_meta/manage_campaign/
