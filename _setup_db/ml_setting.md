---
title: "ML Setting and Model Schemas"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The ml_setting schema holds the machine learning parameter sets, the ml_model schema the registry of final models. Tables, the 24 seed sets and the triggers that protect frozen sets."
permalink: /setup_db/ml_setting/
author_profile: false
date: 2026-10-26 08:00:00 +0200
last_modified_at: 2026-10-26 08:00:00 +0200
---

Two schemas support machine learning. `ml_setting` holds the named, versioned **parameter sets** that every
ML step reads (transformations, filters, regressor hyper-parameters, plot symbols). `ml_model` is the
**registry** of final models. How they are used is described in [ML settings][ml_settings] and
[Predict and register][ml_predict]; this page covers the tables.

Until `ml_setting` exists, the ML processes fall back to the settings files in `ai4sh/default/`, with a
warning. Once it exists, those files are no longer read.

## Process files

All files are in `./setup/zzz/ai4sh/setup_db/json/`. The pilot file `db_setup.txt` runs them last, in the
sections `MACHINE LEARNING SETTINGS` and `MACHINE LEARNING MODEL REGISTRY`:

| File | Content |
|---|---|
| `ml_setting/ml_setting_v10_sql.json` | Tables `transform` (seeded), `parameter_set`, `indicator_transformation`, `indicator_standardisation` |
| `ml_setting/ml_setting_records_v10_sql.json` | The 24 seed parameter sets |
| `ml_setting/ml_setting_frozen_v10_sql.json` | Trigger functions and `BEFORE UPDATE OR DELETE` triggers that protect frozen sets |
| `ml_model/ml_model_v10_sql.json` | Tables `model`, `model_step`, `model_parameter_set`, `model_metric` |

Dependencies: `observation_utility.indicator` (foreign keys from both schemas) and `community.user` (the
creator of a custom set and who registered a model) must exist first.

## ml_setting

| Table | Columns | Notes |
|---|---|---|
| `transform` | `id`, `name` (unique), `abstract` | Seeded: `0 linear`, `1 log`, `2 sqrt`, `3 reciprocal`, `4 boxcox`, `5 yeojohnson`, `6 quantile` |
| `parameter_set` | `id`, `family`, `name`, `version` (default 1), `category` (`raw` / `preset` / `custom`), `creator`, `created_at`, `abstract`, `frozen` (default false), `content` (JSONB), `content_hash` | Unique (`family`, `name`, `version`) |
| `indicator_transformation` | `id`, `set_id` → `parameter_set`, `indicator_id` → `observation_utility.indicator`, `transform_id` → `transform` (default 0) | One row per set and indicator |
| `indicator_standardisation` | `id`, `set_id` → `parameter_set`, `indicator_id` → `observation_utility.indicator`, `standardise` (default false) | One row per set and indicator |

The per-indicator rows are not seeded by `setup_db`. They are written by `sync_indicator_parameter_sets`,
and automatically after every `manage_indicator` (see [Observation utility][setup_db_observation_utility]).

### The 24 seed sets

All seed sets are version 1 with creator `setup`.

| Family | Sets |
|---|---|
| `transformation` | `none` (raw), `standard` |
| `standardisation` | `none` (raw), `standard` |
| `chemometric` | `none` (raw), `standard` |
| `filter` | `none` (raw), `moving-average`, `gauss`, `savitzky-golay`, `lowess` |
| `multi_filter` | `standard` |
| `selector` | `none` (raw), `standard` |
| `regressor` | `none` (raw, scikit-learn defaults), `standard` |
| `spectra_plot` | `none` (raw, viridis), `standard` (jet) |
| `plot_symbol` | `targetfeaturesymbols`, `regressionmodelsymbols` |
| `plot_layout` | `enhancementplotlayout`, `modelplot`, `modelplotlayout`, `outliersplotlayout` (not used by the code yet) |

Sets without "(raw)" are `preset`.

### Frozen sets

Registering a model sets `frozen = true` on every parameter set it used. Two trigger functions then guard
the data:

| Function | Trigger on | Refuses |
|---|---|---|
| `ml_setting.block_frozen_set()` | `parameter_set` (`parameter_set_frozen`) | Changing `content`, `content_hash`, `family`, `name`, `version` or `category` of a frozen set, unfreezing it, and deleting it. The `abstract` can still be edited |
| `ml_setting.block_frozen_rows()` | `indicator_transformation`, `indicator_standardisation` (`…_frozen`) | Update and delete of rows belonging to a frozen set |

Inserts are allowed, so a new indicator still gets its row in a frozen set. The error names the set:

```
parameter set transformation/standard@1 is frozen (used by a registered model) and can not be deleted
```

The triggers act on every client: notebook, `psql` or DBeaver.

## ml_model

| Table | Columns | Notes |
|---|---|---|
| `model` | `id`, `name` (unique), `indicator_id` → `observation_utility.indicator`, `regressor`, `mode`, `signal_type`, `n_bands`, `n_train`, `n_test`, `model_fp`, `model_sha256` (unique), `source_parquet`, `source_sha256`, `manifest` (JSONB), `status` (`registered` / `retired`), `abstract`, `registered_by` (default `current_user`), `registered_at` | One row per registered model |
| `model_step` | `id`, `model_id` → `model`, `step_no`, `step_id`, `process`, `parameters` (JSONB) | The preprocessing chain; unique (`model_id`, `step_no`) |
| `model_parameter_set` | `id`, `model_id` → `model`, `set_id` → `ml_setting.parameter_set`, `used_by` | Which sets the model used, and in which step |
| `model_metric` | `id`, `model_id` → `model`, `dataset`, `metric`, `value`, `n` | Test, k-fold and `independent:<parquet>` metrics; unique (`model_id`, `dataset`, `metric`) |

Child rows are deleted with their model (`ON DELETE CASCADE`), but a model is meant to be retired, not
deleted.

## Auditing

All eight tables are audited for `INSERT`, `UPDATE` and `DELETE` (see [Auditing][auditing]).

[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
[setup_db_observation_utility]: {{ site.baseurl }}/setup_db/observation_utility/
[auditing]: {{ site.baseurl }}/auditing/
