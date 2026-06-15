---
title: "FTIR Observation Log"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Inserting FTIR observation log records into the AI4SH database. Each record links a sampling log to the Bruker Invenio FTIR provision with sample handling metadata."
permalink: /spectra/manage_ftir_obs_log/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

A FTIR observation log links a sampling log to the `bruker invenio s ftir` provision. FTIR requires milled samples, so the preparation is `ds2m` (dried, sieved to 2 mm, then milled) rather than the `ds2` used by the NIR instruments.

## Prerequisites

- [Manage FTIR spectrometer] must be complete.
- Sampling logs must exist from [Load dataset metadata].
- Preparation, preservation, and storage utility records must exist from [Load utility data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage FTIR observation log** cell runs:

```python
job_file = 'import_data/FTIR_spectra/job_ftir_observation_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/FTIR_spectra/job_ftir_observation_log.json`

```json
{
  "process": {
    "job_folder": "import_data/FTIR_spectra/observation_log",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_observation_log_pilot.txt"
  }
}
```

## Process file example

**Path**: `./ai4sh/import_data/FTIR_spectra/observation_log/process/ai4sh_ch_lens_2026@ftir_ds2m_observation_log.json`

```json
{
  "process": [
    {
      "process": "manage_observation_log",
      "parameters": {
        "sampling_log_id__sampling_log_name": "ai4sh_ch_lens_2025",
        "provision_id__provision_name": "bruker invenio s ftir",
        "name": "ai4sh_ch_lens_2026@ftir_ds2m",
        "contact_name": "thomas gumbricht",
        "contact_email": "thomas.gumbricht@natgeo.su.se",
        "preparation_id__preparation_name": "ds2m",
        "preservation_id__preservation_name": "ds2",
        "storage_id__storage_name": "amb",
        "laboratory": 1
      }
    }
  ]
}
```

14 observation log process files — one per field site.

## Parameters

| Parameter | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: parent sampling log |
| `provision_id__provision_name` | FK lookup: `bruker invenio s ftir` |
| `name` | Observation log name — pattern: `<sampling_log_year+1>@ftir_<preparation>` |
| `contact_name` | Responsible person |
| `contact_email` | Contact email |
| `preparation_id__preparation_name` | FK lookup: `ds2m` (dried, sieved 2 mm, milled) |
| `preservation_id__preservation_name` | FK lookup: preservation method |
| `storage_id__storage_name` | FK lookup: storage condition |
| `laboratory` | Integer flag — `1` for laboratory observations |

## Next step

Proceed to [Manage FTIR observation].

[Manage FTIR spectrometer]: /spectra/manage_ftir_spectrometer/
[Load dataset metadata]: /dataset_meta/
[Load utility data]: /utility/
[Manage FTIR observation]: /spectra/manage_ftir_obs/
