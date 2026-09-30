---
title: "Observation Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The observation schema stores actual soil property data, organised hierarchically through data source, datasets, campaigns, sampling logs, samples and observations. All observation records depend on the observation_utility reference catalogues."
permalink: /setup_db/observation/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

The `observation` schema stores actual soil property data. Data is organised in a strict hierarchy: an observation belongs to a sample, a sample belongs to a sampling log, a sampling log belongs to a campaign, a campaign belongs to a dataset. Every step in this chain must exist before the next can be created. All observation values link to reference records in `observation_utility`, and taxon-resolved records (macrofauna, eDNA ASVs) also to `organism_utility.taxon`.

## Process files

| File | Tables created | Description |
|---|---|---|
| `observation/data_source_v10_sql.json` | `data_source` | Simplified copy of `community.organisation` — passive data suppliers who need not be full system users |
| `observation/person_v10_sql.json` | `person` | Simplified copy of `community.user` with no login credentials — allows attributing data to non-registered persons (GDPR consideration) |
| `observation/dataset_v10_sql.json` | `dataset`, `dataset_meta`, `dataset_tag`, `dataset_method_tier`, `dataset_location`, `dataset_setting_system` | Top-level grouping of related data (e.g. LUCAS, AI4SoilHealth), with companion tables mirroring those of campaign |
| `observation/campaign_v10_sql.json` | `campaign`, `campaign_meta`, `campaign_tag`, `campaign_method_tier`, `campaign_location`, `campaign_provision`, `campaign_setting_system` | Child of dataset; captures details of a specific data collection effort |
| `observation/sampling_log_v10_sql.json` | `sampling_log`, `sampling_log_setting_system` | A sampling event — one or multiple samples taken with consistent methods over a period |
| `observation/sample_geolocation_v10_sql.json` | `geolocation` | Named spatial point (x/y, lat/lon, elevation, spatial reference). Note: file name and table name differ — the table is `geolocation`, not `sample_geolocation` |
| `observation/sample_v10_sql.json` | `sample`, `sample_juxtaposition`, `sample_profile`, `sample_geotag`, `sample_composition`, `sample_proximity` | An individual sample acquired under a sampling log, with companion tables for setting, depth profile, link to a `geolocation`, composition and proximity |
| `observation/sample_z_profile_v10_sql.json` | `sample_profile` | Stand-alone definition of the depth profile table (also created by `sample_v10_sql.json`); not in the pilot file |
| `observation/sample_image_v10_sql.json` | `sample_image`, `sample_image_orientation` | Image records associated with samples, plus their orientation metadata |
| `observation/observation_log_v10_sql.json` | `observation_log`, `observation_log_meta`, `observation_log_method_tier`, `observation_log_provision` | Links a sampling log directly to a provision; companion tables record the logistics (preparation, preservation, storage, transportation), method-tier flags, and — since a log's main `provision_id` is one column — any additional provisions used |
| `observation/observation_v10_sql.json` | `observation`, `observation_temperature`, `observation_provision_serial_nr` | One observation event for a specific sample and provision, plus companion tables for temperature context and the specific provision serial number used |
| `observation/spectra_v10_sql.json` | `spectra_scan` | Spectral scan metadata — signal statistics, spectroscopy method, and quality flags linked to an observation |
| `observation/measurement_v10_sql.json` | `measurement`, `observation_measurement`, `observation_measurement_array` | Indicator values per observation (`observation_id`, `indicator_id`, `value`, `standard_deviation`, `n_repeat`); `observation_measurement` is the table filled by `manage_observation`, the `_array` variant holds array values |
| `observation/microbiometer_measurement_v10_sql.json` | `microbiometer_measurement` | Microbiometer carbon content measurements linked to an observation |
| `observation/infiltration_beerkan_v10_sql.json` | `infiltration_beerkan`, `infiltration_beerkan_time` | BeerKan ring infiltration measurements and their time intervals |
| `observation/slakes_v10_sql.json` | `app_aggregate_stability` | Aggregate stability measurements from the SLAKES app |
| `observation/macrofauna_v10_sql.json` | `monolith`, `macrofauna` | Excavated monoliths and the macrofauna counted in them; `macrofauna.taxon_id` references `organism_utility.taxon` |
| `observation/macrofauna_image_v10_sql.json` | `observation_utility.macrofauna_image_setting` | Image settings for automated macrofauna detection (note: created in the `observation_utility` schema although the file sits here) |
| `observation/edna_asv_v10_sql.json` | `edna_asv`, `edna_asv_abundance` | eDNA amplicon sequence variants and their abundance per observation — see below |
| `observation/edna_run_step_v10_sql.json` | `edna_run_step` | eDNA reads in/out per bioinformatics step per observation — see below |

