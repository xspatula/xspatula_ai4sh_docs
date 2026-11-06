---
title: "ML Modeling 3: Arrays and plot layouts"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Modeling many indicators or many regressors in one regression_modeling run, and plot layouts that compose the shown plots into grid figures: presets, the default rule, layout keys, partial and custom layout files."
permalink: /machine_learning/ml_model/plots/
author_profile: false
date: 2026-11-05 08:00:00 +0100
last_modified_at: 2026-11-06 12:00:00 +0100
---

In the JSON process file for `regression_modeling`, the parameters for `indicator_array` and `regressor_array` take any number of entries, and the modeling fits every
combination: 7 indicators and 1 regressor give 7 models per mode, 2 indicators and 4 regressors 8.

Section 3 of
the notebook `ml_model.ipynb` uses this to compare many models at a glance, with the plots composed into one
grid figure by a (custom) plot layout. With many indicators, regressors and k-fold, a run can take long.

## 3.1.1 Many indicators, one regressor

**Process file**: `project_neospectra/process/ml_model/regression_modeling_all-indicators_plsr_snv-d1-mc_kf5.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_olif_texture",
        "dataframe": "neospectra_10nm_olif_texture.snv.d1.mc",
        "indicator_array": ["cec", "clay", "ca2+", "ec", "n-tot", "c-org", "ph-water"],
        "covariate_array": ["soil texture usda"],
        "class_grouping": [
          {
            "covariate": "soil texture usda",
            "groups": [
              ["sand", "loamy sand", "sandy loam"],
              ["loam", "silt loam", "silt", "sandy clay loam"],
              ["clay loam", "silty clay loam", "sandy clay", "silty clay", "clay"]
            ]
          }
        ],
        "regressor_array": ["plsr"],
        "model_parameters_fp": "./project_neospectra/regressors/regression_model_parameters_tuned.json",
        "traintest": false,
        "kfold": true,
        "folds": 5,
        "plot_layout": "./project_neospectra/plot_layout/scatter_kf_4-per-row.json",
        "save_plot_layout": true
      }
    }
  ]
}
```

The seven indicators are every indicator of the compiled dataset; an empty `indicator_array` gives the same. The
layout file `project_neospectra/plot_layout/scatter_kf_4-per-row.json`:

```json
{
  "columns": ["scatter_kf"],
  "wrap": 4,
  "group_by": "indicator",
  "draw": "run",
  "panel_size": [2.6, 2.6],
  "max_units_per_row": 4
}
```

With one regressor the indicators flow four per row, so the seven scatter plots fill two rows:

![PLSR k-fold scatter plots of seven indicators]({{ "/assets/media/ml_model/3-1-1_all-indicators_plsr_kf5.png" | relative_url }})

The grid is saved as
`plot/neospectra_10nm_olif_texture.snv.d1.mc_cv-soiltextureusda_grid/scatter_kf_4-per-row.png`. A figure like this
shows quickly which indicators the spectra predict at all, and which need more work.

## 3.2.1 Many regressors, one indicator

**Process file**: `project_neospectra/process/ml_model/regression_modeling_many-regressors_c-org_snv-d1-mc_kf5.json`,
the same as above with `"indicator_array": ["c-org"]` and `"regressor_array": ["randfor", "cubist", "mlp", "plsr"]`,
and the same layout file. With several regressors the regressors of one indicator share a row:

![Four regressors for c-org, k-fold]({{ "/assets/media/ml_model/3-2-1_many-regressors_c-org_kf5.png" | relative_url }})

## Plot layout

The plots shown in the notebook are composed into grid figures by the layout named in `plot_layout`. The saved
plots (`save_observed_predicted`, `save_feature_importance`, `save_permutation_importance`) are always single
figures, one per indicator, regressor, mode and plot type. `save_plot_layout: true` also saves the grids, as
`plot/<name>_grid/<layout>.png` (`<layout>_<indicator>.png` when drawn per indicator).

| `plot_layout` | Meaning |
|---|---|
| `default` | A preset picked from what the process shows (see below) |
| a preset name | One of the presets in the table below |
| a set name (`name` or `name@version`) | A custom `plot_layout` set, see [ML settings][ml_settings] |
| a path (contains `/`, starts with `~` or ends with `.json`) | A JSON layout file, relative to the project root or absolute |
| `none` | No grid: every shown plot is a figure of its own |

A shown plot that has no column in the layout is shown as a single figure. The run prints the layout in use,
e.g. `Plot layout: importance_8-bars on importance (grid drawn per indicator)`.

### Presets

| Preset | Columns | Rows | Drawn |
|---|---|---|---|
| `scatter-1` … `scatter-4` | scatter, n units per row | one row per indicator, the regressors side by side; with one regressor the indicators flow | one grid at the end of the run |
| `scatter-tt-kf` | train/test and k-fold scatter side by side | one row per indicator and regressor | one grid at the end of the run |
| `importance` | scatter, feature importance (2 units wide), permutation importance (6 most important features) | one row per indicator and regressor; train/test and k-fold on separate rows when the row is wider than 4 units | one grid per indicator, as soon as it is done |

