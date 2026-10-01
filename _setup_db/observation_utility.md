---
title: "Observation Utility Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The observation_utility schema holds all reference catalogues needed for FAIR-compliant soil observations — units, methods, instruments, spatial references, software, eDNA primer pairs, lab protocols, method pipelines and more. It is defined across 36 JSON files that must be seeded before any observation data can be entered."
permalink: /setup_db/observation_utility/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

The `observation_utility` schema holds all reference catalogues needed for FAIR-compliant soil observations. Think of it as the controlled vocabulary layer of the database: every instrument, method, unit, spatial reference etc. used in an observation must first exist here. The `observation` schema tables cannot be populated until the relevant `observation_utility` records exist.

The schema is defined across 36 JSON files, split into those with no internal foreign key dependencies (seeded first), those that reference other observation_utility tables (seeded second), and the eDNA method catalogues and method pipelines, covered separately below.

Taxonomy (taxon, rank, status, function) is **not** part of this schema — it lives in [organism_utility][setup_db_organism_utility]. Tables here that need a taxon (e.g. `method_pipeline_step`) reference `organism_utility` directly.

**A note on file names**: none of the files in this schema's folder carry an `observation_utility_` prefix — every file name below is exactly as it appears on disk (e.g. `analysis_method_v10_sql.json`, not `observation_utility_analysis_method_v10_sql.json`).

## Process files — independent tables

These tables have no foreign key dependencies within `observation_utility` and can be created and seeded in any order among themselves. All include a default record with `id = 0` for unknown or undisclosed values, allowing observations to be entered even when a specific attribute is not known.

| File | Tables created | Description |
|---|---|---|
| `analysis_method_v10_sql.json` | `analysis_method` | Method used for chemical, physical or biological analysis of a sample, with `standard_schema`, `standard_code` and `url` (e.g. `ai4sh 16s metabarcoding`) |
| `apparatus_v10_sql.json` | `apparatus` | Any instrument, tool, laboratory, or service that delivers observation data |
| `classification_v10_sql.json` | `order`, `family`, `genus`, `species`, `specimen` | Five-level Linnean substance classification hierarchy |
| `image_v10_sql.json` | `image_approach`, `image_substrate`, `image_camera`, `image_setup`, `image_lightsource` | Image acquisition metadata |
| `license_v10_sql.json` | `license` | Data license attached to a dataset or campaign |
| `location_method_v10_sql.json` | `location_method` | Method used to determine geolocation (e.g. GPS, map reading) |
| `macrofauna_v10_sql.json` | `macrofauna_extraction_method`, `life_cycle_stage` | Methods and life stages for macrofauna sampling |
| `method_tier_v10_sql.json` | `method_tier` | Level of professionality (in-situ, lab, drone, satellite, document, etc.) |
| `monolith_extraction_v10_sql.json` | `monolith_excavation_method` | Method for excavating soil, sediment or ice core monoliths (note: file name and table name differ) |
| `preparation_v10_sql.json` | `preparation` | Sample preparation method prior to analysis |
| `preservation_v10_sql.json` | `preservation` | Sample preservation method (cold, frozen, chemical shield, etc.) |
| `provider_v10_sql.json` | `provider` | Any tool, instrument, sensing service, or data provider delivering results |
| `quantity_v10_sql.json` | `quantity` | Physical quantities that can be observed (e.g. pH, organic carbon) |
| `reference_proprietor_v10_sql.json` | `reference_proprietor` | Proprietor of a reference standard |
| `setting_system_v10_sql.json` | `setting_system` | Thematic frame for defining local juxtapositions (field, forest, lake, etc.) |
| `soil_horizon_v10_sql.json` | `soil_horizon` | Standard soil horizon designations |
| `sound_v10_sql.json` | `sound_setup`, `sound_mic` | Sound recording setup and microphone types |
| `spatial_reference_v10_sql.json` | `spatial_reference` | Spatial reference systems for geolocations |
| `spectroscopy_method_v10_sql.json` | `spectroscopy_method` | Type of spectral analysis method |
| `storage_v10_sql.json` | `storage` | Sample storage conditions before analysis |
| `transportation_v10_sql.json` | `transportation` | Transportation conditions for samples before analysis |
| `unit_v10_sql.json` | `unit` | Units of reported observation values |

## Process files — tables with internal dependencies

These tables reference other `observation_utility` tables and must be created after their dependencies.

| File | Tables created | Dependencies |
|---|---|---|
| `analysis_method_translate_v10_sql.json` | `analysis_method_translate` | `species`, `analysis_method` |
| `indicator_v10_sql.json` | `indicator`, `indicator_parity` | `quantity` |
| `indicator_default_unit_v10_sql.json` | `indicator_default_unit` | `species`, `indicator`, `unit` |
| `juxtaposition_v10_sql.json` | `juxtaposition`, `composition`, `proximity` | `setting_system` |
| `profiling_v10_sql.json` | `profiling` | `unit` (depth profiling for cores) |
| `provision_v10_sql.json` | `provision`, `provision_method_tier`, `provision_serial_nr`, `provision_calibration` | `apparatus`, `provider`, `method_tier` — combines these three into a single instrument+service+tier record, plus companion tables for method tier flags, serial numbers, and calibration |
| `provision_indicator_v10_sql.json` | `provision_indicator` | `provision`, `indicator`, `analysis_method`, `unit` — the tangible observation values delivered by a specific provision |
| `spectrometer_v10_sql.json` | `spectrometer` | `provision`, `provision_serial_nr` — registers a spectral sensor with its full wavelength axis array |
| `quantity_default_unit_v10_sql.json` | `quantity_default_unit` | `quantity`, `species`, `unit` |
| `unit_translate_v10_sql.json` | `unit_translate` | `unit` — mathematical conversion factors between units |

