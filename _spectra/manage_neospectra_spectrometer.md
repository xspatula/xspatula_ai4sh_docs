---
title: "Neospectra Spectrometer"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Summary of the Si-Ware Neospectra NIR spectrometer and its data range. Required before loading Neospectra observation logs or observations."
permalink: /spectra/manage_neospectra_spectrometer/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-10-16 08:00:00 +0200
---

The Si-Ware Neospectra is a handheld near-infrared (NIR) reflectance spectrometer covering 1350–2550 nm at approximately 2.5 nm resolution, producing 257-element spectral arrays. Unlike the benchtop instruments, Neospectra is used both in the field and in the laboratory. Its observation logs use `home: 1` rather than `laboratory: 1` to reflect this.

## Prerequisites

The `neospectra` provision must exist from [Load utility data].

## Notebook cell

In `./ai4sh/import_data/insert_ai4sh_data.ipynb`, block **3.3 AI4SH spectral data**, cell **3.3.4 Load Neospectra spectrometer** runs:

```python
job_file = 'import_data/neospectra/job_neospectra_spectrometer.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Job file

**Path**: `./ai4sh/import_data/neospectra/job_neospectra_spectrometer.json`

```json
{
  "process": {
    "job_folder": "import_data/neospectra/spectrometer",
    "process_sub_folder": "manage_process",
    "pilot_file": "xspatula_add_spectrometer_pilot.txt"
  }
}
```

## Pilot file

**Path**: `./ai4sh/import_data/neospectra/spectrometer/xspatula_add_spectrometer_pilot.txt`

```
neospectra_spectrometer.json
```

One entry — one spectrometer record.

## Process file

**Path**: `./ai4sh/import_data/neospectra/spectrometer/manage_process/neospectra_spectrometer.json`

```json
{
  "process": [
    {
      "process": "manage_spectrometer",
      "parameters": {
        "provision_id__provision_name": "neospectra",
        "wavelength_unit_id__wavelength_unit_name": "um",
        "wavelength_array": [1350.0, 1352.5, 1355.0, 1357.5, "...257 values total...", 2549.9]
      }
    }
  ]
}
```

The `wavelength_array` contains 257 values from ~1350 to ~2550 nm at ~2.5 nm steps. The example above is abbreviated.

## Instrument summary

| Property | Value |
|---|---|
| Provision | `neospectra` |
| Wavelength range | ~1350–2550 nm |
| Spectral resolution | ~2.5 nm |
| Number of bands | 257 |
| Measurement type | Reflectance |
| Form factor | Handheld |
| Use context | Field and laboratory |

## Next step

Proceed to [Manage Neospectra observation log].

[Load utility data]: {{ site.baseurl }}/utility/
[Manage Neospectra observation log]: {{ site.baseurl }}/spectra/manage_neospectra_obs_log/
