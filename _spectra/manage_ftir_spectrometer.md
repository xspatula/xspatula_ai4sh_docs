---
title: "FTIR Spectrometer"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Summary of the Bruker Invenio S FTIR spectrometer and its data range. Required before loading FTIR observation logs or observations."
permalink: /spectra/manage_ftir_spectrometer/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

The Bruker Invenio S is a benchtop Fourier Transform Infrared (FTIR) spectrometer operating in the mid-infrared (MIR) range. It covers approximately 2501–16686 nm (4000–600 cm⁻¹) at ~0.9 nm resolution, producing 2383-element spectral arrays stored in ascending wavelength order. FTIR requires milled samples (`ds2m` preparation) for optimal performance.

## Prerequisites

The `bruker invenio s ftir` provision and its `provision_serial_nr` must exist from [Load utility data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage FTIR spectrometer** cell runs:

```python
job_file = 'import_data/FTIR_spectra/job_ftir_spectrometer.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/FTIR_spectra/job_ftir_spectrometer.json`

```json
{
  "process": {
    "job_folder": "import_data/FTIR_spectra/spectrometer",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_spectrometer_pilot.txt"
  }
}
```

## Process file

**Path**: `./ai4sh/import_data/FTIR_spectra/spectrometer/process/bruker-invenio-s-ftir_spectrometer.json`

```json
{
  "process": [
    {
      "process": "manage_spectrometer",
      "parameters": {
        "provision_id__provision_name": "bruker invenio s ftir",
        "wavelength_unit_id__wavelength_unit_name": "um",
        "wavelength_array": [2501.1, 2502.0, 2502.9, 2503.8, "...2383 values total...", 16685.8]
      }
    }
  ]
}
```

The `wavelength_array` contains 2383 values from 2501.1 to 16685.8 nm in ~0.9 nm steps. The example above is abbreviated.

## Instrument summary

| Property | Value |
|---|---|
| Provision | `bruker invenio s ftir` |
| Wavelength range | 2501–16686 nm (4000–600 cm⁻¹) |
| Spectral resolution | ~0.9 nm |
| Number of bands | 2383 |
| Measurement type | Reflectance (MIR) |
| Required preparation | Milled (`ds2m`) |
| Use context | Laboratory |

## Next step

Proceed to [Manage FTIR observation log].

[Load utility data]: /utility/
[Manage FTIR observation log]: /spectra/manage_ftir_obs_log/
