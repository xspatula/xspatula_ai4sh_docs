---
title: "FOSS DS2500 Spectrometer"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Summary of the FOSS DS2500 NIR spectrometer and its data range. Required before loading DS2500 observation logs or observations."
permalink: /spectra/manage_foss_ds2500_spectrometer/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

The FOSS DS2500 is a benchtop visible and near-infrared (Vis-NIR) reflectance spectrometer. It covers 400–2500 nm at 0.5 nm resolution, producing 4200-element spectral arrays. In the AI4SH project it is used for soil analysis on dried and sieved samples in the laboratory.

The spectrometer record registers the instrument provision and its wavelength axis in the database. It must be inserted before any observation log or observation records can reference it.

## Prerequisites

The `foss ds2500 l` provision and its `provision_serial_nr` (`au`) must exist from [Load utility data].

## Notebook cell

In `load_ai4sh_spectral_data.ipynb`, the **Manage FOSS DS2500 spectrometer** cell runs:

```python
job_file = 'import_data/DS2500_spectra/job_DS2500_spectrometer.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/DS2500_spectra/job_DS2500_spectrometer.json`

```json
{
  "process": {
    "job_folder": "import_data/DS2500_spectra/spectrometer",
    "process_sub_folder": "process",
    "pilot_file": "xspatula_add_spectrometer_pilot.txt"
  }
}
```

## Pilot file

**Path**: `./ai4sh/import_data/DS2500_spectra/spectrometer/xspatula_add_spectrometer_pilot.txt`

```
foss-ds2500-l_au_spectrometer.json
```

One entry — one spectrometer record.

## Process file

**Path**: `./ai4sh/import_data/DS2500_spectra/spectrometer/process/foss-ds2500-l_au_spectrometer.json`

```json
{
  "process": [
    {
      "process": "manage_spectrometer",
      "parameters": {
        "provision_id__provision_name": "foss ds2500 l",
        "wavelength_unit_id__wavelength_unit_name": "um",
        "provision_serial_nr_id__provision_serial_nr_name": "au",
        "wavelength_array": [400.0, 400.5, 401.0, 401.5, "...4200 values total...", 2499.5]
      }
    }
  ]
}
```

The `wavelength_array` contains 4200 values from 400.0 to 2499.5 nm in 0.5 nm steps. The example above is abbreviated; the actual file contains the full array.

## Instrument summary

| Property | Value |
|---|---|
| Provision | `foss ds2500 l` |
| Serial number | `au` |
| Wavelength range | 400–2500 nm |
| Spectral resolution | 0.5 nm |
| Number of bands | 4200 |
| Measurement type | Reflectance |
| Use context | Laboratory (dried and sieved samples) |

## Next step

Proceed to [Manage FOSS DS2500 observation log].

[Load utility data]: /utility/
[Manage FOSS DS2500 observation log]: /spectra/manage_foss_ds2500_obs_log/
