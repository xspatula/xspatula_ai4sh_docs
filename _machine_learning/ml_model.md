---
title: "ML Modeling"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Training and evaluating regressors on AI4SoilHealth spectra: regression_modeling parameters, the standard regressor set, sample-id holdout, the preprocessing refitted inside the model, and the saved pipeline with its manifest."
permalink: /machine_learning/ml_model/
author_profile: false
date: 2026-10-26 08:00:00 +0200
last_modified_at: 2026-10-26 08:00:00 +0200
---

Trains and evaluates regressors on the compiled dataset (see [Compile the project dataset][compile]) or on
any dataframe derived from it in ML preprocessing: outlier-cleaned, meancentred, derivative, PCA-reduced,
band-selected, or any other chained variant. Every saved model is a complete pipeline, from unprocessed
spectra to a prediction in the native unit of the indicator, so it can be applied to new spectra (see
[Predict and register][ml_predict]).

The preprocessing steps are the same as for LUCAS; see [LUCAS ML preprocessing][lucas_preprocess].

## Notebook

**Path**: `xspatula_ai4sh/ai4sh/project_neospectra/ml_model.ipynb`

Just one step. Set `scheme_file = '../scheme_ai4sh.json'` in the setup cell (the default), then run the
modeling cell.

## Machine Learning modeling with train-test

**Process file**: `project_neospectra/process/ml_model/regression_modeling.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm",
        "dataframe": "previous",
        "indicator_array": ["c-org", "n-tot"],
        "regressor_array": ["ols", "huber"],
        "model_parameters_fp": "./project_neospectra/regressors/regression_model_parameters.json",
        "indicator_transformation": "none",
        "indicator_standardisation": "none",
        "traintest": true,
        "test_size": 0.3,
        "kfold": false
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `dataframe` | The dataframe to model: `"raw"` for the unprocessed selection, an explicit chained name, or `"previous"` |
| `indicator_array` | The target indicators; one model per indicator and regressor |
| `regressor_array` | The regressors to fit per indicator, see the table below |
| `model_parameters_fp` | Regressor hyper-parameters: `default` (the `standard` set), `none` (scikit-learn defaults), a set name or a JSON file. See [ML settings][ml_settings] |
| `indicator_transformation` | Transformation of the indicator before fitting: `none`, `default`/`standard`, a set name or a JSON file. Default `none` |
| `indicator_standardisation` | Z-score standardisation of the indicator: same forms. Default `none` |
| `refit_preprocessing` | Refit the preprocessing chain of the dataframe inside the model on the training samples (default `true`, see below) |
| `traintest` | Split into train and test samples and evaluate on the test samples |
| `test_size` | Fraction of samples held out for testing (0.3 = 30%) |
| `kfold` | Evaluate with k-fold cross-validation (`folds` folds, default 10) |
| `model_fp` | Folder for the saved models, relative to the project root. Empty = `<project_root_fp>/models` |

**Set `indicator_transformation` and `indicator_standardisation` here, deliberately.** They are not taken
over from the transformation you last looked at in a plot. The process file in the repository leaves them
out, which gives `none`; the example above writes them out.

The example uses the project's own regressor file. Since it holds the same values as the `standard` set,
`"model_parameters_fp": "default"` gives the same models, and records a database set instead of a file in
the provenance.

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
| `cubist` | Cubist | `random_state: 42` |

`random_state: 42` makes the stochastic regressors give the same model every run. To change
hyper-parameters, create a custom regressor set ([ML settings][ml_settings]) rather than editing a file.

## Train and test samples

The test samples are chosen from the `sample_id`, not from the row order: the `round(test_size × n)`
samples with the lowest hash of their `sample_id` are the test samples. The same sample is therefore always
a test sample for the same `test_size`, whatever order or subset the dataframe has.

Band selection (permutation and univariate selection in ML preprocessing) uses the same rule. Give the
selection and the modeling the **same `test_size`**, otherwise the model's test samples include samples
the band selection has already seen, and the test metrics are too optimistic. The model prints a note when
the two differ.

## The preprocessing is refitted inside the model

With `refit_preprocessing: true` (the default), the model does not train on the preprocessed dataframe
directly. It starts from the unprocessed source spectra (`source_parquet` in the dataframe's companion
JSON) and replays its `preprocessing_chain` as steps of a scikit-learn pipeline, fitted on the training
samples only. Mean centring, scaling, the MSC reference spectrum, PCA, Ward clusters, variance threshold
and univariate band selection therefore never see the test samples, and their fitted values are saved
with the model.

Two steps are not refitted, and the model run says so in its notes:

- **Outlier detection** (`_ol`): indicator outliers were set to missing using all samples.
- **Permutation selection**: the selected bands are taken as fixed. They are clean only if the selection
  used the same `test_size` (see above).

On `neospectra_10nm_ol_mc` (outlier detection, then meancentring) the run starts with:

```
    Previous: spectra_scaling — neospectra_10nm_ol_mc.parquet (2026-10-26T21:33:17)
    ⚠️  01_ol: indicator outliers were set to missing using all samples (not refitted)
