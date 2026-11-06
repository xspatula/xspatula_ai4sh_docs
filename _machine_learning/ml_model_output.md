---
title: "ML Modeling 5: Output and manifest"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "What regression_modeling saves: the fitted pipelines and their manifests, the results JSON, plots and permutation data, the names with the covariate tag, and the fitted hyper-parameters."
permalink: /machine_learning/ml_model/output/
author_profile: false
date: 2026-11-06 08:00:00 +0100
last_modified_at: 2026-11-06 12:00:00 +0100
---

`regression_modeling` writes its output under `project_root_fp`, named after the input dataframe. With
`importance_only` only the importance plots and data are saved ([Feature importance][ml_model_imp_only]).

## Names

`<name>` is the dataframe name. With covariates it gets the suffix `_cv-<covariates>` (the names in lower case
without spaces or punctuation, e.g. `_cv-soiltextureusdamajorgroup`; replaced by an 8-character hash when longer
than 40 characters), so the models do not overwrite the spectra-only models of the same dataframe.

The tag names the covariates, not the `class_grouping`: a rerun with the same covariates but another grouping
overwrites the earlier models, plots and results. Move them first if you want to compare groupings.

## Files

| Path | Content |
|---|---|
| `<name>_regression.json` | Per indicator, mode and regressor the metrics `r2`, `rmse`, `mae`, `mape`, `medae`, `bias`, `rpiq`, `ccc`, with `n_train`/`n_test` (k-fold: `n` and the per-fold mean and standard deviation), the model file and its sha256. `input_parquet` is the dataframe you gave, `source_parquet` the unprocessed spectra the pipeline starts from |
| `models/<name>_tt/` (and `_kf/`) | Per indicator and regressor `<name>_<indicator>_<regressor>_tt.joblib`, the fitted pipeline (pre-processing, regressor, indicator transformation), and `<name>_<indicator>_<regressor>_tt.manifest.json` |
| `plot/<name>_tt/` (and `_kf/`) | The saved plots, one figure each: `scatter_…`, `importance_…`, `permutation_…` ([Feature importance][ml_model_imp]) |
| `plot/<name>_grid/` | The grid figures, with `save_plot_layout: true` ([Plot layout][ml_model_layout]) |
| `covariate_importance/<name>_tt/` (and `_kf/`) | `permutation_<indicator>_<regressor>.json`, with `save_permutation_importance: true` |

For the texture example of [2.1 Covariates][ml_model_cov] the names are e.g.
`models/neospectra_10nm_texture_cv-soiltextureusda_tt/neospectra_10nm_texture_cv-soiltextureusda_c-org_randfor_tt.joblib`
and `neospectra_10nm_texture_cv-soiltextureusda_regression.json`.

The train/test model is fitted on the training samples; the k-fold model on all samples, after the folds have
been scored.

## The manifest

The manifest holds everything needed to trust and reuse the model: the sha256 of the `.joblib`, indicator and
unit, regressor and hyper-parameters, the input bands and signal type, the pre-processing chain, the training
sample ids, the metrics, and the provenance (process parameters, every settings set or file used with its
checksum, the software versions and the git commit). An excerpt (`neospectra_10nm.mc`, c-org, OLS):

```json
{
  "manifest_version": 1,
  "model_file": "neospectra_10nm.mc_c-org_ols_tt.joblib",
  "model_sha256": "…",
  "indicator": "c-org",
  "indicator_unit": "weight percent",
  "regressor": "ols",
  "hyper_parameters": { "fit_intercept": true },
  "fitted_hyper_parameters": {},
  "mode": "traintest",
  "input": { "columns": ["wl_nm_1350", "…", "wl_nm_2500"], "signal_type": "refl", "n_bands": 116 },
  "pipeline": {
    "steps": ["bands", "01_mc"],
    "indicator": { "transform": "linear", "standardise": false },
    "holdout": { "test_size": 0.3, "key": "sample_id" },
    "notes": []
  },
  "data": {
    "input_parquet": "neospectra_10nm.mc.parquet",
    "source_parquet": "neospectra_10nm.parquet",
    "n_train": "…",
    "train_sample_ids": ["…"]
  }
}
```

- **`fitted_hyper_parameters`**: for `plsr` the value used when it differs from the requested one, e.g.
  `"hyper_parameters": {"n_components": 10}` and `"fitted_hyper_parameters": {"n_components": 5}` after a `pca5`
  chain ([PLSR components][ml_model_plsr]); `{}` for the other regressors. In `<name>_regression.json` the `plsr`
  metrics also have `fitted_hyper_parameters`.
- **Covariates**: the manifest's `input` also has `covariates`: `numeric`, `nominal`, `class_grouping`, `levels`
  (the classes, after grouping, seen in training), `numeric_standardisation`, the training mean and standard
  deviation of each numeric covariate (6 significant digits; `std: 0` for a covariate that did not vary), e.g.
  `{"clay": {"mean": <mean>, "std": <std>}}`, and `numeric_units`, the unit of each numeric covariate, e.g.
  `{"clay": "weight percent"}`. Prediction checks the unit (see [Predict and register][ml_predict]).

A model without a manifest is refused by `predict_independent` and `register_model`
(`no manifest for … rerun regression_modeling`). Rerun `regression_modeling`.
{: .notice--warning}

**Keep the models you want to register.** A rerun with `"overwrite": true` into the same folder replaces the
models; a registered model whose file changed no longer matches its sha256. Set `model_fp` to a folder you do not
clean up for models you intend to register.

## Next step

[Predict and register][ml_predict]: apply a saved model to new spectra, and register the final model in the
database.

[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
[ml_model_cov]: {{ site.baseurl }}/machine_learning/ml_model/covariates/
[ml_model_plsr]: {{ site.baseurl }}/machine_learning/ml_model/regressors/#notes-on-plsr-components
[ml_model_imp]: {{ site.baseurl }}/machine_learning/ml_model/importance/
[ml_model_imp_only]: {{ site.baseurl }}/machine_learning/ml_model/importance/#importance-without-the-model-tests
[ml_model_layout]: {{ site.baseurl }}/machine_learning/ml_model/plots/#plot-layout
