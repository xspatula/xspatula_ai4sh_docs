---
title: "LIBS Observation Log"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Inserting LIBS observation log records into the AI4SH database. Each record links a sampling log to the FOSS Micral LIBS provision with sample handling metadata."
permalink: /spectra/manage_libs_obs_log/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

A LIBS observation log links a sampling log to the `foss micral` provision. LIBS uses compressed pellets of milled soil (`ds2c` preparation — dried, sieved, and compressed) rather than the sieved or milled powders used by the NIR and FTIR instruments.

## Prerequisites

- [Manage LIBS spectrometer] must be complete.
- Sampling logs must exist from [Load dataset metadata].
- Preparation and preservation utility records must exist from [Load utility data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage LIBS observation log** cell runs:

```python
job_file = 'import_data/LIBS_spectra/job_libs_observation_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/LIBS_spectra/job_libs_observation_log.json`

```json
{
  "process": {
    "job_folder": "import_data/LIBS_spectra/observation_log",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_observation_log_pilot.txt"
  }
}
```

## Process file example

**Path**: `./ai4sh/import_data/LIBS_spectra/observation_log/process/ai4sh_ch_lens_2026@libs_ds2c_observation_log.json`

```json
{
  "process": [
    {
      "process": "manage_observation_log",
      "parameters": {
        "sampling_log_id__sampling_log_name": "ai4sh_ch_lens_2025",
        "provision_id__provision_name": "foss micral",
        "name": "ai4sh_ch_lens_2026@libs_ds2c",
        "contact_name": "thomas gumbricht",
        "contact_email": "thomas.gumbricht@natgeo.su.se",
        "preparation_id__preparation_name": "ds2c",
        "preservation_id__preservation_name": "ds2",
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
| `provision_id__provision_name` | FK lookup: `foss micral` |
| `name` | Observation log name — pattern: `<sampling_log_year+1>@libs_<preparation>` |
| `contact_name` | Responsible person |
| `contact_email` | Contact email |
| `preparation_id__preparation_name` | FK lookup: `ds2c` (dried, sieved 2 mm, compressed) |
| `preservation_id__preservation_name` | FK lookup: preservation method |
| `laboratory` | Integer flag — `1` for laboratory observations |

Note: LIBS observation logs do not include `storage_id__storage_name` or `transportation_id__transportation_name` fields (these are optional and omitted here).

## Next step

Proceed to [Manage LIBS observation].

[Manage LIBS spectrometer]: /spectra/manage_libs_spectrometer/
[Load dataset metadata]: /dataset_meta/
[Load utility data]: /utility/
[Manage LIBS observation]: /spectra/manage_libs_obs/
