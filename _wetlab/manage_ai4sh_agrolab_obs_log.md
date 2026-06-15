---
title: "Manage Agrolab Observation Log"
layout: single
sidebar:
  nav: "wetlab"
  nav2: "loading_data"
excerpt: "Inserting Agrolab observation log records into the AI4SH database. Each record links a sampling log to the Agrolab provision and records laboratory handling metadata."
permalink: /wetlab/manage_ai4sh_agrolab_obs_log/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

An observation log record bridges a sampling log (a field collection event) and a provision (a laboratory protocol). For the Agrolab wetlab data, each entry records which sampling log's samples were analysed, under which protocol, and how they were handled in the lab. Individual observations are later linked to these log records.

## Prerequisites

- [Translate Agrolab wetlab data] must be complete.
- The Agrolab provision (`ai4sh-agrolab`) and its indicators must exist from [Load utility data].
- Sampling logs must exist from [Load dataset metadata].

## Notebook cell

In `load_ai4sh_wetlab_data.ipynb`, the **Manage Agrolab observation log** cell runs:

```python
process_file = 'import_data/wetlab/agrolab/manage_process/manage_observation_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/wetlab/agrolab/manage_process/manage_observation_log.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_observation_log",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "sampling_log_id__sampling_log_name": "ai4sh_dk_foulum_2024",
        "provision_id__provision_name": "ai4sh-agrolab",
        "contact_name": "thomas gumbricht",
        "contact_email": "thomas.gumbricht@natgeo.su.se",
        "name": "auto",
        "preparation_id__preparation_name": "ds2",
        "preservation_id__preservation_name": "ds2",
        "storage_id__storage_name": "amb",
        "transportation_id__transportation_name": "none",
        "laboratory": true
      }
    }
  ]
}
```

The example file contains 12 entries — one per field site sampling log.

## Parameters

| Parameter | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: parent sampling log by name |
| `provision_id__provision_name` | FK lookup: Agrolab provision record (defines the indicators measured) |
| `contact_name` | Name of the responsible person |
| `contact_email` | Email of the responsible person |
| `name` | Log name — `"auto"` generates `<sampling_log_name>@<provision_name>` |
| `preparation_id__preparation_name` | FK lookup: sample preparation method used (e.g. `ds2` = dried and sieved to 2 mm) |
| `preservation_id__preservation_name` | FK lookup: preservation method |
| `storage_id__storage_name` | FK lookup: storage condition (e.g. `amb` = ambient) |
| `transportation_id__transportation_name` | FK lookup: transportation method |
| `laboratory` | Boolean — `true` for laboratory-based analyses |

## Auto-naming

When `name` is set to `"auto"`, the framework generates the observation log name as `<sampling_log_name>@<provision_name>`. For example, the entry above produces the name `ai4sh_dk_foulum_2024@ai4sh-agrolab`. This name is then used as the FK reference in the observation manage file via `observation_log_id__observation_log_name`.

## Next step

Proceed to [Manage Agrolab observation].

[Translate Agrolab wetlab data]: /wetlab/translate_ai4sh_wetlab_data/
[Load utility data]: /utility/
[Load dataset metadata]: /dataset_meta/
[Manage Agrolab observation]: /wetlab/manage_ai4sh_wetlab_data/