`default` picks the preset:

- feature or permutation importance shown → `importance`;
- only the scatter shown, train/test and k-fold → `scatter-tt-kf` (the figures of [2.2 Regressors][ml_model_regr]);
- only the scatter shown, one mode → `scatter-<n>`, with n the number of regressors, or the number of indicators
  when there is one regressor (at most 4; more wrap to new rows; the figures of [block 1][ml_model_pp]);
- nothing shown → no grid.

The presets are parameter sets of the family `plot_layout`, inserted by `setup_db` (see
[ML setting and model schemas][setup_db_ml]). Until they are in the database they are read from a default file,
with a warning:

```
    ⚠️  plot layout "importance" not in ml_setting.parameter_set (rerun setup_db) — taken from plotlayouts.json
```

### Layout keys

The `importance` preset:

```json
{
  "columns": ["scatter", "importance", "permutation"],
  "wrap": 0,
  "group_by": "indicator",
  "draw": "indicator",
  "panel_size": [3, 3],
  "max_units_per_row": 4,
  "importance_width": 2,
  "max_permutation_bars": 6
}
```

| Key | Description | Built-in default |
|---|---|---|
| `columns` | The panels of one row: `scatter`, `importance`, `permutation`, or with the mode `scatter_tt`, `scatter_kf`, `importance_tt`, `permutation_kf`, … | `["scatter"]` |
| `wrap` | `0`: one row per indicator and regressor. n > 0: the panels flow, n units per row; a new indicator starts a new row, unless there is one regressor | `0` |
| `group_by` | `indicator` or `regressor`: the outer loop of the rows. With `regressor`, indicators and regressors swap roles in `wrap` | `indicator` |
| `draw` | `indicator`: one grid per indicator, shown as soon as its models are fitted. `run`: one grid at the end of the run | `indicator` |
| `panel_size` | Width and height of one unit, in inches | `[3, 3]` |
| `max_units_per_row` | With `wrap: 0`, a wider row is split, first into train/test and k-fold | `4` |
| `importance_width` | Units of a feature importance panel; scatter and permutation panels are 1 unit | `2` |
| `max_permutation_bars` | Most important features in a permutation panel of the grid | `6` |
| `base` | Only in a file or custom set: the preset the other keys change (see below) | |

- **Units, not panels.** `wrap: 4` fits four scatter panels, or two feature importance panels, or one importance
  and two other panels.
- **Bare panel types** (`scatter`, `importance`, `permutation`) take every mode that ran: first all train/test
  panels, then the k-fold panels. Columns with a mode are taken in the listed order, and a mode that did not run is
  left out.
- **A panel that is not produced is left out**, not drawn empty, e.g. feature importance for `mlp` (see
  [Feature importance][ml_model_imp]).
- **`draw: run` delays the output.** Nothing is shown until every model is fitted, and every panel's data is held
  in memory until then. Use `indicator` for long runs.
- `max_permutation_bars` applies to the grid only; a saved permutation figure shows `n_top_features_in_plot`
  features.

An unknown panel type, a `group_by` other than `indicator`/`regressor` or a `draw` other than `indicator`/`run`
stops the process before any model is fitted, e.g.:

```
    ERROR: plot layout "scatter_kf_4-per-row": draw must be indicator or run
```

### Layout files

Keep layout files in the project, e.g. `project_neospectra/plot_layout/`, and give the path in `plot_layout`. A
file only needs the keys it changes. What the other keys are depends on what the file holds:

| The file has | The other keys come from |
|---|---|
| `base` | The preset named by `base`, e.g. `{"base": "importance", "max_permutation_bars": 8}` |
| neither `base` nor `columns`/`wrap` | The preset `default` would pick, e.g. `{"panel_size": [2.5, 2.5]}` keeps the picked preset with smaller panels |
| `columns` or `wrap`, no `base` | The built-in defaults in the table above, as in `scatter_kf_4-per-row.json` |

A file without any layout key stops the process:
`ERROR: plot layout "<name>" not found (no layout keys in it: base, columns, …)`.

### Layout sets

To keep a layout in the database, create a `custom` set with `create_parameter_set`, family `plot_layout`, a
preset as `base_set` (e.g. `importance`) and the changes in a JSON file of the same shape
([Create a custom set][ml_settings_custom]). The changes are merged into the base set.

## Next step

[4 Feature importance][ml_model_imp]: which features the models use.

[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[ml_settings_custom]: {{ site.baseurl }}/machine_learning/ml_settings/#create-a-custom-set
[setup_db_ml]: {{ site.baseurl }}/setup_db/ml_setting/
[ml_model_pp]: {{ site.baseurl }}/machine_learning/ml_model/preprocessing/
[ml_model_regr]: {{ site.baseurl }}/machine_learning/ml_model/regressors/
[ml_model_imp]: {{ site.baseurl }}/machine_learning/ml_model/importance/
