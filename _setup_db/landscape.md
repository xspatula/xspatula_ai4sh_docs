---
title: "Landscape Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The landscape and landscape_utility schemas store landscape-level observations that provide environmental context for soil measurements. Two schemas are used: landscape_utility for reference catalogues and landscape for the observations themselves."
permalink: /setup_db/landscape/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-10-17 08:00:00 +0200
---

The AI4SH database uses two schemas for landscape data: `landscape_utility` holds reference catalogues and `landscape` holds the observations. Landscape observations provide the environmental context for soil measurements — land use, land cover, terrain attributes and similar properties that describe the broader setting of a sampling site.

## Process files

The files are split over two folders under `./setup/zzz/ai4sh/setup_db/json/`, one per schema. The pilot file `db_setup.txt` runs the `landscape_utility/` files first, in a `LANDSCAPE UTILITY` section, and then the `landscape/` files.

### landscape_utility/

| File | Tables created | In pilot |
|---|---|---|
| `crop_growth_stage_v10_sql.json` | `crop_growth_stage` | yes |
| `biogeo16_v10_sql.json` | `biogeo16` (biogeographical regions) | yes |
| `class_rank_v10_sql.json` | `class_rank` (seeded with `order`, `family`, `genus`) | yes |
| `land_use_v10_sql.json` | `landuse` (all ranks, → `class_rank`, self reference to the parent class) | yes |
| `land_cover_v10_sql.json` | `landcover` (all ranks, → `class_rank`, self reference to the parent class) | yes |
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
| `observation_biogeo16_v10_sql.json` | `landscape` | `biogeo16_observation` | yes |
| `observation_v10_sql.json` | `landscape` | `sampling_weather`, `cultivation`, `cultivation_crop` (→ `cultivation`), `cultivation_soil_preparation` (→ `cultivation`), `landscape_state`, `landscape_geomorphology` | no — disabled (`### REMOVE TO RUN ###`) |
| `utility_v10_sql.json` | `landscape_utility` | `cultivation_species`, `crop_growth_stage`, `erosion_conservation_measure` | no |

Files marked *no* define tables that are prepared but not created by `setup_db.ipynb`. To enable the remaining landscape observations, `erosion_conservation_measure` (from `landscape/utility_v10_sql.json`) must be created before `observation_v10_sql.json`, because `landscape_state` references it.

## Schema: landscape_utility

The `landscape_utility` schema holds the reference catalogues for landscape classification — 13 tables created by the 11 files in the pilot. They are analogous to the `observation_utility` tables but specific to landscape-level attributes:

- **class_rank** — the ranks of the hierarchical classifications: `order` (alias `ordo`, id 1), `family` (`familia`, 2) and `genus` (`genus`, 3). Seeded by setup, so nothing needs to be imported.
- **landuse**, **landcover** — one table per scheme holding the classes of **all** ranks, modelled on [`organism_utility.taxon`][organism_utility]. Each class has `parent_landuse_id` / `parent_landcover_id` (a self reference, NULL for an order), `class_rank_id`, `name`, `alias` (the class code, unique), `tag` (unique), `display_name` and `abstract`. A name must be unique under its parent (`UNIQUE NULLS NOT DISTINCT (parent_<scheme>_id, name)`).
- **soil classification** — `soil_texture_usda`, `soil_texture_isss`, `reference_soil_group`
- **general landscape attributes** — `major_landform`, `slope_position`, `sky_condition`, `ground_condition`, `soil_preparation`, `crop_growth_stage`, `biogeo16` (with a unique `tag`)

Why one ranked table instead of three: land cover and land use are often recorded only at family or order rank (about half of the LUCAS 2009 records). The earlier `_order` → `_family` → `_genus` tables could only be referenced at genus rank, so coarser records had to be faked by copying the higher class down. With one ranked table an observation references the class at whatever rank it was recorded.

The reference tables in `landscape_utility` must be populated before any `landscape` observation records can be inserted, because landscape observations reference these catalogues through foreign keys.

**Diagram out of date:** the figure below still shows the separate `_order`/`_family`/`_genus` tables and `biogeo16_eu`. It will be redrawn.
{: .notice--warning}

[![Landscape utility schema]({{ "/assets/media/landscape/landscape_utility.png" | relative_url }})]({{ "/assets/media/landscape/landscape_utility.png" | relative_url }})

## Schema: landscape

The `landscape` schema stores landscape observations. Three tables are created by the current pilot:

- **landuse_observation**, **landcover_observation** — land use and land cover at a geolocation and time, each linking to `observation.geolocation`, `observation.observation_log` and to a class in `landscape_utility.landuse` or `landcover` **at any rank** (order, family or genus); unique per geolocation and `observed_at`
- **biogeo16_observation** — the biogeographical region of a geolocation, linking to `observation.geolocation`, `observation.observation_log` and `landscape_utility.biogeo16`; unique per geolocation and `observed_at`

Six more tables are defined in `observation_v10_sql.json` but not yet created (see above):

- **sampling_weather** — sky and ground condition, precipitation and temperature at the time of sampling
- **cultivation**, **cultivation_crop**, **cultivation_soil_preparation** — the cultivation season at a geolocation, its crops (as `organism_utility.taxon` with role main, catch or cover) and soil preparation
- **landscape_state** — vegetation presence, crop growth stage, erosion and erosion/conservation measures, surface cover
- **landscape_geomorphology** — slope, aspect, major landform and slope position

These observations help contextualise soil property measurements — the same soil property can behave differently under different land use or cover conditions.

**Diagram out of date:** the figure below still links the observations to `sampling_log` and the `_genus` tables, and lacks `biogeo16_observation`. It will be redrawn.
{: .notice--warning}

[![Landscape schema]({{ "/assets/media/landscape/landscape.png" | relative_url }})]({{ "/assets/media/landscape/landscape.png" | relative_url }})

## Relationship to observation schema

Every `landscape` table above links directly to `observation.geolocation` via `geolocation_id` — not to a sample. (The land use, land cover and biogeo16 observations also record the `observation_log` they belong to, and through it the sampling log and provision.) A sampling location may have both soil observations in the `observation` schema and landscape observations in the `landscape` schema, connected through that shared geolocation. A major reason for this is that historical landscape changes, such as land use and land cover shifts over time, can affect soil properties. To retrieve such historical changes landscape recording is tied to locations with the location linking to the sample.

This separation also keeps AI4SH-specific landscape classification independent of the more generic observation infrastructure, making it easier to extend or replace the landscape classification system without affecting core soil data tables.

[organism_utility]: {{ site.baseurl }}/setup_db/organism_utility/
