---
title: "ML Modeling 4: Feature importance"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Feature importance and permutation importance of regression models: what each regressor offers, train/test and k-fold permutation, importance_only for mapping importance without testing the models, PCA components, spectral bands and covariates, and importance next to the model results."
permalink: /machine_learning/ml_model/importance/
author_profile: false
date: 2026-11-06 08:00:00 +0100
last_modified_at: 2026-11-06 12:00:00 +0100
---

Which features does a model rely on? A feature is what the regressor gets after the pre-processing chain: a
band, a PCA component or a Ward cluster, plus the covariate columns. Section 4 of the notebook `ml_model.ipynb`
maps two kinds of importance:

- **feature importance**: the weight the fitted regressor gives each feature;
- **permutation importance**: how much r² drops when the values of one feature are shuffled, i.e. how much the
  predictions depend on it.

The importance depends on the indicator, the regressor and the chain: map it per combination.

## Importance per regressor

| Regressor | Feature importance | Permutation importance |
|---|---|---|
| `ols`, `theil_sen`, `huber` | absolute coefficient | yes |
| `svr` | absolute coefficient with `kernel: linear`; none with `rbf` | yes |
| `dectree`, `randfor` | mean decrease in impurity: how much the splits on a feature reduce the error, summed over the trees | yes |
| `plsr` | squared loadings of each feature summed over the components: how much it builds the components | yes |
| `knn`, `mlp` | none | yes |
| `cubist` | none | none (too slow on many bands) |

Permutation importance works for every regressor in the table except `cubist`, and is repeated 10 times per
feature, scored by r²; the bar is the mean, the line the standard deviation of the repeats.

- **Train/test:** computed on the test samples, with the model fitted on the training samples.
- **K-fold:** computed fold by fold, on the held-out samples of each fold with a model fitted on the other folds,
  and the repeats of all folds pooled. On the training samples it would overrate the features the model overfits.
  If the folds end with different features (a band selection step that selects per fold), it falls back to the
  training samples, with a warning:
  `⚠️  <indicator> <regressor> [kf]: the folds select different features — permutation importance on the training samples`.

The k-fold feature importance is that of the model fitted on all samples.

Numeric covariates are standardised, so their coefficient is per standard deviation, not per unit
([Covariates][ml_model_cov]). With covariates the feature importance has a second panel with the covariate
columns; with one nominal covariate the panel is titled with its name and each class group labelled by its first
class and the number of others (`clay loam +4`). The feature axis reads `Wavelength (nm)` for bands and
`Component` for PCA components.

## Showing and saving

| Parameter | Description |
|---|---|
| `show_feature_importance` / `save_feature_importance` | Feature importance plot |
| `show_permutation_importance` / `save_permutation_importance` | Permutation importance plot; saving also writes the data |
| `n_top_features_in_plot` | Most important features in a saved permutation figure (default 8; 0 = all) |

All four flags default to `false`. Shown plots are composed into grids by the [plot layout][ml_model_layout];
with importance shown, `default` picks the preset `importance`. Saved files, under `project_root_fp`:

- `plot/<name>_tt/importance_<indicator>_<regressor>_tt.png` and `permutation_…` (and `_kf`);
- `covariate_importance/<name>_tt/permutation_<indicator>_<regressor>.json`: the permutation importance of every
  feature, most important first, with its mean, standard deviation and the value of each repeat.

## Importance without the model tests

`importance_only: true` fits the regressors and maps the importance, nothing else: no scatter plots, metrics,
k-fold predictions, saved models or results JSON. It is the quick way to look at importance while exploring.

- **Train/test:** each regressor is fitted on the training samples; the permutation importance is computed on the
  test samples.
- **K-fold:** each regressor is fitted on all samples for the feature importance; the permutation importance is
  computed per held-out fold, as above.
- It needs a show or save flag for feature or permutation importance, otherwise the run stops:
  `ERROR: importance_only needs show/save_feature_importance or show/save_permutation_importance.`
- Per model the run prints `[tt] importance mapped (fitted on <n>, permutation on <n> samples)` instead of the
  metrics, and ends with what was saved, e.g.
  `importance_only: saved feature importance plots and permutation importance plots and data; no models or results`,
  or `importance_only: shown only, nothing saved; no models or results`.

## 4.1.1 PCA components

**Process file**: `project_neospectra/process/ml_model/regression_modeling_importance_only_pca5_tt.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_olif_10nm",
        "dataframe": "neospectra_olif_10nm.snv.d1.mc.pca5",
        "indicator_array": ["c-org", "n-tot"],
        "regressor_array": ["ols", "huber"],
        "indicator_transformation": "none",
        "model_parameters_fp": "./project_neospectra/regressors/regression_model_parameters.json",
        "traintest": true,
        "test_size": 0.3,
        "kfold": false,
        "importance_only": true,
        "show_feature_importance": true,
        "save_feature_importance": true,
        "show_permutation_importance": true,
        "save_permutation_importance": true,
        "n_top_features_in_plot": 0,
        "plot_layout": "./project_neospectra/plot_layout/importance_pca.json",
        "save_plot_layout": true
      }
    }
  ]
}
```

with the layout file `project_neospectra/plot_layout/importance_pca.json`:

```json
{
  "columns": ["importance_tt", "permutation_tt"],
  "wrap": 4,
  "group_by": "indicator",
  "draw": "run",
  "panel_size": [3, 2.6],
  "importance_width": 1,
  "max_permutation_bars": 5
}
```

With five components a feature importance panel needs no extra width (`importance_width: 1`), and
`max_permutation_bars: 5` shows every component (the default is 6, so set it to at least the number of
components). Each indicator gets one row: feature and permutation importance of `ols`, then of `huber`.

