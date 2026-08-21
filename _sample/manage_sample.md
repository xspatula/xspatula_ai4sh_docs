---
title: "Manage Samples"
layout: single
sidebar:
  nav: "sample"
  nav2: "loading_data"
excerpt: "Inserting geolocated profile sample records into the AI4SH database. Each sample links a sampling log, a geolocation, and a depth profile interval."
permalink: /sample/manage_sample/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

A geolocated profile sample ties together a sampling log (the collection event), a geolocation (the GPS position), and a depth profile interval (the vertical extent of the collected material). Once samples exist in the database, wetlab and spectral observations can be linked to them.

## Prerequisites

- [Manage sample geolocation] must be complete.
- [Manage sampling log] (from the dataset metadata workflow) must be complete.
- Utility records required by samples must be populated (from [Load utility data]): `unit`, `juxtaposition`.

## Notebook cell

In `load_ai4sh_sample_data.ipynb`, the **Manage samples** cell runs:

```python
process_file = 'import_data/sample/manage_process/manage_geolocated_profile_sample.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/sample/manage_process/manage_geolocated_profile_sample.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_geolocated_profile_sample",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "sampling_log_id__sampling_log_name": "ai4sh_es_arkaute_2024",
        "tag": "10-1",
        "geolocation_id__geolocation_name": "es_arkaute@10-1",
        "area_coverage": 16.0,
        "area_unit_id__area_unit_name": "m^2",
        "profile_min": 0,
        "profile_max": 20,
        "sampling_date": 20241015,
        "juxtaposition_id__juxtaposition_name": "uniform",
        "composition_id__composition_name": "composite",
        "proximity_id__proximity_name": "general"
      }
    }
  ]
}
```

The example file contains 670 entries — one per sample across all AI4SH field sites.

## Parameters

| Parameter | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: parent sampling log by name |
| `tag` | Short sample identifier within the sampling log (e.g. `10-1`) |
| `geolocation_id__geolocation_name` | FK lookup: geolocation by name (must exist in `observation.geolocation`) |
| `area_coverage` | Surface area represented by the sample |
| `area_unit_id__area_unit_name` | FK lookup: unit for `area_coverage` (e.g. `m^2`) |
| `profile_min` | Top of the sampled depth interval (cm) |
| `profile_max` | Bottom of the sampled depth interval (cm) |
| `sampling_date` | Date of collection as integer (`YYYYMMDD`) |
| `juxtaposition_id__juxtaposition_name` | FK lookup: local thematic setting (e.g. `uniform`) |
| `composition_id__composition_name` | FK lookup: sample composition type (e.g. `composite`) |
| `proximity_id__proximity_name` | FK lookup: spatial proximity type (e.g. `general`) |

## Depth profile

`profile_min` and `profile_max` define the vertical extent of the collected material in centimetres below the surface. For a standard 0–20 cm topsoil sample: `profile_min = 0`, `profile_max = 20`. Sub-samples at different depths from the same location are entered as separate rows with the same geolocation name but different profile bounds and tags.

## Next step

All sample data is now loaded. Proceed to [Load wetlab data] or [Load spectral data] to add analytical observations.

**Or, do it in one step instead**: [Insert sample data][insert_sample_data] covers both sample tables, including this one, via the single-step `insert_tabular_data` route.

[Manage sample geolocation]: /sample/manage_sample_geolocation/
[Manage sampling log]: /dataset_meta/manage_sampling_log/
[Load utility data]: /utility/
[Load wetlab data]: /wetlab/
[Load spectral data]: /spectra/
[insert_sample_data]: /sample/insert_sample_data/
