---
title: "Observation Utility Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The observation_utility schema holds all reference catalogues needed for FAIR-compliant soil observations — units, methods, instruments, taxa, spatial references, eDNA metabarcoding methods and more. It is defined across 47 JSON files (38 core catalogues + 9 eDNA) that must be seeded before any observation data can be entered."
permalink: /setup_db/observation_utility/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The `observation_utility` schema holds all reference catalogues needed for FAIR-compliant soil observations. Think of it as the controlled vocabulary layer of the database: every instrument, method, unit, taxon, spatial reference etc. used in an observation must first exist here. The `observation` schema tables cannot be populated until the relevant `observation_utility` records exist.

The schema is defined across 47 JSON files: 38 covering the core catalogues (split into those with no internal foreign key dependencies, seeded first, and those that reference other observation_utility tables, seeded second) plus 9 eDNA metabarcoding catalogue files, covered separately below.

**A note on file names**: none of the files in this schema's folder carry an `observation_utility_` prefix — every file name below is exactly as it appears on disk (e.g. `analysis_method_v10_sql.json`, not `observation_utility_analysis_method_v10_sql.json`).

## Process files — independent tables

These tables have no foreign key dependencies within `observation_utility` and can be created and seeded in any order among themselves. All include a default record with `id = 0` for unknown or undisclosed values, allowing observations to be entered even when a specific attribute is not known.

| File | Tables created | Description |
|---|---|---|
| `analysis_method_v10_sql.json` | `analysis_method` | Method used for chemical or physical analysis of a sample |
| `apparatus_v10_sql.json` | `apparatus` | Any instrument, tool, laboratory, or service that delivers observation data |
| `classification_v10_sql.json` | `order`, `family`, `genus`, `species`, `specimen` | Five-level Linnean substance classification hierarchy |
| `image_v10_sql.json` | `image_approach`, `image_substrate`, `image_camera`, `image_setup`, `image_lightsource` | Image acquisition metadata |
| `license_v10_sql.json` | `license` | Data license attached to a dataset or campaign |
| `location_method_v10_sql.json` | `location_method` | Method used to determine geolocation (e.g. GPS, map reading) |
| `macrofauna_v10_sql.json` | `macrofauna_extraction_method`, `life_cycle_stage` | Methods and life stages for macrofauna sampling |
| `method_tier_v10_sql.json` | `method_tier` | Level of professionality (in-situ, lab, drone, satellite, document, etc.) |
| `microbiometer_version_v10_sql.json` | `microbiometer_version` | Microbiometer kit version |
| `monolith_extraction_v10_sql.json` | `monolith_extraction` | Method for extracting soil, sediment or ice core monoliths |
| `preparation_v10_sql.json` | `preparation` | Sample preparation method prior to analysis |
| `preservation_v10_sql.json` | `preservation` | Sample preservation method (cold, frozen, chemical shield, etc.) |
| `provider_v10_sql.json` | `provider` | Any tool, instrument, sensing service, or data provider delivering results |
| `quantity_v10_sql.json` | `quantity` | Physical quantities that can be observed (e.g. pH, organic carbon) |
| `reference_proprietor_v10_sql.json` | `reference_proprietor` | Proprietor of a reference standard |
| `setting_system_v10_sql.json` | `setting_system` | Thematic frame for defining local juxtapositions (field, forest, lake, etc.) |
| `software_v10_sql.json` | `software` | Named software tool/version (e.g. a bioinformatics pipeline component), attributed by name and version |
| `soil_horizon_v10_sql.json` | `soil_horizon` | Standard soil horizon designations |
| `sound_v10_sql.json` | `sound_setup`, `sound_mic` | Sound recording setup and microphone types |
| `spatial_reference_v10_sql.json` | `spatial_reference` | Spatial reference systems for geolocations |
| `spectroscopy_method_v10_sql.json` | `spectroscopy_method` | Type of spectral analysis method |
| `storage_v10_sql.json` | `storage` | Sample storage conditions before analysis |
| `taxa_level_v10_sql.json` | `taxa_level` | Linnean classification levels |
| `taxa_status_v10_sql.json` | `taxa_status` | Status flags for outdated or redundant taxa definitions |
| `transportation_v10_sql.json` | `transportation` | Transportation conditions for samples before analysis |
| `unit_v10_sql.json` | `unit` | Units of reported observation values |

## Process files — tables with internal dependencies

These tables reference other `observation_utility` tables and must be created after their dependencies.