## eDNA observation tables

eDNA metabarcoding results use the normal hierarchy — dataset → campaign → sampling log → observation log → observation. The 19 summary indicators per sample (richness, Shannon, Simpson, Pielou, Chao1, functional predictions) are ordinary indicator values in `observation_measurement`. What is new is the community composition — which organisms were found and how abundant they were — held in three dedicated tables:

| Table | Columns | Constraints | Description |
|---|---|---|---|
| `edna_asv` | `method_pipeline_id`, `asv_key`, `taxon_id`, `lineage`, `sequence`, `sequence_md5` | `UNIQUE (method_pipeline_id, asv_key)`; `sequence_md5` UNIQUE; index on `taxon_id` | One row per amplicon sequence variant (ASV) and pipeline. `taxon_id` is the deepest resolved taxon of the lineage |
| `edna_asv_abundance` | `observation_id`, `edna_asv_id`, `rel_abundance`, `read_count`, `raw_read_count` | PK `(observation_id, edna_asv_id)`; index on `edna_asv_id` | Abundance of an ASV in one observation. Only non-zero values are stored. `read_count` is the **rarefied** count; `raw_read_count` is empty until the lab delivers unrarefied counts |
| `edna_run_step` | `observation_id`, `method_pipeline_step_id`, `reads_in`, `reads_out` | PK `(observation_id, method_pipeline_step_id)` | Reads entering and leaving each bioinformatics step per observation (e.g. DADA2 denoising stats). Created but empty |

![eDNA observation tables]({{ "/assets/media/observation/edna.png" | relative_url }})

All three are audited on `UPDATE` and `DELETE` only — the bulk `INSERT` of hundreds of thousands of abundance rows is not written to the audit log.

The *method* behind these results (primers, kits, software versions, reference databases, parameters) is not repeated per record — it is reached through `edna_asv.method_pipeline_id` → `observation_utility.method_pipeline` → `method_pipeline_step`. See [eDNA metabarcoding][setup_db_edna] for the full picture.

The `organism` folder also holds `observation_measurement_taxon_v10_sql.json`, which creates `observation.taxon_observation_measurement` (taxon-level indicator values). It is not in the pilot file and must be inserted as a utility beforehand — see [Organism][setup_db_organism].

## Table hierarchy

`sample` and `observation_log` are **siblings** — both children of `sampling_log`, not one
nested inside the other. This is needed because several different types of observations (each defined by its own `observation_log`) can be made on the same sample; it is also possible to repeat an observation of the same type on the same sample at a later time, again requiring a separate `observation_log`. To solve this in the database both `sample` and `observation_log` converge as parallel foreign keys on `observation`:

```
data_source
dataset (→ data_source)
  campaign (→ dataset)
    campaign_meta (→ campaign, observation_utility.*)
    campaign_tag (→ campaign)
    campaign_method_tier (→ campaign)
    campaign_location (→ campaign, utility.territory, observation_utility.spatial_reference)
    campaign_provision (→ campaign, observation_utility.provision)
    campaign_setting_system (→ campaign, observation_utility.setting_system)
    sampling_log (→ campaign)
      sample (→ sampling_log)                                    ─┐
        sample_geotag (→ sample, geolocation)                     │
      observation_log (→ sampling_log, observation_utility.provision)  ├─ siblings
        observation_log_meta / _method_tier / _provision (→ observation_log) ─┘

geolocation (→ observation_utility.spatial_reference)

observation (→ sample, observation_log, observation_utility.provision)
  observation_temperature, observation_provision_serial_nr (→ observation)
  observation_measurement (→ observation, observation_utility.indicator)
  edna_asv_abundance (→ observation, edna_asv)
    edna_asv (→ observation_utility.method_pipeline, organism_utility.taxon)
  edna_run_step (→ observation, observation_utility.method_pipeline_step)
```

