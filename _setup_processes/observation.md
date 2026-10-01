---
title: "Observation Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Observation processes register all data management operations for the observation schema — data sources, datasets, campaigns, sampling logs, samples, observations, eDNA ASVs and specialised measurement types."
permalink: /setup_process/observation/
author_profile: false
date: 2026-04-07 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

Observation processes register the operations for managing data in the `observation` schema. These processes follow the same data hierarchy as the tables themselves: you must be able to manage datasets before campaigns, campaigns before sampling logs, and so on down to individual observations.

## Process files

Process files are located at:

```
./setup/zzz/ai4sh/setup_processes/json/observation/
```

| File | Process registered | Target table(s) | Min stratum | In pilot |
|---|---|---|---|---|
| `data_source_v10_sql.json` | `manage_data_source` | `observation.data_source` | 2 | yes |
| `person_v10_sql.json` | `manage_person` | `observation.person` | 1 | yes |
| `dataset_v10_sql.json` | `manage_dataset` | `observation.dataset` + companion tables | 3 | yes |
| `dataset_tag_v10_sql.json` | `manage_dataset_tag` | `observation.dataset_tag` | 4 | yes |
| `campaign_v10_sql.json` | `manage_campaign` | `observation.campaign` + companion tables | 3 | yes |
| `campaign_tag_v10_sql.json` | `manage_campaign_tag` | `observation.campaign_tag` | 4 | yes |
| `sampling_log_v10_sql.json` | `manage_sampling_log` | `observation.sampling_log`, `sampling_log_setting_system` | 2 | yes |
| `sample_geolocation_v10_sql.json` | `manage_geolocation` / `manage_sample_geotag` | `observation.geolocation` / `observation.sample_geotag` | 3 | yes |
| `sample_v10_sql.json` | `manage_simple_sample`, `manage_geo_sample`, `manage_geolocated_sample`, `manage_geo_profile_sample`, `manage_geolocated_profile_sample` | `observation.sample` + `sample_geotag`, `sample_juxtaposition`, `sample_profile`, `sample_composition`, `sample_proximity` (and `geolocation` for the `geo_` variants) | 4 | yes |
| `observation_log_v10_sql.json` | `manage_observation_log` | `observation.observation_log` + companion tables | 2 | yes |
| `observation_v10_sql.json` | `manage_observation` | `observation.observation`, `observation_measurement`, `observation_provision_serial_nr` | 2 | yes |
| `measurement_v10_sql.json` | `manage_measurement_array` / `manage_measurement` | `observation.measurement` | 2 / 4 | yes |
| `edna_asv_v10_sql.json` | `manage_edna_asv` | `observation.edna_asv`, `observation.edna_asv_abundance` (bulk COPY) | 3 | yes |
| `sample_image_v10_sql.json` | `manage_sample_image_orientation` | `observation.sample_image_orientation` | 4 | no |
| `quantity_v10_sql.json` | `manage_quantity` | duplicate of the observation utility `manage_quantity` | 4 | no |
| `measurement_reference_v10_sql.json` | `manage_measurement_reference_array` / `manage_measurement_reference` | `observation.measurement` | 2 / 4 | no |
| `observation_reference_v10_sql.json` | `manage_observation_reference` | `observation.observation` + reference tables | 2 | no |
| `beerkan_infiltration_v10_sql.json` | `manage_beerkan_observation` / `manage_beerkan_infiltration_interval` | `observation.infiltration_beerkan`, `infiltration_beerkan_time` | 2 | no |
| `aggregate_app_observation_v10_sql.json` | `manage_aggregate_app_observation` | `observation.app_aggregate_stability`, `observation`, `observation_measurement` | 2 | no |

Files marked *no* exist on disk but are not listed in `setup_processes.txt`, so `setup_processes.ipynb` does not register them.

**Side note:**
The eDNA summary indicators need no dedicated process: they are indicator columns in the file passed to `manage_observation`, written to `observation_measurement`. The eDNA community composition (ASVs and their abundances) is loaded in bulk by `manage_edna_asv`, described below.
{: .notice}

## Data entry sequence

To enter a complete soil observation, the following sequence must be followed:

1. `manage_data_source` (if the data source is new)
2. `manage_dataset`
3. `manage_campaign`
4. `manage_sampling_log`
5. `manage_geolocation`, then a `manage_*_sample` variant
6. `manage_observation_log`
7. `manage_observation`
8. eDNA only: `manage_taxon` (see [organism_utility][setup_process_organism_utility]), then `manage_edna_asv`

If any step in this chain is missing, the foreign key constraints prevent entry of the subsequent records. This enforced chain is what ensures FAIR data compliance — every observation can be traced back to a fully documented campaign and sampling event.

{% capture notice-2 %}
## Access levels

Access levels reflect the operational role of each process:

- Stratum 1: `manage_person` — minimal privilege for attributing data to individuals
- Stratum 2: `manage_data_source`, `manage_sampling_log`, `manage_observation_log`, `manage_observation`, `manage_measurement_array`, `manage_beerkan_observation`, `manage_beerkan_infiltration_interval`, `manage_aggregate_app_observation` — regular data entry operations
- Stratum 3: `manage_dataset`, `manage_campaign`, `manage_geolocation`, `manage_sample_geotag`, `manage_edna_asv` — campaign-level management and bulk loads
- Stratum 4: `manage_*_sample`, `manage_dataset_tag`, `manage_campaign_tag`, `manage_sample_image_orientation`, `manage_quantity`, `manage_measurement` — elevated operations requiring administrative oversight
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

{% capture notice-2 %}
## Key processes in detail

### manage_data_source

Registers an external organisation or data supplier as a data source. This is a simplified record for attributing datasets to their origin — separate from the full `community.organisation` user record. Required parameters:

