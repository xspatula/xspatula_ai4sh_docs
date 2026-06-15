---
title: "FTIR Observation"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Inserting FTIR spectral observations into the AI4SH database. Each record stores a 2383-element MIR reflectance array for one sample scan."
permalink: /spectra/manage_ftir_obs/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Each FTIR observation record stores one spectral scan — a 2383-element reflectance array covering 2501–16686 nm — linked to a sample and an observation log. The spectral array is stored in ascending wavelength order.

## Prerequisites

- [Manage FTIR observation log] must be complete.
- Samples must exist from [Load sample data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage FTIR observation** cell runs:

```python
job_file = 'import_data/FTIR_spectra/job_ftir_observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/FTIR_spectra/job_ftir_observation.json`

```json
{
  "process": {
    "job_folder": "import_data/FTIR_spectra/observation",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_observation_pilot.txt"
  }
}
```

## Process file example

**Path**: `./ai4sh/import_data/FTIR_spectra/observation/process/ai4sh_ch_lens_2026@ftir_ds2m_lens@0-5_a_1_none_20260513_observation.json`

```json
{
  "process": [
    {
      "process": "manage_observation",
      "parameters": {
        "observation_log_id__observation_log_name": "ai4sh_ch_lens_2026@ftir_ds2m",
        "sample_id__sample_name": "lens@0-5",
        "provision_id__provision_name": "bruker invenio s ftir",
        "subsample": "a",
        "replicate": 1,
        "observation_date": 20260513,
        "provision_serial_nr_id__provision_serial_nr_name": "au",
        "@spectra_array": [0.6864, 0.6864, 0.6863, 0.6863]
      }
    }
  ]
}
```

1419 process files — one per individual scan. The `@spectra_array` contains 2383 reflectance values; the example shows the first 4 (at ~2501 nm).

## Parameters

| Parameter | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: parent observation log |
| `sample_id__sample_name` | FK lookup: sample record |
| `provision_id__provision_name` | FK lookup: `bruker invenio s ftir` |
| `subsample` | Sub-sample identifier (e.g. `a`) |
| `replicate` | Scan replicate number (integer) |
| `observation_date` | Date of scanning as integer (`YYYYMMDD`) |
| `provision_serial_nr_id__provision_serial_nr_name` | FK lookup: instrument serial number |
| `@spectra_array` | Reflectance array — 2383 values, 2501–16686 nm, ascending wavelength |

## Next step

Proceed to [Manage LIBS spectrometer] to continue with the final instrument.

[Manage FTIR observation log]: /spectra/manage_ftir_obs_log/
[Load sample data]: /sample/
[Manage LIBS spectrometer]: /spectra/manage_libs_spectrometer/
