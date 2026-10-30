---
title: "ML Settings"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Machine learning settings as named, versioned parameter sets in the ml_setting schema: categories, families, how to name a set in a process file, new indicators, custom sets and frozen sets."
permalink: /machine_learning/ml_settings/
author_profile: false
date: 2026-10-26 08:00:00 +0200
last_modified_at: 2026-10-26 08:00:00 +0200
---

Every machine learning step reads settings: which transformation an indicator gets, the filter window,
the regressor hyper-parameters, the plot symbols. These settings live in the database as named, versioned
**parameter sets** (schema `ml_setting`, see [ML setting and model schemas][setup_db_ml]), not in loose
JSON files. Each run records exactly which set and version it used, so a model can always be traced back
to its settings.

**Settings no longer come from `ai4sh/default/`.** Those files are only a fallback until `setup_db` has
created the `ml_setting` schema.
{: .notice--warning}

## Notebook

**Path**: `xspatula_ai4sh/ai4sh/project_neospectra/ml_settings.ipynb`

Three cells: sync, list, create a custom set. Set `scheme_file = '../scheme_ai4sh.json'` in the setup cell
(the default).

## Categories

| Category | What it is | Example names |
|---|---|---|
| `raw` | No operation or library defaults (scikit-learn, matplotlib) | `none` |
| `preset` | Curated settings shipped with the project | `standard`, `gauss`, `savitzky-golay`, `targetfeaturesymbols` |
| `custom` | Your own settings, made with [create_parameter_set](#create-a-custom-set) (user stratum 4 and 5) | anything you choose |

## Families

| Family | Used by (process parameter) | Presets |
|---|---|---|
| `transformation` | `indicator_transformation` (modeling), `transformation` (plots) | `none`, `standard` |
| `standardisation` | `indicator_standardisation` (modeling), `standardisation` (plots) | `none`, `standard` |
| `chemometric` | `chemometrics` | `none`, `standard` |
| `filter` | `filter_fp` | `none`, `gauss`, `lowess`, `moving-average`, `savitzky-golay` |
| `multi_filter` | `multi_filter_fp` | `standard` |
| `selector` | band selection (permutation, RFE, tree based) | `none`, `standard` |
| `regressor` | `model_parameters_fp` | `none` (scikit-learn defaults), `standard` |
| `spectra_plot` | `colormap` | `none` (viridis), `standard` (jet) |
| `plot_symbol` | `targetfeaturesymbols`, `regressionmodelsymbols` | `targetfeaturesymbols`, `regressionmodelsymbols` |
| `plot_layout` | not used by the code yet | four layouts |

Together these are the 24 sets created by `setup_db`. Run `list_parameter_sets` to see every set with its
version, category, creator and description (`family` empty = all families):

**Process file**: `project_neospectra/process/ml_setting/list_parameter_sets.json`

```json
{
  "process": [
    { "process": "list_parameter_sets", "parameters": { "family": "" } }
  ]
}
```

## Naming settings in a process file

Every parameter in the table above accepts the same four forms:

| Value | Meaning |
|---|---|
| `"none"` (or `""`) | No settings: the step is skipped or uses the library defaults |
| `"default"` | The preset `standard` of that family (`targetfeaturesymbols` / `regressionmodelsymbols` for symbols, `jet` for `colormap`) |
| `"<name>"` or `"<name>@<version>"` | That parameter set; without `@` the latest version |
| a path (contains `/`, starts with `~` or ends with `.json`) | A JSON settings file, relative to the project root |

`colormap` also takes any matplotlib colour ramp name directly (`"viridis"`, `"plasma"`).

```json
"indicator_transformation": "standard",
"model_parameters_fp": "big_forest@2",
"filter_fp": "savitzky-golay",
"colormap": "default"
```

Until the `ml_setting` schema is set up, `"default"` and the preset filter names fall back to the files in
`ai4sh/default/`, with one warning per run
(`⚠️ ml_setting schema not available — using the default settings files`). A set name then fails with an
error.

**`"none"` means none.** For `model_parameters_fp`, `"none"` gives the scikit-learn defaults; use
`"default"` for the `standard` hyper-parameters (`random_state: 42` for the stochastic regressors).

## Indicator transformation and standardisation

These two families hold one value per indicator, as rows in `ml_setting.indicator_transformation` and
`ml_setting.indicator_standardisation`. The `standard` transformation uses `log` for skewed indicators
(`n-tot`, `ec`, `k-ext`, `coarse sand` and their long-name variants) and `linear` for the rest. Allowed
transformations: `linear`, `log`, `sqrt`, `reciprocal`, `boxcox`, `yeojohnson`, `quantile`.

The model fits the transformation on the training samples only and stores it with the model, so
predictions come back in the native unit (see [ML modeling][ml_model]). **Set `indicator_transformation`
and `indicator_standardisation` explicitly in `regression_modeling`**: they are never taken over from a
plot.

## New indicators

When `manage_indicator` (directly, or through `insert_tabular_data`) adds an indicator:

- every transformation and standardisation set gets a row for it: the set's own value if its template
  names the indicator, otherwise `linear` and no standardisation;
- the symbol set `targetfeaturesymbols` gets a grey placeholder for it, with the display name as label,
  unit `native` and `"needs_review": true`. A warning lists the indicators that got one.

After the first sync on AI4SH, for example, `ph`, `k+` and `na+` had placeholders:

```json
"ph": { "unit": "native", "alpha": 0.2, "color": "grey", "label": "pH", "needs_review": true }
```

To repeat this by hand, for example after `setup_db` or after inserting indicators outside the processes,
run `sync_indicator_parameter_sets` (user stratum 4 and 5):

**Process file**: `project_neospectra/process/ml_setting/sync_indicator_parameter_sets.json`

```json
{
  "process": [
    {
      "process": "sync_indicator_parameter_sets",
      "parameters": {
        "family_array": ["transformation", "standardisation", "plot_symbol"],
        "rebuild": false
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `family_array` | `transformation`, `standardisation`, `plot_symbol`; empty = all three |
| `rebuild` | Rewrite existing rows from the set templates (`false` = only add missing rows). Frozen sets are never rewritten |

With verbose 0 the sync prints nothing. Check the rows in `ml_setting.indicator_transformation` (e.g. in
DBeaver).

## Create a custom set

`create_parameter_set` (user stratum 4 and 5) copies a set, applies your changes and saves the result as a
`custom` set:

**Process file**: `project_neospectra/process/ml_setting/create_parameter_set.json`

```json
{
  "process": [
    {
      "process": "create_parameter_set",
      "parameters": {
        "family": "transformation",
        "name": "ai4sh_log_carbon",
        "base_set": "standard",
        "settings_fp": "./project_neospectra/settings/transformation_log_carbon.json",
        "abstract": "standard, with log-transformed organic and total carbon"
      }
    }
  ]
}
```

with the changes in `project_neospectra/settings/transformation_log_carbon.json`:

```json
{
  "transformation": {
    "c-org": { "log": true },
    "c-tot": { "log": true }
  }
}
```

| Parameter | Description |
|---|---|
| `family` | One of the families above |
| `name` | Name of the new set. Not `none` or `default`, no `@`, and not the name of a raw or preset set |
| `base_set` | Set to start from: `name`, `name@version`, `default` (= `standard`) or `none` |
| `settings_fp` | JSON file with the changes, in the same shape as the files in `ai4sh/default/`. Empty = an exact copy of `base_set` |
| `abstract` | Short description; empty = "based on &lt;base set&gt;" |

How changes are applied:

- **Transformation and standardisation**: one value per indicator, by indicator name or alias. A
  transformation is `{}` (linear) or exactly one of `{"log": true}`, `{"sqrt": true}`, …; a
  standardisation is `true` or `false`. Unknown indicators or values stop the process with an error.
- **All other families**: your JSON is merged into the base set. Nested objects are merged key by key, so
  `{"randfor": {"hyper_parameters": {"n_estimators": 500}}}` changes only that value and keeps every other
  regressor and `random_state`.

Running `create_parameter_set` again with the same `name` saves **version 2**, and so on. Version 1 stays
as it was. A process that names the set without `@` uses the latest version; the run records which
version that was.

## Frozen sets

When a model is [registered][ml_predict], every parameter set it used becomes **frozen**. The database
then refuses to change, unfreeze or delete the set and its per-indicator rows (only its `abstract` can
still be edited):

```
parameter set transformation/standard@1 is frozen (used by a registered model) - save the change as a new version
```

To change frozen settings, create a new version or a new custom set. New indicators still get rows in a
frozen set, because that does not change anything a model used.

## Next step

[ML modeling][ml_model]: train regressors with explicitly named settings.

[setup_db_ml]: {{ site.baseurl }}/setup_db/ml_setting/
[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
