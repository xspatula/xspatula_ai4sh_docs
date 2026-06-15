---
title: "FOSS DS2500 Observation Log"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Inserting FOSS DS2500 observation log records into the AI4SH database. Each record links a sampling log to the DS2500 provision and records sample handling metadata."
permalink: /spectra/manage_foss_ds2500_obs_log/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

An observation log record links a sampling log to the FOSS DS2500 provision, recording how samples were prepared and stored before scanning. One observation log is created per sampling-log/preparation combination. Individual spectral scans reference these log records.

## Prerequisites

- [Manage FOSS DS2500 spectrometer] must be complete.
- Sampling logs must exist from [Load dataset metadata].
- Preparation, preservation, and storage utility records must exist from [Load utility data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage FOSS DS2500 observation log** cell runs:

```python
job_file = 'import_data/DS2500_spectra/job_DS2500_observation_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/DS2500_spectra/job_DS2500_observation_log.json`

```json
{
  "process": {
    "job_folder": "import_data/DS2500_spectra/observation_log",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_observation_log_pilot.txt"
  }
}
```

## Pilot file

**Path**: `./ai4sh/import_data/DS2500_spectra/observation_log/xspatula_add_observation_log_pilot.txt`

```
ai4sh_dk_foulum_2024@foss ds2500 l_ds2_observation_log.json
ai4sh_dk_foulum_2024@foss ds2500 l_ilmx_observation_log.json
ai4sh_fi_jokioinen_2024@foss ds2500 l_ds2_observation_log.json
...
```

14 entries — one per field site / preparation combination.

## Process file example

**Path**: `./ai4sh/import_data/DS2500_spectra/observation_log/process/ai4sh_dk_foulum_2024@foss ds2500 l_ds2_observation_log.json`

```json
{
  "process": [
    {
      "process": "manage_observation_log",
      "parameters": {
        "sampling_log_id__sampling_log_name": "ai4sh_dk_foulum_2024",
        "provision_id__provision_name": "foss ds2500 l",
        "name": "ai4sh_dk_foulum_2024@foss ds2500 l_ds2",
        "contact_name": "thomas gumbricht",
        "contact_email": "thomas.gumbricht@natgeo.su.se",
        "preparation_id__preparation_name": "ds2",
        "preservation_id__preservation_name": "ds2",
        "storage_id__storage_name": "amb",
        "laboratory": 1
      }
    }
  ]
}
```

## Parameters

| Parameter | Description |
|---|---|
| `sampling_log_id__sampling_log_name` | FK lookup: parent sampling log |
| `provision_id__provision_name` | FK lookup: `foss ds2500 l` |
| `name` | Observation log name — pattern: `<sampling_log>@<provision_short>_<preparation>` |
| `contact_name` | Responsible person |
| `contact_email` | Contact email |
| `preparation_id__preparation_name` | FK lookup: sample preparation (e.g. `ds2` = dried and sieved 2 mm) |
| `preservation_id__preservation_name` | FK lookup: preservation method |
| `storage_id__storage_name` | FK lookup: storage condition (e.g. `amb` = ambient) |
| `laboratory` | Integer flag — `1` for laboratory-based observations |

## Next step

Proceed to [Manage FOSS DS2500 observation].

[Manage FOSS DS2500 spectrometer]: /spectra/manage_foss_ds2500_spectrometer/
[Load dataset metadata]: /dataset_meta/
[Load utility data]: /utility/
[Manage FOSS DS2500 observation]: /spectra/manage_foss_ds2500_obs/