![Importance of five PCA components for OLS and Huber]({{ "/assets/media/ml_model/4-1-1_importance_pca5_tt.png" | relative_url }})

The absolute coefficient depends on the scale of the component: the first components have the largest
variance, so a small coefficient can carry much of the prediction. Permutation importance does not depend on the
scale; rank the components by it.

## 4.2 Spectral bands

With 116 bands the feature importance is drawn over the spectrum, and the permutation panel shows only the most
important features (`max_permutation_bars`, default 6). The examples use the data of
[2.2 Regressors][ml_model_regr]: `neospectra_10nm_olif_texture.snv.d1.mc` with the texture covariate in three
groups and the tuned regressor file.

### 4.2.1 PLSR and random forest

**Process file**: `project_neospectra/process/ml_model/regression_modeling_importance_only_randfor+plsr_snv-d1-mc_tt.json`,
with `"indicator_array": ["c-org", "n-tot", "clay"]`, `"regressor_array": ["plsr", "randfor"]`, train/test only,
`"importance_only": true`, both show flags `true`, nothing saved and `"plot_layout": "default"`.

As importance is shown, `default` picks the preset `importance`: one grid per indicator, one row per regressor,
with the scatter column left out (`importance_only` makes no scatter plots). Each row has the feature importance
over the spectrum (2 units wide, the texture classes in a panel of their own) and the 6 most important features by
permutation:

![Band importance for c-org]({{ "/assets/media/ml_model/4-2-1_importance_randfor-plsr_c-org.png" | relative_url }})

![Band importance for n-tot]({{ "/assets/media/ml_model/4-2-1_importance_randfor-plsr_n-tot.png" | relative_url }})

![Band importance for clay]({{ "/assets/media/ml_model/4-2-1_importance_randfor-plsr_clay.png" | relative_url }})

The two regressors measure importance differently, so compare the regions, not the values. Bands that are
important in both, and among the top permutation features, are the robust ones.

### 4.2.2 Four regressors, one indicator, a partial layout

**Process file**: `project_neospectra/process/ml_model/regression_modeling_importance_only_c-org_all-regressors_snv-d1-mc_tt.json`,
with `"indicator_array": ["c-org"]`, `"regressor_array": ["plsr", "svr", "randfor", "mlp"]`, `importance_only`,
shown only (`save_feature_importance` and `save_permutation_importance` `false`), and the layout file
`project_neospectra/plot_layout/importance_8-bars.json`:

```json
{
  "base": "importance",
  "max_permutation_bars": 8
}
```

The file changes one key of the preset `importance`: the permutation panels show the 8 most important features
instead of 6 ([Layout files][ml_model_layout_files]). Cubist is left out: it has neither kind of importance.
MLP has no feature importance, so its row has only the permutation panel.

![Band importance of four regressors for c-org]({{ "/assets/media/ml_model/4-2-2_importance_c-org_all-regressors.png" | relative_url }})

## Importance next to the model results

Without `importance_only`, the models are tested and saved as usual, and the importance can be shown next to the
scatter plot. Cell 4.2.3 does this for c-org and n-tot with random forest and 5-fold cross-validation.

**Process file**: `project_neospectra/process/ml_model/regression_modeling_randfor_importance+scatter_snv-d1-mc_kf5.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_olif_texture",
        "dataframe": "neospectra_10nm_olif_texture.snv.d1.mc",
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
        "model_parameters_fp": "./project_neospectra/regressors/regression_model_parameters_tuned.json",
        "indicator_array": ["c-org", "n-tot"],
        "regressor_array": ["randfor"],
        "traintest": false,
        "kfold": true,
        "folds": 5,
        "show_observed_predicted": true,
        "show_feature_importance": true,
        "show_permutation_importance": true,
        "plot_layout": "./project_neospectra/plot_layout/importance_permutation_scatter_kf.json",
        "save_plot_layout": true
      }
    }
  ]
}
```

with the layout file `project_neospectra/plot_layout/importance_permutation_scatter_kf.json`, which sets the panels
of a row in the order wanted:

```json
{
  "columns": ["importance_kf", "permutation_kf", "scatter_kf"],
  "group_by": "indicator",
  "draw": "run",
  "max_permutation_bars": 6
}
```

A row is 4 units wide (importance 2, permutation 1, scatter 1); `draw: run` draws one grid for both indicators at
the end.

![Random forest with texture: importance, permutation and k-fold scatter]({{ "/assets/media/ml_model/4-2-3_importance-scatter_randfor_texture.png" | relative_url }})

Cell 4.2.4 runs the same without the covariate
(`regression_modeling_randfor_spectra_importance+scatter_snv-d1-mc_kf5.json`, no `covariate_array` and
`class_grouping`):

![Random forest without texture: importance, permutation and k-fold scatter]({{ "/assets/media/ml_model/4-2-4_importance-scatter_randfor_spectra.png" | relative_url }})

Comparing the two shows what the covariate adds on the same samples: its share of the feature importance, whether
a texture class reaches the top permutation features, and the change in the k-fold metrics. A covariate that adds
little importance and no gain in the metrics can be left out.

## Next step

[5 Output and manifest][ml_model_out]: what a run saves.

[ml_model_cov]: {{ site.baseurl }}/machine_learning/ml_model/covariates/
[ml_model_regr]: {{ site.baseurl }}/machine_learning/ml_model/regressors/
[ml_model_layout]: {{ site.baseurl }}/machine_learning/ml_model/plots/#plot-layout
[ml_model_layout_files]: {{ site.baseurl }}/machine_learning/ml_model/plots/#layout-files
[ml_model_out]: {{ site.baseurl }}/machine_learning/ml_model/output/
