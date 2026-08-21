---
title: "Schema Conventions: Array Columns vs. Junction Tables"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "FK-shaped array columns have no referential integrity in Postgres. As of 2026-08-15, they've been replaced with junction tables everywhere except ordered numeric series like spectral data. Read this before adding a new multi-value field."
permalink: /setup_process/schema_conventions/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

This page documents a schema-design convention, not an end-user workflow — it's aimed at whoever (human or Claude) next adds a multi-value field to the AI4SH schema. If you're looking for how to load data day to day, the rest of this site's collections cover that; this page is about how the schema itself is shaped.

## The problem

An array column typed like a foreign key reference — `INTEGER[]`, or the more surprising `BOOLEAN[]` — has **zero referential integrity** in Postgres. Postgres can validate a scalar foreign key column against another table, but it cannot constrain individual elements *inside* an array. Nothing stops an invalid id from sitting silently inside `main_crop_id`, `catch_crop_id`, `cover_crop_id`, or the mistyped `soil_preparation_id_array` (a `BOOLEAN[]` column *named* like an id reference, whose real intent was a checkbox set — see below).

## The fix: junction tables

As of 2026-08-15, every FK-shaped array column in the schema was replaced with a junction table: one row per `(parent id, referenced id)` pair, with a real `REFERENCES` foreign key. Postgres now rejects an invalid reference outright instead of silently storing it.

## The boolean-checkbox special case

`soil_preparation_id_array` was a `BOOLEAN[]` meant to represent a checkbox set — which soil preparation methods applied to a given cultivation. A junction table is strictly better here too, and simpler than it sounds: a row's *existence* is the checked box. `landscape.cultivation_soil_preparation` has no boolean column at all — just `cultivation_id` and `soil_preparation_id`. If the pair exists, the box is checked; if not, it isn't.

## Free-text tags get the same treatment, for a different reason

`campaign.substance_array` and `campaign.keyword_array` (and the equivalent columns on `dataset`) were confirmed free text with no lookup table to enforce — there's no referential-integrity problem to fix. They were normalized into junction tables anyway, purely for indexing and querying: `campaign_tag`/`dataset_tag`, with a `tag_type` column (`substance`/`keyword`) distinguishing the two former array columns that now share one table.

## What's deliberately excluded, and why

Not every array column was converted. **Ordered numeric series** stay as Postgres arrays: `observation.observation_measurement_array`, and the spectral-data columns `observation_utility.spectrometer.wavelength_array`, `observation.spectra_scan.signal_mean`, `observation.spectra_scan.signal_standard_deviation`.

The reasoning is the same in both cases: these aren't a set of FK references or independent tags, they're an ordered sequence where position carries meaning (the Nth wavelength value corresponds to the Nth signal value). A junction table has no natural row order — you'd need an explicit sequence column and would lose the "array preserves order for free" property a Postgres array gives you. Converting these would add complexity for no referential-integrity gain, since there's nothing to reference.

This finishes a rollout the project's own `indicator` → `indicator_parity` trial started, a pattern already proven live elsewhere in the schema: `campaign_setting_system`, `dataset_setting_system`, `sampling_log_setting_system`, `campaign_provision`, `taxa_parity`.

## The four new tables

| Table | Replaces | Key columns | Uniqueness |
|---|---|---|---|
| `landscape.cultivation_crop` | `main_crop_id`/`catch_crop_id`/`cover_crop_id INTEGER[]` on `landscape.cultivation` | `cultivation_id`, `taxa_id` (→ `observation_utility.taxa`), `role` (`main`/`catch`/`cover`) | `(cultivation_id, taxa_id, role)` |
| `landscape.cultivation_soil_preparation` | `soil_preparation_id_array BOOLEAN[]` | `cultivation_id`, `soil_preparation_id` (→ `landscape_utility.soil_preparation`) | `(cultivation_id, soil_preparation_id)` |
| `observation.campaign_tag` | `substance_array`/`keyword_array TEXT[]` on `observation.campaign` | `campaign_id`, `tag`, `tag_type` (`substance`/`keyword`) | `(campaign_id, tag, tag_type)` |
| `observation.dataset_tag` | `substance_array`/`keyword_array TEXT[]` on `observation.dataset` | `dataset_id`, `tag`, `tag_type` (`substance`/`keyword`) | `(dataset_id, tag, tag_type)` |

`landscape.cultivation_crop` unifies all three former crop-role array columns into a single table via the `role` discriminator column, rather than three separate junction tables. This `role`/`tag_type` discriminator shape recurs twice in this migration (also in `campaign_tag`/`dataset_tag`) — it's the pattern to reach for whenever several former array columns share one target type.

One thing that breaks a convention you may already know from elsewhere on this site: `landscape.cultivation`'s natural key isn't a name lookup, it's geolocation + `season_start_at`. So unlike most `..._id__..._name` parameters documented elsewhere in this framework, `cultivation_id` on `manage_cultivation_crop`/`manage_cultivation_soil_preparation` takes a raw integer id, not a lookup-by-name string.

## Registering and using the new processes

Each new table gets its own independent `manage_*` process via `add_process` — `manage_cultivation_crop`, `manage_cultivation_soil_preparation`, `manage_campaign_tag`, `manage_dataset_tag` — all under `root_process: manage_table_data` (see [Root Processes][setup_process_root_process]), `min_user_stratum: 4`.

**Why independent processes, not the existing array-parameter fan-out mechanism.** `src/ai4sh/import_data/import_data.py`'s `_Define_specifics` already has code that infers a target junction table from a parameter name ending in `_array` and containing `__`, splitting it into multiple records automatically. It's flagged `# TG TODO` in the source and infers its target column via fragile string-splitting — not a mechanism worth adding new load-bearing usage to. This is a deliberate design choice, worth keeping documented here so a future reader doesn't "helpfully" wire a new array-shaped field through that mechanism instead of writing a proper `manage_*` process.

**Pilot-list wiring**, for a reader replicating this pattern for a new column: in `ai4sh_setup_processes.txt`, the two `*_tag_v10_sql.json` files sit in the existing observation section, right after `sampling_log_v10_sql.json`. The two `cultivation_*_v10_sql.json` files got a **new** `### landscape ###` section, since `setup_processes` previously had no landscape-schema entries at all.

## Audit coverage

All four new tables audit `UPDATE`/`DELETE` only, matching the existing rule for their schema group — `landscape` and `observation` are bulk pipeline-written data, not admin/catalogue data. See [Auditing][auditing_introduction] for the full two-tier coverage rule; it isn't repeated here.

## Verification

This was a schema-only change — zero live rows existed in any of the replaced array columns, so no data migration was needed. Verified with a full `delete_db` → `setup_db` → `setup_processes` cycle, real test rows through each new process (correct FK resolution by name), and a deliberately bad lookup rejected cleanly with no row inserted.

[setup_process_root_process]: /setup_process/root_process/
[auditing_introduction]: /auditing/
