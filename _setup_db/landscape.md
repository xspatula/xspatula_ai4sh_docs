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

| File | Schema | Purpose |
|---|---|---|
| `landscape/landscape_utility_v10_sql.json` | `landscape_utility` | Creates landscape reference catalogue tables |
| `landscape/landscape_observation_v10_sql.json` | `landscape` | Creates landscape observation tables |

## Schema: landscape_utility

The `landscape_utility` schema holds reference tables for landscape classification. These are analogous to the `observation_utility` tables but specific to landscape-level attributes. Typical content includes land use classes, land cover classifications, and terrain descriptors.

The reference tables in `landscape_utility` must be populated before any `landscape` observation records can be inserted, because landscape observations reference these catalogs through foreign keys.

## Schema: landscape

The `landscape` schema stores actual landscape observations linked to sampling locations. A landscape observation typically characterises the environment at or around a sampling point with attributes such as:

- Land use type
- Land cover class
- Terrain position
- Vegetation cover

These observations help contextualise soil property measurements — the same soil property can behave differently under different land use or cover conditions.

## Relationship to observation schema

Landscape observations are linked to the same sampling infrastructure as soil observations (sampling logs, campaigns, datasets). A sampling location may have both soil observations in the `observation` schema and landscape observations in the `landscape` schema, connected via the shared sampling log.

This separation keeps AI4SH-specific landscape classification independent of the more generic observation infrastructure, making it easier to extend or replace the landscape classification system without affecting core soil data tables.
