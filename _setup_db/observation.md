---
title: "Observation Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The observation schema stores actual soil property data, organised hierarchically through data source, datasets, campaigns, sampling logs, samples and observations. All observation records depend on the observation_utility reference catalogues."
permalink: /setup_db/observation/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The `observation` schema stores actual soil property data. Data is organised in a strict hierarchy: an observation belongs to a sample, a sample belongs to a sampling log, a sampling log belongs to a campaign, a campaign belongs to a dataset. Every step in this chain must exist before the next can be created. All observation values link to reference records in `observation_utility`.

## Process files

| File | Tables created | Description |
|---|---|---|
| `observation/data_source_v10_sql.json` | `data_source` | Simplified copy of `community.organisation` — passive data suppliers who need not be full system users |
| `observation/person_v10_sql.json` | `person` | Simplified copy of `community.user` with no login credentials — allows attributing data to non-registered persons (GDPR consideration) |
| `observation/dataset_v10_sql.json` | `dataset` | Top-level grouping of related data (e.g. LUCAS, AI4SoilHealth) |
| `observation/campaign_v10_sql.json` | `campaign`, `campaign_meta`, `campaign_method_tier`, `campaign_location`, `campaign_provision`, `campaign_setting` | Child of dataset; captures details of a specific data collection effort |
| `observation/sampling_log_v10_sql.json` | `sampling_log` | A sampling event — one or multiple samples taken with consistent methods over a period |
| `observation/sample_geolocation_v10_sql.json` | `sample_geolocation` | Spatial coordinates for a geolocated sample |
| `observation/sample_v10_sql.json` | `sample` | An individual sample acquired under a sampling log |
| `observation/sample_z_profile_v10_sql.json` | `sample_profile` | Depth profile interval (profile_min, profile_max) for samples with a z-dimension |
| `observation/sample_image_v10_sql.json` | `sample_image` | Image records associated with samples |
| `observation/observation_log_v10_sql.json` | `observation_log` | Links a sampling log to a provision; records the logistics (preservation, storage, transportation) of getting the sample to analysis |
| `observation/observation_v10_sql.json` | `observation` | The actual measured value for a specific indicator from a specific provision |
| `observation/spectra_v10_sql.json` | `spectra_scan` | Spectral scan metadata — signal statistics, spectroscopy method, and quality flags linked to an observation |
| `observation/measurement_v10_sql.json` | `measurement` | General sensor or instrument measurements |
| `observation/microbiometer_measurement_v10_sql.json` | `microbiometer_measurement` | Microbiometer carbon content measurements linked to an observation |
| `observation/infiltration_beerkan_v10_sql.json` | `infiltration_beerkan` | BeerKan ring infiltration measurements |
| `observation/slakes_v10_sql.json` | `app_aggregate_stability` | Aggregate stability measurements from the SLAKES app |
| `observation/macrofauna_v10_sql.json` | `macrofauna` | Macrofauna count and biomass observations |
| `observation/macrofauna_image_v10_sql.json` | `macrofauna_image` | Image records for automated macrofauna detection |

## Table hierarchy

```
data_source
dataset (→ data_source)
  campaign (→ dataset)
    campaign_meta (→ campaign, observation_utility.*)
    campaign_method_tier (→ campaign)
    campaign_location (→ campaign, utility.territory, observation_utility.spatial_reference)
    campaign_provision (→ campaign, observation_utility.provision)
    campaign_setting (→ campaign, observation_utility.setting_system)
    sampling_log (→ campaign)
      sample_geolocation (→ sampling_log, observation_utility.spatial_reference)
      sample (→ sampling_log)
        observation_log (→ sampling_log, observation_utility.provision)
          observation (→ observation_log, observation_utility.provision_indicator)
```

## Key tables in detail

### dataset

A dataset is a coherent collection of data from one or more campaigns — for example, LUCAS 2009–2022 or the AI4SoilHealth project. Datasets are associated with a `data_source`.

Key columns: `id`, `data_source_id`, `name`, `alias`, `status_code`.

### campaign

A campaign is a child of a dataset where sampling methods may vary in detail. For example, LUCAS 2009 and LUCAS 2015 are two campaigns within the LUCAS dataset. Each AI4SH pilot site is a separate campaign.

The `campaign` table has five companion meta tables:

- **campaign_meta** — abstract, DOI, URL, keywords, time series flag, license, taxonomic scope
- **campaign_method_tier** — boolean flags for which method tiers (in-situ, in-home, in-lab, from-drone, from-satellite, etc.) are used in the campaign
- **campaign_location** — geographic bounding box and spatial reference
- **campaign_provision** — which provisions (instrument+provider+method_tier combinations) are used
- **campaign_setting** — which setting systems (field, forest, lake, etc.) apply

### sampling_log

A sampling log is a sampling event within a campaign, covering one or multiple samples taken with consistent methods. It records the responsible person and time window.

### sample and sample_geolocation

`sample` represents an individual physical sample. `sample_geolocation` optionally records its latitude, longitude, elevation, and spatial reference. Decoupled from `sample` so that non-geolocated samples are still representable.

### observation_log and observation

`observation_log` bridges a sampling log to a provision (instrument + service + method tier) and records the sample handling chain (preservation, storage, transportation method). `observation` stores the actual numeric or text result for a specific `provision_indicator` (a measurable value from a specific provision).

### Specialised observation tables

Three additional tables cover non-standard observation types:

- **infiltration_beerkan** — BeerKan ring infiltration test results
- **macrofauna** and **macrofauna_image** — macrofauna counts, biomass, and associated image records
- **measurement** — direct sensor readings (e.g. from a handheld spectral sensor)
