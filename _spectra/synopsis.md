---
title: "Load Spectral Data"
layout: single
sidebar:
  nav: "spectra"
  nav2: "loading_data"
excerpt: "Overview of spectral data in the AI4SH database. All data is stored as ascending-wavelength arrays. Covers FOSS DS2500, Neospectra, FTIR, and LIBS instruments."
permalink: /spectra/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Spectral data in the AI4SH database covers four instrument types spanning ultraviolet to mid-infrared wavelengths. All spectral arrays are stored in ascending wavelength order. Each instrument requires three loading steps: register the spectrometer (wavelength axis), create observation logs (linking sampling logs to the instrument provision), and insert observations (the spectral arrays tied to individual samples).

All spectral loading is driven by:

```
./ai4sh/import_data/load_ai4sh_spectral_data.ipynb
```

## Four instruments

| Instrument | Provision | Range | Bands | Type |
|---|---|---|---|---|
| FOSS DS2500 | `foss ds2500 l` | 400–2500 nm | 4200 | Vis-NIR reflectance |
| Si-Ware Neospectra | `neospectra` | 1350–2550 nm | 257 | NIR reflectance (handheld) |
| Bruker Invenio S | `bruker invenio s ftir` | 2501–16686 nm | 2383 | MIR reflectance (FTIR) |
| FOSS Micral | `foss micral` | 178–427 nm | 2491 | UV-Vis emission intensity (LIBS) |

## Required loading sequence per instrument

For each of the four instruments, load in this order:

1. **Manage spectrometer** — register the instrument and its wavelength axis
2. **Manage observation log** — link sampling logs to the instrument provision
3. **Manage observation** — insert spectral arrays linked to individual samples

All cells use `job_file`, not `process_file`.

## Full loading sequence

### FOSS DS2500
1. [Manage FOSS DS2500 spectrometer]
2. [Manage FOSS DS2500 observation log]
3. [Manage FOSS DS2500 observation]

### Neospectra
4. [Manage Neospectra spectrometer]
5. [Manage Neospectra observation log]
6. [Manage Neospectra observation]

### FTIR
7. [Manage FTIR spectrometer]
8. [Manage FTIR observation log]
9. [Manage FTIR observation]

### LIBS
10. [Manage LIBS spectrometer]
11. [Manage LIBS observation log]
12. [Manage LIBS observation]

## Prerequisites

[Load sample data] must be complete. Observation utility records (provision, provision_indicator, preparation, preservation, storage) must exist from [Load utility data]. FOSS DS2500 and FTIR also require a `provision_serial_nr` record for the instrument serial number.

[Manage FOSS DS2500 spectrometer]: /spectra/manage_foss_ds2500_spectrometer/
[Manage FOSS DS2500 observation log]: /spectra/manage_foss_ds2500_obs_log/
[Manage FOSS DS2500 observation]: /spectra/manage_foss_ds2500_obs/
[Manage Neospectra spectrometer]: /spectra/manage_neospectra_spectrometer/
[Manage Neospectra observation log]: /spectra/manage_neospectra_obs_log/
[Manage Neospectra observation]: /spectra/manage_neospectra_obs/
[Manage FTIR spectrometer]: /spectra/manage_ftir_spectrometer/
[Manage FTIR observation log]: /spectra/manage_ftir_obs_log/
[Manage FTIR observation]: /spectra/manage_ftir_obs/
[Manage LIBS spectrometer]: /spectra/manage_libs_spectrometer/
[Manage LIBS observation log]: /spectra/manage_libs_obs_log/
[Manage LIBS observation]: /spectra/manage_libs_obs/
[Load sample data]: /sample/
[Load utility data]: /utility/
