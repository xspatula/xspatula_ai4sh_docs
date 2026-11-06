---
title: "ML Modeling 2.2: Regressors"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "The regressors of regression_modeling and their hyper-parameters: the standard set, PLSR components and their cap, and PLSR, SVR, Cubist, random forest and MLP on the full pre-processed spectra with a tuned parameter file."
permalink: /machine_learning/ml_model/regressors/
author_profile: false
date: 2026-11-06 08:00:00 +0100
last_modified_at: 2026-11-06 12:00:00 +0100
---

Section 2.2 of the notebook `ml_model.ipynb` runs the more advanced regressors on the full pre-processed spectra. This
page lists every regressor, its `standard` hyper-parameters and what to consider when you tune them.

## Available regressors

Hyper-parameters of the `standard` regressor set (`model_parameters_fp: "default"`):

| Key | Regressor | Hyper-parameters |
|---|---|---|
| `ols` | Ordinary least squares | `fit_intercept: true` |
| `theil_sen` | Theil-Sen estimator | `random_state: 42` |
| `huber` | Huber regressor | library defaults |
| `knn` | K-nearest neighbours | library defaults |
| `dectree` | Decision tree | `random_state: 42` |
| `svr` | Support vector regression | `kernel: linear`, `C: 1.5`, `epsilon: 0.05` |
| `randfor` | Random forest | `n_estimators: 30`, `random_state: 42` |
| `mlp` | Multi-layer perceptron | `hidden_layer_sizes: [100, 100]`, `max_iter: 200`, `tol: 0.001`, `epsilon: 1e-8`, `random_state: 42` |
| `plsr` | Partial least squares regression | `n_components: 10` (capped to the data, see below), `scale: false` |
| `cubist` | Cubist | `random_state: 42` |

`random_state: 42` makes the stochastic regressors give the same model every run. To change hyper-parameters,
give a JSON file (as in the examples below) or create a custom regressor set ([ML settings][ml_settings]); a
custom set is recorded in the database and frozen when a model using it is registered.

`plsr` was added in version 2 of the `standard` set. Version 1 (`standard@1`) is the set without it. In an
existing database, version 2 is added by rerunning the main cell of `setup/setup_db.ipynb` after updating
xspatula_ai4sh; existing records are skipped. Until then `"default"` gives version 1, and `plsr` runs with the
scikit-learn defaults (2 components, `scale: true`).
{: .notice--info}

### Notes on PLSR components

