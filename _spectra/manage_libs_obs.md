---
title: "LIBS Observation"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Inserting LIBS spectral observations into the AI4SH database. Each record stores a 2491-element UV-Vis emission intensity array for one sample shot."
permalink: /spectra/manage_libs_obs/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Each LIBS observation record stores one spectral shot — a 2491-element emission intensity array covering 178–427 nm — linked to a sample and an observation log. LIBS values are raw photon counts (intensity), not reflectance; values are typically in the thousands. Multiple replicates per sample are common.

## Prerequisites

- [Manage LIBS observation log] must be complete.
- Samples must exist from [Load sample data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage LIBS observation** cell runs:

```python
job_file = 'import_data/LIBS_spectra/job_libs_observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/LIBS_spectra/job_libs_observation.json`

```json
{
  "process": {
    "job_folder": "import_data/LIBS_spectra/observation",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_observation_pilot.txt"
  }
}
```

## Process file example

**Path**: `./ai4sh/import_data/LIBS_spectra/observation/process/ai4sh_ch_lens_2026@libs_ds2c_lens@0-5_a_0_none_20260213_observation.json`

```json
{
  "process": [
    {
      "process": "manage_observation",
      "parameters": {
        "observation_log_id__observation_log_name": "ai4sh_ch_lens_2026@libs_ds2c",
        "sample_id__sample_name": "lens@0-5",
        "provision_id__provision_name": "foss micral",
        "subsample": "a",
        "replicate": 0,
        "observation_date": 20260213,
        "@spectra_array": [6339.48, 6235.74, 6255.93, 6381.71]
      }
    }
  ]
}
```

492 process files — one per individual shot. The `@spectra_array` contains 2491 intensity values; the example shows the first 4 (at ~178 nm).

## Parameters

| Parameter | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: parent observation log |
| `sample_id__sample_name` | FK lookup: sample record |
| `provision_id__provision_name` | FK lookup: `foss micral` |
| `subsample` | Sub-sample identifier (e.g. `a`) |
| `replicate` | Shot replicate number (integer, starting at 0) |
| `observation_date` | Date of analysis as integer (`YYYYMMDD`) |
| `@spectra_array` | Emission intensity array — 2491 values, 178–427 nm, ascending wavelength |

Unlike the reflectance instruments, LIBS observations do not include a `provision_serial_nr_id` field.

## All spectral data loaded

All four instruments are now loaded. All spectral observations are stored in the AI4SH database linked to their samples, observation logs, and spectrometer records. To load all data types in a single notebook, see [Load all data].

[Manage LIBS observation log]: /spectra/manage_libs_obs_log/
[Load sample data]: /sample/
[Load all data]: /all_data/
