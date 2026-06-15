---
title: "Neospectra Observation"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Inserting Neospectra spectral observations into the AI4SH database. Each record stores a 257-element NIR reflectance array for one sample scan."
permalink: /spectra/manage_neospectra_obs/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Each Neospectra observation record stores one spectral scan — a 257-element reflectance array covering ~1350–2550 nm — linked to a sample and an observation log. Multiple sub-samples and replicates are common; each is a separate process file.

## Prerequisites

- [Manage Neospectra observation log] must be complete.
- Samples must exist from [Load sample data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage Neospectra observation** cell runs:

```python
job_file = 'import_data/neospectra/job_neospectra_observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/neospectra/job_neospectra_observation.json`

```json
{
  "process": {
    "job_folder": "import_data/neospectra/observation",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_observation_pilot.txt"
  }
}
```

## Process file example

**Path**: `./ai4sh/import_data/neospectra/observation/process/ai4sh_dk_foulum_2024@neospectra_ds2_219-c5@0-20_a_0_none_20241106_observation.json`

```json
{
  "process": [
    {
      "process": "manage_observation",
      "parameters": {
        "observation_log_id__observation_log_name": "ai4sh_dk_foulum_2024@neospectra_ds2",
        "sample_id__sample_name": "219-c5@0-20",
        "provision_id__provision_name": "neospectra",
        "subsample": "a",
        "replicate": 0,
        "observation_date": 20241106,
        "@spectra_array": [0.2274, 0.2284, 0.2295, 0.2305]
      }
    }
  ]
}
```

1672 process files — one per individual scan. The `@spectra_array` contains 257 reflectance values; the example shows the first 4.

## Parameters

| Parameter | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: parent observation log |
| `sample_id__sample_name` | FK lookup: sample record |
| `provision_id__provision_name` | FK lookup: `neospectra` |
| `subsample` | Sub-sample identifier (e.g. `a`) |
| `replicate` | Scan replicate number (integer, starting at 0) |
| `observation_date` | Date of scanning as integer (`YYYYMMDD`) |
| `@spectra_array` | Reflectance array — 257 values, ~1350–2550 nm, ascending wavelength |

Unlike the FOSS DS2500, the Neospectra observation does not include a `provision_serial_nr_id` field — serial number tracking is optional and not used in the current AI4SH dataset for this instrument.

## Next step

Proceed to [Manage FTIR spectrometer] to continue with the FTIR instrument.

[Manage Neospectra observation log]: /spectra/manage_neospectra_obs_log/
[Load sample data]: /sample/
[Manage FTIR spectrometer]: /spectra/manage_ftir_spectrometer/
