---
title: "Observation Utility Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The observation_utility schema holds all reference catalogues needed for FAIR-compliant soil observations — units, methods, instruments, taxa, spatial references and more. It is defined across 37 JSON files that must be seeded before any observation data can be entered."
permalink: /setup_db/observation_utility/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The `observation_utility` schema holds all reference catalogues needed for FAIR-compliant soil observations. Think of it as the controlled vocabulary layer of the database: every instrument, method, unit, taxon, and spatial reference used in an observation must first exist here. The `observation` schema tables cannot be populated until the relevant `observation_utility` records exist.

The schema is defined across 37 JSON files, split into two groups: those with no internal foreign key dependencies (seeded first), and those that reference other observation_utility tables (seeded second).

## Process files — independent tables

These tables have no foreign key dependencies within `observation_utility` and can be created and seeded in any order among themselves. All include a default record with `id = 0` for unknown or undisclosed values, allowing observations to be entered even when a specific attribute is not known.

| File | Tables created | Description |
|---|---|---|
| `observation_utility_analysis_method_v10_sql.json` | `analysis_method` | Method used for chemical or physical analysis of a sample |
| `observation_utility_apparatus_v10_sql.json` | `apparatus` | Any instrument, tool, laboratory, or service that delivers observation data |
| `observation_utility_classification_v10_sql.json` | `order`, `family`, `genus`, `species`, `specimen` | Five-level Linnean substance classification hierarchy |
| `observation_utility_image_v10_sql.json` | `image_approach`, `image_substrate`, `image_camera`, `image_setup`, `image_lightsource` | Image acquisition metadata |
| `observation_utility_license_v10_sql.json` | `license` | Data license attached to a dataset or campaign |
| `observation_utility_location_method_v10_sql.json` | `location_method` | Method used to determine geolocation (e.g. GPS, map reading) |
| `observation_utility_macrofauna_v10_sql.json` | `macrofauna_extraction_method`, `life_cycle_stage` | Methods and life stages for macrofauna sampling |
| `observation_utility_method_tier_v10_sql.json` | `method_tier` | Level of professionality (in-situ, lab, drone, satellite, document, etc.) |
| `observation_utility_microbiometer_version_v10_sql.json` | `microbiometer_version` | Microbiometer kit version |
| `observation_utility_monolith_extraction_v10_sql.json` | `monolith_extraction` | Method for extracting soil, sediment or ice core monoliths |
| `observation_utility_preparation_v10_sql.json` | `preparation` | Sample preparation method prior to analysis |
| `observation_utility_preservation_v10_sql.json` | `preservation` | Sample preservation method (cold, frozen, chemical shield, etc.) |
| `observation_utility_provider_v10_sql.json` | `provider` | Any tool, instrument, sensing service, or data provider delivering results |
| `observation_utility_quantity_v10_sql.json` | `quantity` | Physical quantities that can be observed (e.g. pH, organic carbon) |
| `observation_utility_reference_proprietor_v10_sql.json` | `reference_proprietor` | Proprietor of a reference standard |
| `observation_utility_setting_system_v10_sql.json` | `setting_system` | Thematic frame for defining local juxtapositions (field, forest, lake, etc.) |
| `observation_utility_soil_horizon_v10_sql.json` | `soil_horizon` | Standard soil horizon designations |
| `observation_utility_sound_v10_sql.json` | `sound_setup`, `sound_mic` | Sound recording setup and microphone types |
| `observation_utility_spatial_reference_v10_sql.json` | `spatial_reference` | Spatial reference systems for geolocations |
| `observation_utility_spectroscopy_method_v10_sql.json` | `spectroscopy_method` | Type of spectral analysis method |
| `observation_utility_storage_v10_sql.json` | `storage` | Sample storage conditions before analysis |
| `observation_utility_taxa_level_v10_sql.json` | `taxa_level` | Linnean classification levels |
| `observation_utility_taxa_status_v10_sql.json` | `taxa_status` | Status flags for outdated or redundant taxa definitions |
| `observation_utility_transportation_v10_sql.json` | `transportation` | Transportation conditions for samples before analysis |
| `observation_utility_unit_v10_sql.json` | `unit` | Units of reported observation values |

## Process files — tables with internal dependencies

These tables reference other `observation_utility` tables and must be created after their dependencies.

| File | Tables created | Dependencies |
|---|---|---|
| `observation_utility_analysis_method_translate_v10_sql.json` | `analysis_method_translate` | `species`, `analysis_method` |
| `observation_utility_indicator_v10_sql.json` | `indicator` | `quantity` |
| `observation_utility_indicator_default_unit_v10_sql.json` | `indicator_default_unit` | `species`, `indicator`, `unit` |
| `observation_utility_juxtaposition_v10_sql.json` | `juxtaposition` | `setting_system` |
| `observation_utility_profiling_v10_sql.json` | `profiling` | `unit` (depth profiling for cores) |
| `observation_utility_provision_v10_sql.json` | `provision`, `provision_method_tier`, `provision_serial_nr`, `provision_calibration` | `apparatus`, `provider`, `method_tier` — combines these three into a single instrument+service+tier record, plus companion tables for method tier flags, serial numbers, and calibration |
| `observation_utility_provision_indicator_v10_sql.json` | `provision_indicator` | `provision`, `indicator`, `analysis_method`, `unit` — the tangible observation values delivered by a specific provision |
| `observation_utility_spectrometer_v10_sql.json` | `spectrometer` | `provision`, `provision_serial_nr` — registers a spectral sensor with its full wavelength axis array |
| `observation_utility_quantity_default_unit_v10_sql.json` | `quantity_default_unit` | `quantity`, `species`, `unit` |
| `observation_utility_taxa_v10_sql.json` | `taxa` | `taxa_level`, `taxa_status` |
| `observation_utility_taxa_function_v10_sql.json` | `taxa_function` | `taxa_level`, `taxa_status` |
| `observation_utility_unit_translate_v10_sql.json` | `unit_translate` | `unit` — mathematical conversion factors between units |

## Key concepts

**Provision** is the central linking concept in observation_utility. A provision combines an `apparatus` (what instrument/tool), a `provider` (what service or supplier), and a `method_tier` (what level of professionality). A `provision_indicator` then specifies exactly which measurable quantities a provision delivers, with associated analysis method and unit. Every observation in the `observation` schema links back to a provision.

**Indicator** represents a single measurable result (e.g. soil pH, organic carbon %). Indicators belong to a `quantity` (the physical property type) and have a default unit. The same indicator can be delivered by multiple provisions.

**Profiling** describes z-dimension sampling profiles (soil cores, sediment cores, ice cores) by specifying depth increments in a given unit. Samples with a profile dimension reference a profiling record.