| File | Tables created | Dependencies |
|---|---|---|
| `analysis_method_translate_v10_sql.json` | `analysis_method_translate` | `species`, `analysis_method` |
| `indicator_v10_sql.json` | `indicator` | `quantity` |
| `indicator_default_unit_v10_sql.json` | `indicator_default_unit` | `species`, `indicator`, `unit` |
| `juxtaposition_v10_sql.json` | `juxtaposition` | `setting_system` |
| `profiling_v10_sql.json` | `profiling` | `unit` (depth profiling for cores) |
| `provision_v10_sql.json` | `provision`, `provision_method_tier`, `provision_serial_nr`, `provision_calibration` | `apparatus`, `provider`, `method_tier` — combines these three into a single instrument+service+tier record, plus companion tables for method tier flags, serial numbers, and calibration |
| `provision_indicator_v10_sql.json` | `provision_indicator` | `provision`, `indicator`, `analysis_method`, `unit` — the tangible observation values delivered by a specific provision |
| `spectrometer_v10_sql.json` | `spectrometer` | `provision`, `provision_serial_nr` — registers a spectral sensor with its full wavelength axis array |
| `quantity_default_unit_v10_sql.json` | `quantity_default_unit` | `quantity`, `species`, `unit` |
| `taxa_v10_sql.json` | `taxa` | `taxa_level`, `taxa_status` |
| `taxa_function_v10_sql.json` | `taxa_function` | `taxa_level`, `taxa_status` |
| `unit_translate_v10_sql.json` | `unit_translate` | `unit` — mathematical conversion factors between units |

The 9 eDNA files are covered separately below rather than in this table, since they form their own thematic group.

## eDNA reference catalogues

Environmental DNA (eDNA) metabarcoding — identifying soil taxa by sequencing DNA extracted directly from a sample.

Nine files/tables cover the metabarcoding pipeline step by step. Every real table name in this
schema is `edna_`-prefixed:

| File | Table | Dependencies | Description |
|---|---|---|---|
| `edna_reference_db_v10_sql.json` | `edna_reference_db` | none | A named reference database (name + version), with URL/DOI/abstract |
| `edna_sequence_library_v10_sql.json` | `edna_sequence_library` | `reference_proprietor` | A named DNA sequence reference library (e.g. a curated barcode database) |
| `edna_nucleotide_archive_v10_sql.json` | `edna_nucleotide_archive` | `reference_proprietor` | A public nucleotide sequence archive (e.g. GenBank) |
| `edna_extraction_v10_sql.json` | `edna_extraction` | none | DNA extraction method/protocol from a soil sample |
| `edna_amplification_v10_sql.json` | `edna_amplification` | none | PCR amplification protocol — used for both forward and reverse primers |
| `edna_purification_v10_sql.json` | `edna_purification` | none | Clean-up method for extracted or amplified DNA |
| `edna_sequencing_v10_sql.json` | `edna_sequencing` | none | Sequencing platform, strategy, and data requirement (e.g. Illumina, Nanopore) |
| `edna_Metabarcoding_pipeline_v10_sql.json`* | `edna_metabarcoding_pipeline` | `edna_extraction`, `edna_amplification` (×2, forward/reverse), `edna_purification`, `edna_sequencing` | Combines one extraction + two amplifications + one purification + one sequencing into a single, uniquely identifiable pipeline definition |
| `edna_bioiformatics_v10_sql.json`* | `edna_bioinformatics` | `analysis_method`, `taxa`, `taxa_function`, `unit` | A named bioinformatics analysis, combining an `analysis_method` with a `taxa` and `taxa_function` target |

\* These two file names are exactly as they appear on disk, including a stray capital letter (`edna_Metabarcoding_pipeline_v10_sql.json`) and a typo (`edna_bioiformatics_v10_sql.json`, missing the "n" in "bioinformatics") — the tables they create are spelled correctly (`edna_metabarcoding_pipeline`, `edna_bioinformatics`). Don't be thrown off if the file you're looking for doesn't match the usual naming pattern.

`edna_metabarcoding_pipeline` is the eDNA equivalent of `provision` elsewhere in this schema: it bundles several method choices into one reusable, referenceable combination, unique on `(edna_extraction_id, edna_forward_amplification_id, edna_reverse_amplification_id, edna_purification_id, edna_sequencing_id)`.

## Key concepts

**Provision** is the central linking concept in observation_utility. A provision combines an `apparatus` (what instrument/tool), a `provider` (what service or supplier), and a `method_tier` (what level of professionality). A `provision_indicator` then specifies exactly which measurable quantities a provision delivers, with associated analysis method and unit. Every observation in the `observation` schema links back to a provision.

![Provision schema]({{ "/assets/media/observation_utility/provision.png" | relative_url }})

**Indicator** represents a single measurable result (e.g. soil pH, organic carbon %). Indicators belong to a `quantity` (the physical property type). The same indicator can be delivered by multiple provisions, and one indicator can be declared equivalent to another via `indicator_parity` (`src_indicator_id`/`dst_indicator_id`, both referencing `indicator`) — useful when two differently-named indicators from different sources measure the same thing.

![Indicator schema]({{ "/assets/media/observation_utility/indicator.png" | relative_url }})

**Profiling** describes z-dimension sampling profiles (soil cores, sediment cores, ice cores) by specifying depth increments in a given unit. Samples with a profile dimension reference a profiling record.
