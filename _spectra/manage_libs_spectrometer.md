---
title: "LIBS Spectrometer"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Summary of the FOSS Micral LIBS spectrometer and its data range. Required before loading LIBS observation logs or observations."
permalink: /spectra/manage_libs_spectrometer/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

The FOSS Micral is a Laser-Induced Breakdown Spectroscopy (LIBS) instrument. Unlike the reflectance spectrometers, LIBS measures elemental emission intensities in the ultraviolet and visible range. It covers 178–427 nm at 0.1 nm resolution, producing 2491-element arrays of raw emission intensity values (not reflectance). LIBS ablates the sample surface with a laser pulse and records the resulting atomic emission spectrum.

## Prerequisites

The `foss micral` provision must exist from [Load utility data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage LIBS spectrometer** cell runs:

```python
job_file = 'import_data/LIBS_spectra/job_libs_spectrometer.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/LIBS_spectra/job_libs_spectrometer.json`

```json
{
  "process": {
    "job_folder": "import_data/LIBS_spectra/spectrometer",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_spectrometer_pilot.txt"
  }
}
```

## Process file

**Path**: `./ai4sh/import_data/LIBS_spectra/spectrometer/process/foss-micral_spectrometer.json`

```json
{
  "process": [
    {
      "process": "manage_spectrometer",
      "parameters": {
        "provision_id__provision_name": "foss micral",
        "wavelength_unit_id__wavelength_unit_name": "um",
        "wavelength_array": [178.0, 178.1, 178.2, 178.3, "...2491 values total...", 427.0]
      }
    }
  ]
}
```

The `wavelength_array` contains 2491 values from 178.0 to 427.0 nm in 0.1 nm steps. The example above is abbreviated.

## Instrument summary

| Property | Value |
|---|---|
| Provision | `foss micral` |
| Wavelength range | 178–427 nm |
| Spectral resolution | 0.1 nm |
| Number of bands | 2491 |
| Measurement type | Emission intensity (not reflectance) |
| Use context | Laboratory |

## Note on stored values

LIBS observations store emission intensity counts, not reflectance values in the 0–1 range. Intensity values are typically in the thousands. This is different from all other spectrometers in the AI4SH database, which store reflectance.

## Next step

Proceed to [Manage LIBS observation log].

[Load utility data]: /utility/
[Manage LIBS observation log]: /spectra/manage_libs_obs_log/
