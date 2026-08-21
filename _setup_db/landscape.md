---
title: "Landscape Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The landscape and landscape_utility schemas store landscape-level observations that provide environmental context for soil measurements. Two schemas are used: landscape_utility for reference catalogues and landscape for the observations themselves."
permalink: /setup_db/landscape/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The AI4SH database uses two schemas for landscape data: `landscape_utility` holds reference catalogues and `landscape` holds the observations. Landscape observations provide the environmental context for soil measurements — land use, land cover, terrain attributes and similar properties that describe the broader setting of a sampling site.

## Process files

All 6 files live under `json_ai4sh/landscape/` — there is no separate `landscape_utility/`
folder; files in this one folder create tables in both schemas:

| File | Schema | Tables created |
|---|---|---|
| `landscape/land_use_v10_sql.json` | `landscape_utility` | `landuse_order`, `landuse_family` (→ `landuse_order`), `landuse_genus` (→ `landuse_family`) |
| `landscape/land_cover_v10_sql.json` | `landscape_utility` | `landcover_order`, `landcover_family` (→ `landcover_order`), `landcover_genus` (→ `landcover_family`) |
| `landscape/soil_classification_v10_sql.json` | `landscape_utility` | `soil_texture_usda`, `soil_texture_isss`, `reference_soil_groups` |
| `landscape/utility_v10_sql.json` | `landscape_utility` | `major_landform`, `slope_position`, `sky_condition`, `ground_condition`, `cultivation_species`, `soil_preparation`, `crop_growth_stage`, `erosion_conservation_measure` |
| `landscape/observation_lucc_v10_sql.json` | `landscape` | `landuse_observation`, `landcover_observation` — both link to `observation.geolocation` and their respective `_genus` catalogue |
| `landscape/observation_v10_sql.json` | `landscape` | `sampling_weather`, `cultivation`, `cultivation_crop` (→ `cultivation`), `cultivation_soil_preparation` (→ `cultivation`), `landscape_state`, `landscape_geomorphology` |

## Schema: landscape_utility

The `landscape_utility` schema holds reference tables for landscape classification — 14 tables
across the 4 files above. These are analogous to the `observation_utility` tables but specific
to landscape-level attributes:

- **land use** — `landuse_order` → `landuse_family` → `landuse_genus` (3-level hierarchy)
- **land cover** — `landcover_order` → `landcover_family` → `landcover_genus` (3-level hierarchy)
- **soil classification** — `soil_texture_usda`, `soil_texture_isss`, `reference_soil_groups`
- **general landscape attributes** — `major_landform`, `slope_position`, `sky_condition`, `ground_condition`, `cultivation_species`, `soil_preparation`, `crop_growth_stage`, `erosion_conservation_measure`

The reference tables in `landscape_utility` must be populated before any `landscape` observation records can be inserted, because landscape observations reference these catalogs through foreign keys.

## Schema: landscape

The `landscape` schema stores actual landscape observations — 8 tables across the 2
observation files above:

- **landuse_observation**, **landcover_observation** — land use/cover at a geolocation, each linking to its respective `_genus` catalogue
- **sampling_weather** — sky and ground condition at the time of sampling
- **cultivation**, **cultivation_crop**, **cultivation_soil_preparation** — what's being cultivated at a geolocation, which crop taxa, and what soil preparation was used
- **landscape_state** — crop growth stage and erosion/conservation measures observed
- **landscape_geomorphology** — landform, slope position, and slope value at a geolocation

These observations help contextualise soil property measurements — the same soil property can behave differently under different land use or cover conditions.

## Relationship to observation schema

Every `landscape` table above links directly to `observation.geolocation` via `geolocation_id`
— not via `sampling_log` as such. A sampling location may have both soil observations in the
`observation` schema and landscape observations in the `landscape` schema, connected through
that shared geolocation.

This separation keeps AI4SH-specific landscape classification independent of the more generic observation infrastructure, making it easier to extend or replace the landscape classification system without affecting core soil data tables.