## eDNA method catalogues and method pipelines

Five tables describe *how* eDNA metabarcoding results were produced — the laboratory kits, the primers, the bioinformatics software and the ordered steps that combine them. They are placed last among the observation utilities in the pilot file. `method_pipeline_v10_sql.json` is run in a separate section *after* [organism_utility][setup_db_organism_utility], because pipeline steps reference `organism_utility.taxonomy_reference`.

| File | Tables created | Dependencies | Description |
|---|---|---|---|
| `software_v10_sql.json` | `software` | none | A software tool **and version** (e.g. `qiime2 2024.10`), with `url`, `doi`, `abstract` |
| `edna_primer_pair_v10_sql.json` | `edna_primer_pair` | none | A forward + reverse primer used together in PCR, with marker gene, target region and target group |
| `lab_protocol_v10_sql.json` | `lab_protocol` | none | A laboratory protocol: extraction, amplification, purification, library preparation or sequencing |
| `method_pipeline_v10_sql.json` | `method_pipeline`, `method_pipeline_step` | `analysis_method`, `lab_protocol`, `edna_primer_pair`, `software`, `organism_utility.taxonomy_reference` | A named, versioned multi-step analysis method and its numbered steps |

Key columns and constraints:

| Table | Columns | Constraints |
|---|---|---|
| `software` | `name`, `version` (default `'unspecified'`), `alias`, `url`, `doi`, `abstract` | `UNIQUE (name, version)`; `alias` UNIQUE, by convention `'<name> <version>'` |
| `edna_primer_pair` | `name`, `marker_gene`, `target_region`, `target_group`, `forward_name`, `forward_sequence`, `reverse_name`, `reverse_sequence`, `doi`, `url`, `abstract` | `name` UNIQUE |
| `lab_protocol` | `name`, `protocol_type`, `kit`, `manufacturer`, `platform`, `strategy`, `data_requirement`, `doi`, `url`, `abstract` | `name` UNIQUE; `protocol_type` ∈ {`''`, `extraction`, `amplification`, `purification`, `library_preparation`, `sequencing`, `other`} |
| `method_pipeline` | `name`, `version` (default `'unspecified'`), `analysis_method_id`, `url`, `doi`, `abstract` | `name` UNIQUE; `analysis_method_id` NOT NULL |
| `method_pipeline_step` | `method_pipeline_id`, `step`, `stage`, `name`, `lab_protocol_id`, `edna_primer_pair_id`, `software_id`, `taxonomy_reference_id`, `parameters` (JSONB, default `{}`), `abstract` | `UNIQUE (method_pipeline_id, step)`; `stage` ∈ {`laboratory`, `bioinformatics`}; the four references are nullable |

[![eDNA method catalogues and method pipelines]({{ "/assets/media/observation_utility/edna_method_pipeline.png" | relative_url }})]({{ "/assets/media/observation_utility/edna_method_pipeline.png" | relative_url }})

**Structure only**: these setup files create the tables and — for `software`, `edna_primer_pair` and `lab_protocol` — insert a blank `id = 0` row; `method_pipeline_v10_sql.json` creates empty tables. The actual content (the two AI4SH pipelines `ai4sh-16s` and `ai4sh-its` with 12 steps each, 7 software tools, 2 primer pairs, 4 lab protocols) is inserted from Excel in the utility insert chain — see [eDNA method catalogues][edna_catalogues].

A `method_pipeline` hangs off an `analysis_method`, so it plugs into the existing `provision_indicator` mechanism: the summary indicators delivered by provision `ai4sh-metabarcoding` point to the analysis methods `ai4sh 16s metabarcoding` and `ai4sh its metabarcoding`, and each of those has exactly one pipeline. For the full eDNA picture, see [eDNA metabarcoding][setup_db_edna].

## Key concepts

**Provision** is the central linking concept in observation_utility. A provision combines an `apparatus` (what instrument/tool), a `provider` (what service or supplier), and a `method_tier` (what level of professionality). A `provision_indicator` then specifies exactly which measurable quantities a provision delivers, with associated analysis method and unit. Every observation in the `observation` schema links back to a provision.

[![Provision schema]({{ "/assets/media/observation_utility/provision.png" | relative_url }})]({{ "/assets/media/observation_utility/provision.png" | relative_url }})

**Indicator** represents a single measurable result (e.g. soil pH, organic carbon %). Indicators belong to a `quantity` (the physical property type). The same indicator can be delivered by multiple provisions, and one indicator can be declared equivalent to another via `indicator_parity` (`src_indicator_id`/`dst_indicator_id`, both referencing `indicator`) — useful when two differently-named indicators from different sources measure the same thing.

[![Indicator schema]({{ "/assets/media/observation_utility/indicator.png" | relative_url }})]({{ "/assets/media/observation_utility/indicator.png" | relative_url }})

**Profiling** describes z-dimension sampling profiles (soil cores, sediment cores, ice cores) by specifying depth increments in a given unit. Samples with a profile dimension reference a profiling record.

[setup_db_organism_utility]: /setup_db/organism_utility/
[setup_db_edna]: /setup_db/edna_metabarcoding/
[edna_catalogues]: /edna/edna_catalogues/
