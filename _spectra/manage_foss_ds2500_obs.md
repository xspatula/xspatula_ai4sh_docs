---
title: "FOSS DS2500 Observation"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Inserting FOSS DS2500 spectral observations into the AI4SH database. Each record stores a 4200-element reflectance array for one sample scan."
permalink: /spectra/manage_foss_ds2500_obs/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Each observation record stores one spectral scan — a 4200-element reflectance array covering 400–2500 nm — linked to a sample and an observation log. Multiple sub-samples and replicates per sample are common: each is a separate process file and a separate database record.

## Prerequisites

- [Manage FOSS DS2500 observation log] must be complete.
- Samples must exist from [Load sample data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage FOSS DS2500 observation** cell runs:

```python
job_file = 'import_data/DS2500_spectra/job_DS2500_observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/DS2500_spectra/job_DS2500_observation.json`

```json
{
  "process": {
    "job_folder": "import_data/DS2500_spectra/observation",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_observation_pilot.txt"
  }
}
```

## Pilot file

**Path**: `./ai4sh/import_data/DS2500_spectra/observation/xspatula_add_observation_pilot.txt`

1517 entries — one process file per individual scan.

## Process file example

**Path**: `./ai4sh/import_data/DS2500_spectra/observation/process/ai4sh_ch_lens_2025@foss ds2500 l_ds2_lens@0-5_a_1_none_20251023_observation.json`

```json
{
  "process": [
    {
      "process": "manage_observation",
      "parameters": {
        "observation_log_id__observation_log_name": "ai4sh_ch_lens_2025@foss ds2500 l_ds2",
        "sample_id__sample_name": "lens@0-5",
        "provision_id__provision_name": "foss ds2500 l",
        "subsample": "a",
        "replicate": 1,
        "observation_date": 20251023,
        "provision_serial_nr_id__provision_serial_nr_name": "au",
        "@spectra_array": [0.3396, 0.3396, 0.3396, 0.3362]
      }
    }
  ]
}
```

The `@spectra_array` contains 4200 reflectance values. The example above shows the first 4.

## Parameters

| Parameter | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: parent observation log |
| `sample_id__sample_name` | FK lookup: sample record (format: `<tag>@<profile_min>-<profile_max>`) |
| `provision_id__provision_name` | FK lookup: `foss ds2500 l` |
| `subsample` | Sub-sample identifier within the sample (e.g. `a`) |
| `replicate` | Scan replicate number (integer) |
| `observation_date` | Date of scanning as integer (`YYYYMMDD`) |
| `provision_serial_nr_id__provision_serial_nr_name` | FK lookup: instrument serial number record (`au`) |
| `@spectra_array` | Reflectance array — 4200 values, 400–2500 nm, ascending wavelength |

## File naming pattern

Process files follow the convention:
```
<obs_log_name>_<sample_tag>_<subsample>_<replicate>_<filter>_<date>_observation.json
```

The `none` segment represents the filter used (no filter in most cases).

## Next step

Proceed to [Manage Neospectra spectrometer] to continue with the next instrument.

[Manage FOSS DS2500 observation log]: /spectra/manage_foss_ds2500_obs_log/
[Load sample data]: /sample/
[Manage Neospectra spectrometer]: /spectra/manage_neospectra_spectrometer/
