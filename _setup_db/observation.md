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
| `observation/campaign_v10_sql.json` | `campaign`, `campaign_meta`, `campaign_tag`, `campaign_method_tier`, `campaign_location`, `campaign_provision`, `campaign_setting_system` | Child of dataset; captures details of a specific data collection effort |
| `observation/sampling_log_v10_sql.json` | `sampling_log` | A sampling event — one or multiple samples taken with consistent methods over a period |
| `observation/sample_geolocation_v10_sql.json` | `sample_geolocation` | Spatial coordinates for a geolocated sample |
| `observation/sample_v10_sql.json` | `sample` | An individual sample acquired under a sampling log |
| `observation/sample_z_profile_v10_sql.json` | `sample_profile` | Depth profile interval (profile_min, profile_max) for samples with a z-dimension |
| `observation/sample_image_v10_sql.json` | `sample_image`, `sample_image_orientation` | Image records associated with samples, plus their orientation metadata |
| `observation/observation_log_v10_sql.json` | `observation_log`, `observation_log_meta`, `observation_log_method_tier`, `observation_log_provision` | Links a sampling log directly to a provision; companion tables record the logistics (preparation, preservation, storage, transportation), method-tier flags, and — since a log's main `provision_id` is one column — any additional provisions used |
| `observation/observation_v10_sql.json` | `observation`, `observation_temperature`, `observation_provision_serial_nr` | The actual measured value for a specific provision, plus companion tables for temperature context and the specific provision serial number used |
| `observation/spectra_v10_sql.json` | `spectra_scan` | Spectral scan metadata — signal statistics, spectroscopy method, and quality flags linked to an observation |
| `observation/measurement_v10_sql.json` | `measurement` | General sensor or instrument measurements |
| `observation/microbiometer_measurement_v10_sql.json` | `microbiometer_measurement` | Microbiometer carbon content measurements linked to an observation |
| `observation/infiltration_beerkan_v10_sql.json` | `infiltration_beerkan` | BeerKan ring infiltration measurements |
| `observation/slakes_v10_sql.json` | `app_aggregate_stability` | Aggregate stability measurements from the SLAKES app |
| `observation/macrofauna_v10_sql.json` | `macrofauna` | Macrofauna count and biomass observations |
| `observation/macrofauna_image_v10_sql.json` | `macrofauna_image` | Image records for automated macrofauna detection |
| `observation/edna_v10_sql.json` | `edna_nucleotide`, `taxa_bioinformatics`, `edna_measurement`, `taxa_biodiversity_measurement` | Environmental DNA (eDNA) metabarcoding observations, see below |
| `observation/edna_fastp_qc_v10_sql.json` | `edna_fastp_qc` | eDNA read quality-control step, see below |
| `observation/edna_demux_trim_v10_sql.json` | `edna_feature_abundance` | eDNA demultiplex/trim pipeline output, see below |
| `observation/edna_denoise_v10_sql.json` | `edna_denoise` | eDNA denoising pipeline step, see below |
| `observation/edna_chimera_vsearch_v10_sql.json` | `edna_chimera_vsearch` | eDNA chimera detection (vsearch) pipeline step, see below |
| `observation/edna_annotation_v10_sql.json` | `edna_annotation` | eDNA taxonomic annotation pipeline step, see below |
| `observation/edna_merge_flash_v10_sql.json` | `edna_merge_count` | eDNA read-merging (FLASh) pipeline step, see below |
| `observation/edna_taxa_abundance_v10_sql.json` | `edna_taxa_abundance` | eDNA taxon abundance pipeline output, see below |
| `observation/edna_functional_abundance_v10_sql.json` | `edna_functional_abundance` | eDNA functional abundance pipeline output, see below |

## eDNA observation tables

| Table | Description |
|---|---|
| `edna_nucleotide` | Links a `sample` to a submitted nucleotide sequence — the archive it was deposited in and its accession code/URL |
| `taxa_bioinformatics` | One bioinformatics run against an `edna_nucleotide` record: which `metabarcoding_pipeline`, which `bioinformatics` treatment, treatment date, and the taxon it resolved to |
| `edna_measurement` | An indicator value attached to a regular `observation` record, specific to eDNA-derived measurements (value, standard deviation, number of repeats) |
| `taxa_biodiversity_measurement` | The biodiversity value (e.g. relative abundance) produced by a `taxa_bioinformatics` run |

eDNA observations still use the same sample infrastructure as any other observation — dataset → campaign → sampling log → sample — the eDNA-specific tables attach to a `sample` and to the eDNA reference catalogues, not to a separate hierarchy.

Beyond these 4 tables, 8 further tables cover the eDNA bioinformatics pipeline itself — read quality control, denoising, chimera detection, taxonomic annotation, and the resulting taxon/functional abundance outputs: `edna_fastp_qc`, `edna_feature_abundance`, `edna_denoise`, `edna_chimera_vsearch`, `edna_annotation`, `edna_merge_count`, `edna_taxa_abundance`, `edna_functional_abundance` (see the process-files table above for which file creates each).

**Partly wired into process setup.** The 7 `manage_edna_*` process files for the `observation_utility` eDNA reference catalogues (`edna_sequence_library`, `edna_nucleotide_archive`, `edna_extraction`, `edna_purification`, `edna_amplification`, `edna_sequencing`, `edna_metabarcoding_pipeline`) **are** now registered and listed in the process pilot list, under an `### OBSERVATION UTILITIES eDNA ###` section — this table's data can be entered through the normal `manage_*` workflow. What's still missing: no `manage_*` process registrations exist yet for any of the 4 `observation`-schema eDNA tables above (`edna_nucleotide`, `taxa_bioinformatics`, `edna_measurement`, `taxa_biodiversity_measurement`), nor for the 8 pipeline-output tables just listed. Those 12 tables are created by `setup_db.ipynb`, but can't yet be entered through the normal `manage_*` process workflow — see [Observation Processes][setup_process_observation].

## Table hierarchy

`sample` and `observation_log` are **siblings** — both children of `sampling_log`, not one
nested inside the other — and both converge as parallel foreign keys on `observation`:

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
      sample_geolocation (→ sampling_log, observation_utility.spatial_reference)
      sample (→ sampling_log)                                    ─┐
      observation_log (→ sampling_log, observation_utility.provision)  ├─ siblings
        observation_log_meta / _method_tier / _provision (→ observation_log) ─┘

observation (→ sample, observation_log, observation_utility.provision)
  observation_temperature, observation_provision_serial_nr (→ observation)
```

See the diagram below for the same shape drawn visually.

![Sample and observation_log both feed observation](/assets/media/observation/sampling_log.png)

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

### sample and sample_geolocation

`sample` represents an individual physical sample. `sample_geolocation` optionally records its latitude, longitude, elevation, and spatial reference. Decoupled from `sample` so that non-geolocated samples are still representable.

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
- **macrofauna** and **macrofauna_image** — macrofauna counts, biomass, and associated image records
- **measurement** — direct sensor readings (e.g. from a handheld spectral sensor)
- **edna_nucleotide**, **taxa_bioinformatics**, **edna_measurement**, **taxa_biodiversity_measurement**, plus 8 eDNA pipeline-output tables — eDNA metabarcoding results, see [eDNA observation tables](#edna-observation-tables) above


[setup_db_observation_utility]: /setup_db/observation_utility/
[setup_process_observation]: /setup_process/observation/
[setup_process_schema_conventions]: /setup_process/schema_conventions/
[spectra]: /spectra/
