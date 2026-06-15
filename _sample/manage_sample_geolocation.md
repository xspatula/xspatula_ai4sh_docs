---
title: "Manage Sample Geolocation"
layout: single
sidebar:
  nav: "sample"
  nav2: "loading_data"
excerpt: "Inserting geolocation records into the AI4SH database. Each geolocation is a named GPS coordinate used to anchor one or more samples."
permalink: /sample/manage_sample_geolocation/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Geolocation records store the GPS coordinates of sampling points. Each geolocation has a unique name that is later referenced by sample records via the `__` FK lookup notation. Geolocations must be inserted before samples.

## Prerequisite

Run [Translate geotag sample] first.

## Notebook cell

In `load_ai4sh_sample_data.ipynb`, the **Manage sample geolocation** cell runs:

```python
process_file = 'import_data/sample/manage_process/manage_geolocation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/sample/manage_process/manage_geolocation.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_geolocation",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "name": "es_arkaute@10-1",
        "x_coordinate": -2.62319677333332,
        "y_coordinate": 42.8543979992745
      }
    }
  ]
}
```

The example file contains 287 entries (one per sampling point in the AI4SH project). The `name` field is the handle used by the FK lookup `geolocation_id__geolocation_name` in the sample manage file.

## Parameters

| Parameter | Description |
|---|---|
| `name` | Unique geolocation identifier — referenced by samples via FK lookup |
| `x_coordinate` | Longitude in decimal degrees (WGS84) |
| `y_coordinate` | Latitude in decimal degrees (WGS84) |

## Naming convention

The AI4SH project uses the pattern `site@tag` for geolocation names (e.g. `es_arkaute@10-1`). The `@` is a visual separator only — the framework treats the entire string as a single identifier. Consistency between geolocation names in `AI4SH_geolocation.xlsx` and the references in `AI4SH_samples_geotag.xlsx` is essential: a mismatch will cause the sample manage step to fail with a foreign key error.

## Next step

Proceed to [Manage samples].

[Translate geotag sample]: /sample/translate_geotag_sample/
[Manage samples]: /sample/manage_sample/