See the diagram below for the same shape drawn visually.

![Sample and observation_log both feed observation]({{ "/assets/media/observation/sampling_log.png" | relative_url }})

## Key tables in detail

### dataset

A dataset is a coherent collection of data from one or more campaigns — for example, LUCAS 2009–2022 or the AI4SoilHealth project. Datasets are associated with a `data_source`.

Key columns: `id`, `data_source_id`, `name`, `alias`, `status_code`.

### campaign

A campaign is a child of a dataset where sampling methods may vary in detail. For example, LUCAS 2009 and LUCAS 2015 are two campaigns within the LUCAS dataset. Each AI4SH pilot site is a separate campaign.

The `campaign` table has six companion meta tables:

- **campaign_meta** — abstract, DOI, URL, keywords, time series flag, license, taxonomic scope
- **campaign_tag** — free-text tags (substance or keyword), one row per tag — the junction-table replacement for what used to be array columns, see [Schema conventions][setup_process_schema_conventions]
- **campaign_method_tier** — boolean flags for which method tiers (in-situ, in-home, in-lab, from-drone, from-satellite, etc.) are used in the campaign
- **campaign_location** — geographic bounding box and spatial reference
- **campaign_provision** — which provisions (instrument+provider+method_tier combinations) are used
- **campaign_setting_system** — which setting systems (field, forest, lake, etc.) apply

### sampling_log

A sampling log is a sampling event within a campaign, covering one or multiple samples taken with consistent methods. It records the responsible person and time window.

### sample and geolocation

`sample` represents an individual physical sample. `geolocation` is a named spatial point (x/y, latitude/longitude, elevation, spatial reference), and `sample_geotag` links a sample to one. Decoupled from `sample` so that non-geolocated samples are still representable, and so that several samples (e.g. depths) can share one point.

### observation_log and observation

`observation_log` links directly to `sampling_log` (not to `sample`) and to a provision (instrument + service + method tier); its companion tables record the sample handling chain (preparation, preservation, storage, transportation via `observation_log_meta`), method-tier flags, and any additional provisions used beyond the log's own `provision_id`.

`sample` and `observation_log` are **siblings**, both children of `sampling_log` — `observation_log` is not a child of `sample`. `observation` itself carries three foreign keys — `sample_id`, `observation_log_id`, and `provision_id` — converging both branches: it's the actual numeric or text result for a specific sample, recorded under a specific observation log, from a specific provision. There is no `provision_indicator` column on `observation` itself; which indicator a value represents is determined by the `provision_id` in context of that provision's registered `provision_indicator` records.

### Spectral data

Spectral scans are a different shape from a regular scalar observation — one row holds a whole
signal array, not a single value:

- **spectra_scan** — `signal_mean` and `signal_standard_deviation` array columns plus
  `spectroscopy_method_id`, linked directly to `observation`. Covers the FOSS DS2500,
  Neospectra, FTIR, and LIBS instrument families documented in the [Spectra][spectra]
  collection.

### Specialised observation tables

Additional tables cover other non-standard observation types:

- **infiltration_beerkan** — BeerKan ring infiltration test results
- **monolith** and **macrofauna** — excavated monoliths and macrofauna counts and biomass, resolved to `organism_utility.taxon`
- **observation_measurement** — indicator values per observation, for all scalar results (wet chemistry, eDNA summary indicators etc.)
- **edna_asv**, **edna_asv_abundance**, **edna_run_step** — eDNA metabarcoding community composition, see [eDNA observation tables](#edna-observation-tables) above


[setup_db_observation_utility]: /setup_db/observation_utility/
[setup_process_observation]: /setup_process/observation/
[setup_process_schema_conventions]: /setup_process/schema_conventions/
[spectra]: /spectra/
[setup_db_edna]: /setup_db/edna_metabarcoding/
[setup_db_organism]: /setup_db/organism/
