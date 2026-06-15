---
title: "Translate Geotag Sample"
layout: single
sidebar:
  nav: "sample"
  nav2: "loading_data"
excerpt: "Translating both sample Excel files to JSON process files in a single process_file call. One translate process file handles geolocation and geolocated_profile_sample together."
permalink: /sample/translate_geotag_sample/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Both sample Excel files are translated in a single notebook cell. Unlike most other translate steps, this cell calls a `process_file` directly rather than a `job_file` — the translate process file contains two entries, one per table.

## Prerequisites

[Load dataset metadata] must be complete. Sampling logs must exist before samples can be managed.

## Notebook cell

In `load_ai4sh_sample_data.ipynb`, the **Translate geotag sample** cell runs:

```python
process_file = 'import_data/sample/process/translate_geotag_sample.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

Note the cell uses `process_file`, not `job_file` — no pilot file is involved.

## Translate process file

**Path**: `./ai4sh/import_data/sample/process/translate_geotag_sample.json`

```json
{
  "process": [
    {
      "process": "translate_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_geolocation",
        "tabular_data_path": "../../sample/excel/AI4SH_geolocation.xlsx",
        "dst_path": "../../sample/manage_process"
      }
    },
    {
      "process": "translate_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_geolocated_profile_sample",
        "tabular_data_path": "../../sample/excel/AI4SH_samples_geotag.xlsx",
        "dst_path": "../../sample/manage_process"
      }
    }
  ]
}
```

The two entries are executed in sequence. Each reads an Excel source file and writes one manage process file to `./ai4sh/import_data/sample/manage_process/`.

## Source files

**`AI4SH_geolocation.xlsx`** — one row per sampling point. Required columns:

| Column | Description |
|---|---|
| `name` | Unique geolocation identifier (e.g. `es_arkaute@10-1`) |
| `x_coordinate` | Longitude (decimal degrees) |
| `y_coordinate` | Latitude (decimal degrees) |

The `@` separator in the name is a convention: `site@sample_tag`. It is not enforced by the framework but is used consistently in the AI4SH project.

**`AI4SH_samples_geotag.xlsx`** — one row per sample. Required columns include sampling log name, tag, geolocation name, depth profile bounds, and optional juxtaposition/composition/proximity classifications.

## Output

Two manage process files are written:

| File | Source |
|---|---|
| `manage_geolocation.json` | `AI4SH_geolocation.xlsx` |
| `manage_geolocated_profile_sample.json` | `AI4SH_samples_geotag.xlsx` |

## Next step

Proceed to [Manage sample geolocation].

[Load dataset metadata]: /dataset_meta/
[Manage sample geolocation]: /sample/manage_sample_geolocation/
