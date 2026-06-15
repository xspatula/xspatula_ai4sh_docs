---
title: "Neospectra Observation Log"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Inserting Neospectra observation log records into the AI4SH database. Each record links a sampling log to the Neospectra provision with sample handling metadata."
permalink: /spectra/manage_neospectra_obs_log/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

A Neospectra observation log links a sampling log to the `neospectra` provision. Unlike the laboratory instruments, Neospectra records use `home: 1` instead of `laboratory: 1` to indicate that the instrument can be operated outside a controlled laboratory environment.

## Prerequisites

- [Manage Neospectra spectrometer] must be complete.
- Sampling logs must exist from [Load dataset metadata].
- Preparation and preservation utility records must exist from [Load utility data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage Neospectra observation log** cell runs:

```python
job_file = 'import_data/neospectra/job_neospectra_observation_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/neospectra/job_neospectra_observation_log.json`

```json
{
  "process": {
    "job_folder": "import_data/neospectra/observation_log",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_observation_log_pilot.txt"
  }
}
```

## Process file example

**Path**: `./ai4sh/import_data/neospectra/observation_log/process/ai4sh_dk_foulum_2024@neospectra_ds2_observation_log.json`

```json
{
  "process": [
    {
      "process": "manage_observation_log",
      "parameters": {
        "sampling_log_id__sampling_log_name": "ai4sh_dk_foulum_2024",
        "provision_id__provision_name": "neospectra",
        "name": "ai4sh_dk_foulum_2024@neospectra_ds2",
        "contact_name": "thomas gumbricht",
        "contact_email": "thomas.gumbricht@natgeo.su.se",
        "preparation_id__preparation_name": "ds2",
        "preservation_id__preservation_name": "ds2",
        "home": 1
      }
    }
  ]
}
```

18 observation log process files — one per field site / preparation combination.

## Parameters

| Parameter | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: parent sampling log |
| `provision_id__provision_name` | FK lookup: `neospectra` |
| `name` | Observation log name — pattern: `<sampling_log>@<provision_short>_<preparation>` |
| `contact_name` | Responsible person |
| `contact_email` | Contact email |
| `preparation_id__preparation_name` | FK lookup: sample preparation (e.g. `ds2`) |
| `preservation_id__preservation_name` | FK lookup: preservation method |
| `home` | Integer flag — `1` for non-laboratory (handheld/field) observations |

## Next step

Proceed to [Manage Neospectra observation].

[Manage Neospectra spectrometer]: /spectra/manage_neospectra_spectrometer/
[Load dataset metadata]: /dataset_meta/
[Load utility data]: /utility/
[Manage Neospectra observation]: /spectra/manage_neospectra_obs/