```

The indicator transformation and standardisation are part of the pipeline too, so predictions are returned
in the native unit of the indicator.

## Output

Written under `project_root_fp`, named after the input dataframe:

- **`<name>_regression.json`**: per indicator and regressor the test metrics `r2`, `rmse`, `mae`, `mape`,
  `medae`, `bias`, `rpiq`, `ccc`, with `n_train`/`n_test`, the model file and its sha256. `input_parquet`
  is the dataframe you gave, `source_parquet` the unprocessed spectra the pipeline starts from.
- **`models/<name>_tt/`** (or `_kf/`), per indicator and regressor:
  - `<name>_<indicator>_<regressor>_tt.joblib`: the fitted pipeline (preprocessing, regressor, indicator
    transformation);
  - `<name>_<indicator>_<regressor>_tt.manifest.json`: everything needed to trust and reuse the model: the
    sha256 of the `.joblib`, indicator and unit, regressor and hyper-parameters, the input bands and signal
    type, the preprocessing chain, the training sample ids, the metrics, and the provenance (process
    parameters, every settings set or file used with its checksum, the software versions and the git
    commit).
- **`covariate_importance/<name>_tt/`**: feature and band importance of the fitted models.
- **`plot/<name>_tt/`**: the observed-predicted and importance plots, if saved.

The run on `neospectra_10nm_ol_mc` (136 samples with both indicators, 116 bands 1350–2500 nm) printed:

```
    ── c-org ──
                  ols/tr              huber/tr
    r2            -5.5885             0.4232
    rmse          3.9150              1.1584
    rpiq          0.5677              1.9187

    ── n-tot ──
                  ols/tr              huber/tr
    r2            -0.2044             0.5973
    rmse          0.1398              0.0808
    rpiq          1.4486              2.5052
```

and saved, among others,
`models/neospectra_10nm_ol_mc_tt/neospectra_10nm_ol_mc_c-org_huber_tt.joblib` with its manifest. An
excerpt of the manifest of the `c-org` OLS model:

```json
{
  "manifest_version": 1,
  "model_file": "neospectra_10nm_ol_mc_c-org_ols_tt.joblib",
  "model_sha256": "6249e5da…8a024f",
  "indicator": "c-org",
  "indicator_unit": "weight percent",
  "regressor": "ols",
  "hyper_parameters": { "fit_intercept": true },
  "mode": "traintest",
  "input": { "columns": ["wl_nm_1350", "…", "wl_nm_2500"], "signal_type": "refl", "n_bands": 116 },
  "pipeline": {
    "steps": ["bands", "02_mc"],
    "indicator": { "transform": "linear", "standardise": false },
    "holdout": { "test_size": 0.3, "key": "sample_id" },
    "notes": ["01_ol: indicator outliers were set to missing using all samples (not refitted)"]
  },
  "data": {
    "input_parquet": "neospectra_10nm_ol_mc.parquet",
    "source_parquet": "neospectra_10nm.parquet",
    "n_train": 94,
    "train_sample_ids": [2, 4, 9, "…"]
  }
}
```

**Reading the numbers.** OLS on 116 bands with 94 training samples has almost as many unknowns as samples
and overfits: a negative `r2` means it predicts the test samples worse than their mean. The robust Huber
regressor does better (`rpiq` 1.9 for `c-org`, 2.5 for `n-tot`), but Huber also warned that it did not
converge in 100 iterations. Reduce the bands (PCA, band selection) or use regularised regressors before
reading anything into these metrics.

**Models saved before 2026-10-23 have no manifest** and are refused by `predict_independent` and
`register_model` (`no manifest for … rerun regression_modeling`). Rerun `regression_modeling`.
{: .notice--warning}

## Next step

[Predict and register][ml_predict]: apply a saved model to new spectra, and register the final model in
the database.

[compile]: {{ site.baseurl }}/machine_learning/parquet_files/#compile-the-project-dataset
[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
[lucas_preprocess]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/ml_preprocess/
