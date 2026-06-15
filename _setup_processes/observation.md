---
title: "Observation Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Observation processes register all data management operations for the observation schema — data sources, datasets, campaigns, sampling logs, samples, observations and specialised measurement types."
permalink: /setup_process/observation/
author_profile: false
date: 2026-04-07 08:00:00 +0200
last_modified_at: 2026-04-07 08:00:00 +0200
---

Observation processes register the operations for managing data in the `observation` schema. These processes follow the same data hierarchy as the tables themselves: you must be able to manage datasets before campaigns, campaigns before sampling logs, and so on down to individual observations.

## Process files

Process files are located at:

```
./setup/zzz/ai4sh/setup_processes/json_ai4sh/observation/
```

| File | Process registered | Target table(s) | Min stratum |
|---|---|---|---|
| `data_source_v10_sql.json` | `manage_data_source` | `observation.data_source` | 2 |
| `person_v10_sql.json` | `manage_person` | `observation.person` | 1 |
| `dataset_v10_sql.json` | `manage_dataset` | `observation.dataset` | 3 |
| `campaign_v10_sql.json` | `manage_campaign` | `observation.campaign` + companion tables | 3 |
| `sample_geolocation_v10_sql.json` | `manage_geolocation` | `observation.sample_geolocation` | 3 |
| `sampling_log_v10_sql.json` | `manage_sampling_log` | `observation.sampling_log` | 2 |
| `sample_v10_sql.json` | `manage_sample` | `observation.sample` | 4 |
| `sample_image_v10_sql.json` | `manage_sample_image_orientation` | `observation.macrofauna_image` | 4 |
| `observation_log_v10_sql.json` | `manage_observation_log` | `observation.observation_log` | 2 |
| `observation_v10_sql.json` | `manage_observation` | `observation.observation` | 2 |
| `quantity_v10_sql.json` | `manage_quantity` | `observation.observation` (quantity-specific) | 4 |
| `measurement_v10_sql.json` | `manage_measurement_array` / `manage_measurement` | `observation.measurement` | 2 / 4 |
| `beerkan_infiltration_v10_sql.json` | `manage_beerkan_observation` / `manage_beerkan_infiltration_interval` | `observation.infiltration_beerkan` | 2 |
| `slakes_v10_sql.json` | `manage_beerkan_observation` | `observation.app_aggregate_stability` | 2 |
| `aggregate_app_observation_v10_sql.json` | `manage_aggregate_app_observation` | `observation.observation` (aggregate) | 2 |

## Access levels

Access levels reflect the operational role of each process:

- Stratum 1: `manage_person` — minimal privilege for attributing data to individuals
- Stratum 2: `manage_data_source`, `manage_sampling_log`, `manage_observation_log`, `manage_observation`, `manage_measurement_array`, `manage_beerkan_observation`, `manage_beerkan_infiltration_interval`, `manage_aggregate_app_observation` — regular data entry operations
- Stratum 3: `manage_dataset`, `manage_campaign`, `manage_geolocation` — campaign-level management
- Stratum 4: `manage_sample`, `manage_sample_image_orientation`, `manage_quantity`, `manage_measurement` — elevated operations requiring administrative oversight

## Key processes in detail

### manage_data_source

Registers an external organisation or data supplier as a data source. This is a simplified record for attributing datasets to their origin — separate from the full `community.organisation` user record. Required parameters:

- `name` — unique data source name
- `url` — URL of the data source

Optional: `alias`, `address1`, `address2`, `postal_address`, `postal_zip_code`, `state`, `territory_id__territory_name`, `telephone`, `contact_name`, `contact_email`.

### manage_dataset

A dataset is the top-level grouping of related data. Required parameters:

- `data_source_id__source_name` — the data source owning the dataset
- `name` — unique dataset name
- `alias` — short identifier (e.g. "AI4SH", "LUCAS")

### manage_campaign

Registers a campaign (child of a dataset) and its metadata. The `manage_campaign` process handles the core `campaign` table. Companion meta tables (`campaign_meta`, `campaign_method_tier`, `campaign_location`, `campaign_provision`, `campaign_setting`) may have separate processes or be populated as part of the campaign registration.

Required parameters include:
- `dataset_id__dataset_name` — the parent dataset
- `name` — campaign name
- `contact_name`, `contact_email`

### manage_sampling_log

Registers a sampling event within a campaign. A sampling log is the bridge between the organisational structure (campaign) and individual physical samples.

Required parameters: `campaign_id`, `person_id`, start and end date, and method references.

### manage_sample

Registers an individual physical sample. Links to a `sampling_log` and optionally to a `sample_geolocation` for spatial referencing.

### manage_observation_log and manage_observation

`manage_observation_log` registers the connection between a sample and the provision used to analyse it, along with sample handling logistics (preservation, storage, transportation). `manage_observation` then stores the actual measured value for a specific `provision_indicator`.

The key parameter in `manage_observation` is the indicator value itself — a numeric or text result that must be consistent with the unit defined in the `provision_indicator`.

### manage_geolocation

Registers spatial coordinates for a sample:

- `latitude_dd_wgs84` — decimal degrees latitude
- `longitude_dd_wgs84` — decimal degrees longitude
- `elevation` — optional elevation in the specified unit
- `spatial_reference_id` — FK to `observation_utility.spatial_reference`

### manage_measurement

Registers a direct instrument reading (e.g. from a handheld spectral sensor). Two variants are available:

- `manage_measurement_array` (stratum 2) — bulk insert of an array of measurements in a single call
- `manage_measurement` (stratum 4) — insert a single measurement record with administrative oversight

Both are lighter-weight observation types for sensor output that do not require the full observation_log chain.

## Data entry sequence

To enter a complete soil observation, the following sequence must be followed:

1. `manage_data_source` (if the data source is new)
2. `manage_dataset`
3. `manage_campaign`
4. `manage_sampling_log`
5. `manage_sample` (optionally with `manage_geolocation`)
6. `manage_observation_log`
7. `manage_observation`

If any step in this chain is missing, the foreign key constraints prevent entry of the subsequent records. This enforced chain is what ensures FAIR data compliance — every observation can be traced back to a fully documented campaign and sampling event.
