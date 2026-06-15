---
title: "Manage Agrolab Observation"
layout: single
sidebar:
  nav: "wetlab"
  nav2: "loading_data"
excerpt: "Inserting Agrolab wetlab observation records into the AI4SH database. Each record links a sample to a set of measured soil properties via an observation log."
permalink: /wetlab/manage_ai4sh_wetlab_data/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Observation records store the actual soil property measurements. Each record links a sample to an observation log and carries the measured values as `@`-prefixed fields that map to provision indicators. One row in the source Excel file produces one observation record per sample.

## Prerequisites

- [Manage Agrolab observation log] must be complete.
- Samples must exist from [Load sample data].

## Notebook cell

In `load_ai4sh_wetlab_data.ipynb`, the **Manage Agrolab observation** cell runs:

```python
process_file = 'import_data/wetlab/agrolab/manage_process/manage_observation.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/wetlab/agrolab/manage_process/manage_observation.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_observation",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "observation_log_id__observation_log_name": "ai4sh_dk_foulum_2024@ai4sh-agrolab",
        "sample_id__sample_name": "366-2-c5@0-20",
        "observation_date": 20250201,
        "@ph-h2o": "5.9",
        "@ec": 32,
        "@c-org": 2.18,
        "@n-tot": 0.17,
        "@ca2+": 6.7,
        "@mg2+": 0.4,
        "@k+": "<0.10",
        "@na+": "0.2",
        "@cec": 19.0,
        "@p-ext": "5.4",
        "@clay": 11.0,
        "@fine silt": 7.0,
        "@coarse silt": 7.0,
        "@fine sand": 51,
        "@coarse sand": 24.0
      }
    }
  ]
}
```

The example file contains 347 entries — one per sample with Agrolab results.

## Parameters

| Parameter | Description |
|---|---|
| `observation_log_id__observation_log_name` | FK lookup: parent observation log (format: `<sampling_log>@<provision>`) |
| `sample_id__sample_name` | FK lookup: sample record (format: `<tag>@<profile_min>-<profile_max>`) |
| `observation_date` | Date of laboratory analysis as integer (`YYYYMMDD`) |
| `@<indicator>` | Measured value for the named provision indicator (see below) |

## The `@` prefix for observation values

Fields whose names begin with `@` are observation values. The name after the `@` must match a `provision_indicator` registered under the observation log's provision (`ai4sh-agrolab`). The framework resolves each `@field` to the corresponding indicator FK at insert time.

Values may be numeric (`2.18`) or string (`"<0.10"` for below-detection). The AI4SH Agrolab provision measures 16 standard indicators:

| Indicator | Description |
|---|---|
| `ph-h2o` | pH in water |
| `ec` | Electrical conductivity (μS/cm) |
| `c-org` | Organic carbon (%) |
| `n-tot` | Total nitrogen (%) |
| `ca2+` | Exchangeable calcium (cmol/kg) |
| `mg2+` | Exchangeable magnesium (cmol/kg) |
| `k+` | Exchangeable potassium (cmol/kg) |
| `na+` | Exchangeable sodium (cmol/kg) |
| `cec` | Cation exchange capacity (cmol/kg) |
| `p-ext` | Extractable phosphorus (mg/kg) |
| `clay` | Clay fraction (%) |
| `fine silt` | Fine silt fraction (%) |
| `coarse silt` | Coarse silt fraction (%) |
| `fine sand` | Fine sand fraction (%) |
| `coarse sand` | Coarse sand fraction (%) |

## Sample name format

The `sample_id__sample_name` FK uses the pattern `<tag>@<profile_min>-<profile_max>` (e.g. `366-2-c5@0-20`). This matches the name generated when samples were inserted during [Load sample data].

## Next step

All wetlab data is now loaded. Proceed to [Load spectral data] to add spectroscopic observations.

[Manage Agrolab observation log]: /wetlab/manage_ai4sh_agrolab_obs_log/
[Load sample data]: /sample/
[Load spectral data]: /spectra/
