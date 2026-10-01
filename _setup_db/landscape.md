---
title: "Landscape Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The landscape and landscape_utility schemas store landscape-level observations that provide environmental context for soil measurements. Two schemas are used: landscape_utility for reference catalogues and landscape for the observations themselves."
permalink: /setup_db/landscape/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-10-01 08:00:00 +0200
---

The AI4SH database uses two schemas for landscape data: `landscape_utility` holds reference catalogues and `landscape` holds the observations. Landscape observations provide the environmental context for soil measurements — land use, land cover, terrain attributes and similar properties that describe the broader setting of a sampling site.

## Process files

The files are split over two folders under `./setup/zzz/ai4sh/setup_db/json/`, one per schema. The pilot file `db_setup.txt` runs the `landscape_utility/` files first, in a `LANDSCAPE UTILITY` section, and then the `landscape/` files.

### landscape_utility/

| File | Tables created | In pilot |
|---|---|---|
| `crop_growth_stage_v10_sql.json` | `crop_growth_stage` | yes |
| `biogeo16_v10_sql.json` | `biogeo16_eu` (biogeographical regions) | yes |
| `land_use_v10_sql.json` | `landuse_order`, `landuse_family` (→ `landuse_order`), `landuse_genus` (→ `landuse_family`) | yes |
| `land_cover_v10_sql.json` | `landcover_order`, `landcover_family` (→ `landcover_order`), `landcover_genus` (→ `landcover_family`) | yes |
| `soil_classification_v10_sql.json` | `soil_texture_usda`, `soil_texture_isss`, `reference_soil_group` | yes |
| `major_landform_v10_sql.json` | `major_landform` | yes |
| `slope_position_v10_sql.json` | `slope_position` | yes |
| `sky_conditions_v10_sql.json` | `sky_condition` | yes |
| `ground_conditions_v10_sql.json` | `ground_condition` | yes |
| `soil_preparation_v10_sql.json` | `soil_preparation` | yes |
| `soil_texture_v10_sql.json` | `soil_texture_usda`, `soil_texture_isss` | no — superseded by `soil_classification_v10_sql.json` |

### landscape/

| File | Schema | Tables created | In pilot |
|---|---|---|---|
| `observation_lucc_v10_sql.json` | `landscape` | `landuse_observation`, `landcover_observation` | yes |
| `observation_v10_sql.json` | `landscape` | `sampling_weather`, `cultivation`, `cultivation_crop` (→ `cultivation`), `cultivation_soil_preparation` (→ `cultivation`), `landscape_state`, `landscape_geomorphology` | no — disabled (`### REMOVE TO RUN ###`) |
| `utility_v10_sql.json` | `landscape_utility` | `cultivation_species`, `crop_growth_stage`, `erosion_conservation_measure` | no |

Files marked *no* define tables that are prepared but not created by `setup_db.ipynb`. To enable the remaining landscape observations, `erosion_conservation_measure` (from `landscape/utility_v10_sql.json`) must be created before `observation_v10_sql.json`, because `landscape_state` references it.

## Schema: landscape_utility

The `landscape_utility` schema holds the reference catalogues for landscape classification — 16 tables created by the 10 files in the pilot. They are analogous to the `observation_utility` tables but specific to landscape-level attributes. Every table has `id`, `name`, `alias`, `display_name` and `abstract`:

- **land use** — `landuse_order` → `landuse_family` → `landuse_genus` (3-level hierarchy)
- **land cover** — `landcover_order` → `landcover_family` → `landcover_genus` (3-level hierarchy)
- **soil classification** — `soil_texture_usda`, `soil_texture_isss`, `reference_soil_group`
- **general landscape attributes** — `major_landform`, `slope_position`, `sky_condition`, `ground_condition`, `soil_preparation`, `crop_growth_stage`, `biogeo16_eu`

The reference tables in `landscape_utility` must be populated before any `landscape` observation records can be inserted, because landscape observations reference these catalogues through foreign keys.

[![Landscape utility schema]({{ "/assets/media/landscape/landscape_utility.png" | relative_url }})]({{ "/assets/media/landscape/landscape_utility.png" | relative_url }})

## Schema: landscape

The `landscape` schema stores landscape observations. Two tables are created by the current pilot:

- **landuse_observation**, **landcover_observation** — land use and land cover at a geolocation and time, each linking to `observation.geolocation`, `observation.sampling_log` and its `_genus` catalogue (`landuse_genus` or `landcover_genus`); unique per geolocation and `observed_at`

Six more tables are defined in `observation_v10_sql.json` but not yet created (see above):

- **sampling_weather** — sky and ground condition, precipitation and temperature at the time of sampling
- **cultivation**, **cultivation_crop**, **cultivation_soil_preparation** — the cultivation season at a geolocation, its crops (as `organism_utility.taxon` with role main, catch or cover) and soil preparation
- **landscape_state** — vegetation presence, crop growth stage, erosion and erosion/conservation measures, surface cover
- **landscape_geomorphology** — slope, aspect, major landform and slope position

These observations help contextualise soil property measurements — the same soil property can behave differently under different land use or cover conditions.

[![Landscape schema]({{ "/assets/media/landscape/landscape.png" | relative_url }})]({{ "/assets/media/landscape/landscape.png" | relative_url }})

## Relationship to observation schema

Every `landscape` table above links directly to `observation.geolocation` via `geolocation_id` — not to a sample. (The land use and land cover observations also record the `sampling_log` during which they were made.) A sampling location may have both soil observations in the `observation` schema and landscape observations in the `landscape` schema, connected through that shared geolocation. A major reason for this is that historical landscape changes, such as land use and land cover shifts over time, can affect soil properties. To retrieve such historical changes landscape recording is tied to locations with the location linking to the sample.

This separation also keeps AI4SH-specific landscape classification independent of the more generic observation infrastructure, making it easier to extend or replace the landscape classification system without affecting core soil data tables.