- `name` — unique data source name
- `url` — URL of the data source

Optional: `alias`, `address1`, `address2`, `postal_address`, `postal_zip_code`, `state`, `territory_id__territory_name`, `telephone`, `contact_name`, `contact_email`.

### manage_dataset

A dataset is the top-level grouping of related data. Required parameters:

- `data_source_id__data_source_name` — the data source owning the dataset
- `name` — unique dataset name
- `alias` — short identifier (e.g. "AI4SH", "LUCAS")

### manage_campaign

Registers a campaign (child of a dataset) and its metadata. The `manage_campaign` process handles the core `campaign` table. The same call also writes the companion tables `campaign_meta`, `campaign_tag`, `campaign_method_tier`, `campaign_location`, `campaign_provision` and `campaign_setting_system`. Extra tags can be added later with `manage_campaign_tag` (`campaign_id__campaign_name`, `tag`, `tag_type` = `substance` or `keyword`); `manage_dataset_tag` does the same for datasets.

Required parameters include:
- `dataset_id__dataset_name` — the parent dataset
- `name` — campaign name
- `contact_name`, `contact_email`

### manage_sampling_log

Registers a sampling event within a campaign. A sampling log is the bridge between the organisational structure (campaign) and individual physical samples.

Required parameters: `campaign_id`, `person_id`, start and end date, and method references.

### manage_sample variants

Registers an individual physical sample under a `sampling_log`. There is no single `manage_sample`; pick the variant matching your data:

| Process | Creates | Use when |
|---|---|---|
| `manage_simple_sample` | `sample` | No location and no depth |
| `manage_geo_sample` | `geolocation` + `sample` + `sample_geotag` + `sample_juxtaposition` | Each sample has its own new coordinates |
| `manage_geolocated_sample` | `sample` + `sample_geotag` + `sample_juxtaposition` | Coordinates already exist as a named `geolocation` |
| `manage_geo_profile_sample` | as `manage_geo_sample` + `sample_profile` | New coordinates and a depth interval |
| `manage_geolocated_profile_sample` | as `manage_geolocated_sample` + `sample_profile`, `sample_composition`, `sample_proximity` | Existing geolocation and a depth interval (the AI4SH default) |

### manage_observation_log and manage_observation

`manage_observation_log` registers the connection between a sampling log and the provision used to analyse its samples, along with sample handling logistics (preservation, storage, transportation). `manage_observation` then registers one observation per sample and stores its indicator values in `observation_measurement`. Indicator columns are named `@<indicator alias>` in the source file; each value must be consistent with the unit defined in the matching `provision_indicator`.

The figure follows one row of the Agrolab wetlab file through `manage_observation`. The process writes to two tables: `observation` (the main table, written first) and `observation_measurement` (the child table). Foreign keys given by name are confirmed and replaced by ids — the sample is looked up within the sampling log of the given observation log. The new `observation.id` is then passed to the child table, where every `@indicator` column becomes its own row. For the general mechanism, see [How Excel columns reach the database][setup_process_utility_mapping].

[![Excel row to observation and observation_measurement]({{ "/assets/media/process_mapping/observation.png" | relative_url }})]({{ "/assets/media/process_mapping/observation.png" | relative_url }})

### manage_geolocation and manage_sample_geotag

`manage_geolocation` registers a named spatial point in `observation.geolocation`:

- `name` — unique name of the point
- `x_coordinate`, `y_coordinate` — coordinates in the given spatial reference
- `spatial_reference_id__spatial-reference_name` — spatial reference (default `geographic`)
- `latitude`, `longitude` — optional WGS 84 (EPSG:4326) coordinates

`manage_sample_geotag` links a sample to a geolocation: `sample_id__sample_name`, `geolocation_id__geolocation_name`, and optionally `area_coverage` with `area_unit_id__area_unit_name` (default `m^2`).

### manage_measurement

Registers a direct instrument reading (e.g. from a handheld spectral sensor). Two variants are available:

- `manage_measurement_array` (stratum 2) — bulk insert of an array of measurements in a single call
- `manage_measurement` (stratum 4) — insert a single measurement record with administrative oversight

Both are lighter-weight observation types for sensor output that do not require the full observation_log chain.

### manage_edna_asv

Bulk-loads eDNA amplicon sequence variants (ASVs) and their abundance per observation using PostgreSQL `COPY`. It is a *bulk* process: it is not called with one parameter set per record, but with the path to a prepared CSV. `insert_tabular_data` (or `translate_tabular_data`) writes that CSV from the Excel file produced by the eDNA preparation CLI, then runs this process.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `asv_data_path` | text | yes | Path to the canonical abundance CSV written by `translate_tabular_data` / `insert_tabular_data` with target process `manage_edna_asv` |
| `method_pipeline` | text | yes | Name of the `method_pipeline` that produced the ASVs, e.g. `ai4sh-16s` |

Behaviour:

- **Insert only.** Existing ASVs and abundances are kept; only new ones are added. Rerunning is safe.
- **All or nothing.** The load aborts without writing anything if an observation is not found, a lineage has no matching taxon (load taxa with `manage_taxon` first), or an existing `asv_key` now carries a different lineage.
- Each ASV is linked to the deepest resolved taxon of its lineage.

The process is routed to its bulk importer (`import_edna_asv.py`) through `BULK_PROCESS_D` in `src/ai4sh/import_data/import_data.py`. For how to run it, see [Insert ASVs][edna_insert_asv].

{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>




[setup_db_observation]: /setup_db/observation/
[setup_process_observation_utility]: /setup_process/observation_utility/
[setup_process_organism_utility]: /setup_process/organism_utility/
[edna_insert_asv]: /edna/insert_edna_asv/
[setup_process_utility_mapping]: /setup_process/utility/#how-excel-columns-reach-the-database