PLSR needs a note of its own: its main hyper-parameter is capped by the data. The tuning of `svr`, `cubist`,
`randfor` and `mlp` is under each regressor in [The examples](#the-examples).

PLSR is the classic regressor for NIR spectra. It compresses the inputs into a few latent components that
explain both the spectra and the indicator. Too few components miss information; too many fit the noise of the
training samples. The `standard` set keeps 10 as a middle value; the best number depends on the samples and the
indicator, so choose it per project.

`scale: false` makes PLSR use the inputs as the pre-processing chain leaves them. The scaling is chosen in the
chain, in one place that the manifest records: meancentring gives classic centred PLS, and autoscaling gives
unit-variance PLS. scikit-learn's default, `scale: true`, would autoscale every input inside the regressor and
silently override the scaling step of the chain. Numeric covariates are standardised by the model itself, class
columns stay 0/1 ([Covariates][ml_model_cov]). A custom regressor set can still set `"scale": true`.

The number of components can not exceed what the training data hold: the rank of the centred training data, at
most the number of training samples minus one. The rank is at most the number of features left after the
pre-processing chain (bands left after band selection, components after PCA, Ward clusters, plus covariate
columns), and can be lower: the one-hot columns of a nominal covariate always sum to 1, so after centring they
count one dimension less. A component beyond the rank is numerical noise. A larger `n_components` is therefore
capped to the rank when the model is fitted, with a note under the `Model: plsr` line, at most one per indicator
and mode:

```
        ⚠️  c-org plsr [tt]: n_components 10 → 5 (<n> samples, <p> features) — fewer components than requested
```

When the rank is below the number of features, the note adds it, e.g. a `pca5` chain with a covariate of three
class groups (5 + 3 features, rank 7):

```
        ⚠️  c-org plsr [kf]: n_components 10 → 7 (<n> samples, 8 features, rank 7) — fewer components than requested
```

The `kf` note refers to the final model, fitted on all samples; the k-fold folds are capped the same way but not
reported. Band selection with `plsr` (permutation, RFE) caps without a note. The manifest records the requested
and the used value ([Output and manifest][ml_model_out]).

## The examples

All cells of section 2.2 use the same data: the indicators c-org, n-tot and clay with outliers removed, the soil
texture class in three groups as covariate (as in [2.1 Covariates][ml_model_cov]), and the spectra after SNV,
first derivative and meancentring without PCA: `neospectra_10nm_olif_texture.snv.d1.mc`, 116 bands (see
[Pre-process the compiled dataset][compile_chain]). Both train/test and 5-fold cross-validation run. One process
file per regressor, e.g.

**Process file**: `project_neospectra/process/ml_model/regression_modeling_texture_plsr_snv-d1-mc_tt+kf5.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_olif_texture",
        "dataframe": "neospectra_10nm_olif_texture.snv.d1.mc",
        "indicator_array": ["c-org", "n-tot", "clay"],
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
        "traintest": true,
        "test_size": 0.3,
        "kfold": true,
        "folds": 5
      }
    }
  ]
}
```

The other files differ only in `regressor_array`: `…_svr_…`, `…_cubist_…`, `…_randfor_…`, `…_mlp_…`. All use
`project_neospectra/regressors/regression_model_parameters_tuned.json`:

```json
{
  "plsr":    {"hyper_parameters": {"n_components": 5, "scale": false}},
  "svr":     {"hyper_parameters": {"kernel": "linear", "C": 10, "epsilon": 0.01}},
  "cubist":  {"hyper_parameters": {"n_committees": 10, "random_state": 42}},
  "randfor": {"hyper_parameters": {"n_estimators": 300, "max_features": 0.33, "random_state": 42}},
  "mlp":     {"hyper_parameters": {"hidden_layer_sizes": [20], "alpha": 10.0, "max_iter": 2000, "random_state": 42}}
}
```

These values were chosen from 5-fold tests on this dataset of about 150 samples. They are a starting point, not
general values: test again when the samples, the indicator or the chain change. Choose from the k-fold
metrics, not from a single train/test split, and treat small differences between settings as noise.

Every figure has one row per indicator, train/test on the left and k-fold on the right.

### 2.2.1 PLSR

![PLSR on the full pre-processed spectra]({{ "/assets/media/ml_model/2-2-1_plsr.png" | relative_url }})

- Do not end the chain with PCA. PLSR builds its own components, chosen for the indicator; after `pca5` it is a
  regression on 5 PCA scores (plus the covariate columns).
- `n_components` is the only important hyper-parameter. Too few underfit, too many fit noise. Here 3–5
  components were best, 10 was clearly worse and 20 overfitted. Test e.g. 3, 5, 8, 12.
- The full pre-processed spectra were as good as, or better than, `pca5`.

### 2.2.2 SVR

![SVR on the full pre-processed spectra]({{ "/assets/media/ml_model/2-2-2_svr.png" | relative_url }})

- `C` sets how hard errors are penalised; higher values fit the training samples closer. Test 1, 10, 100.
- `epsilon` is in the unit of the indicator: errors smaller than epsilon are ignored. Keep it well below the
  RMSE of the indicator; a value that suits c-org (%) can be far too wide for n-tot.
- Try both `kernel: linear` and `rbf`; here linear was better for c-org and n-tot, rbf for clay.
- Inputs are not scaled inside the model: the chain and the standardised covariates set the scale.

### 2.2.3 Cubist

![Cubist on the full pre-processed spectra]({{ "/assets/media/ml_model/2-2-3_cubist.png" | relative_url }})

- `n_committees` (an ensemble of rule sets) of 10 was clearly better than the default 1.
- `neighbors` (1–9) corrects predictions with the nearest training samples; it did not help here.
- Cubist fits the training samples closely: compare its train/test and k-fold results.

### 2.2.4 Random forest

![Random forest on the full pre-processed spectra]({{ "/assets/media/ml_model/2-2-4_randfor.png" | relative_url }})

- The most stable model across the indicators here.
- More trees (`n_estimators: 300` instead of 30) give more stable predictions, at the cost of time.
- `max_features: 0.33` (a third of the features tried at each split) is the usual choice for regression and was
  better than all features.
- Trees do not extrapolate: predictions stay within the range of the training samples.

### 2.2.5 MLP

![MLP on the full pre-processed spectra]({{ "/assets/media/ml_model/2-2-5_mlp.png" | relative_url }})

- Not recommended for a dataset of this size; it was the weakest and least stable model, and its k-fold results
  fall far below its train/test results.
- If used, keep the network small (`hidden_layer_sizes: [20]`) and the regularisation strong (`alpha: 10`). The
  default `[100, 100]` has many times more weights than there are samples.
- The inputs are standardised inside the model.
- `ConvergenceWarning: Stochastic Optimizer: Maximum iterations (2000) reached` is printed for many folds: with
  strong regularisation the optimiser stops before it settles. The model is still saved.

The `standard` set keeps its middle values (e.g. `n_components: 10`); a project file or a custom set overrides
them. To compare the regressors in one figure, run them in one process ([3 Arrays and plot layouts][ml_model_arrays]).

## Next step

[3 Arrays and plot layouts][ml_model_plots]: many indicators or regressors in one run, and grid figures.

[compile_chain]: {{ site.baseurl }}/machine_learning/compile/#pre-process-the-compiled-dataset
[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[ml_model_cov]: {{ site.baseurl }}/machine_learning/ml_model/covariates/
[ml_model_out]: {{ site.baseurl }}/machine_learning/ml_model/output/
[ml_model_plots]: {{ site.baseurl }}/machine_learning/ml_model/plots/
[ml_model_arrays]: {{ site.baseurl }}/machine_learning/ml_model/plots/#321-many-regressors-one-indicator
