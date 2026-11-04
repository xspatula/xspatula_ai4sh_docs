---
title: "ML Modeling"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Training and evaluating regressors on AI4SoilHealth spectra: regression_modeling parameters, the standard regressor set, covariates (numeric and classes such as soil texture) next to the spectra, sample-id holdout, the preprocessing refitted inside the model, and the saved pipeline with its manifest."
permalink: /machine_learning/ml_model/
author_profile: false
date: 2026-10-26 08:00:00 +0200
last_modified_at: 2026-11-04 08:00:00 +0100
---

Trains and evaluates regressors on the compiled dataset (see [Compile dataframe][compile]) or on
any dataframe derived from it in ML preprocessing: outlier-cleaned, meancentred, derivative, PCA-reduced,
band-selected, or any other chained variant. Every saved model is a complete pipeline, from unprocessed
spectra to a prediction in the native unit of the indicator, so it can be applied to new spectra (see
[Predict and register][ml_predict]).

The preprocessing steps are described in [ML pre-processing 1][ml_preprocess].

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
| `covariate_array` | Covariates used next to the spectra: empty = spectra only (default), `all` = every covariate of the compiled dataset, or the covariate names. See [Covariates](#covariates) |
| `class_grouping` | Pool the classes of a nominal covariate into groups before encoding. See [Covariates](#covariates) |
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
over from the transformation you last looked at in a plot. The process file in the repository sets
`indicator_transformation` to `none` and leaves out `indicator_standardisation`, which also gives `none`;
the example above writes both out.

The example uses the project's own regressor file. Since it holds the same values as the `standard` set,
`"model_parameters_fp": "default"` gives the same models, and records a database set instead of a file in
the provenance.

## Available regressors

Hyper-parameters of the `standard` regressor set (`model_parameters_fp: "default"`, the latest version of
`standard`, version 2):

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

PLSR is the classic regressor for NIR spectra. It compresses the inputs into a few latent components that
explain both the spectra and the indicator. Too few components miss information; too many fit the noise of
the training samples. Ten is a middle value for 10 nm spectra and a training set of about a hundred samples
or more. It is still worth comparing a few values (e.g. 5, 10, 15, 20) in a custom regressor set, using the
test metrics.

`scale: false` makes PLSR use the inputs as the preprocessing chain leaves them. The scaling is chosen in
the chain, in one place that the manifest records: meancentring gives classic centred PLS, and autoscaling
gives unit-variance PLS. scikit-learn's default, `scale: true`, would autoscale every input inside the
regressor and silently override the scaling step of the chain. Covariates are not part of the chain: numeric
covariates are standardised by the model itself (see [Covariates](#covariates)), class columns stay 0/1.
A custom regressor set can still set `"scale": true`.

The number of components can not exceed what the training data hold: the number of training samples minus
one, and the number of features left after the preprocessing chain (bands left after band selection,
components after PCA, Ward clusters, plus covariate columns). A larger `n_components` is capped when the
model is fitted. The run prints a note under the `Model: plsr` line of each saved model that was capped, at
most one per indicator and mode (`tt` for train/test, `kf` for k-fold):

```
        ⚠️  c-org plsr [tt]: n_components 10 → 5 (<n> samples, <p> features) — fewer components than requested
```

The `kf` note refers to the final model, fitted on all samples; the k-fold folds are capped the same way but
not reported. Band selection with `plsr` (permutation, RFE) caps without a note.

The common case is a chain that ends in `pca5`: PLSR then uses 5 components, not 10. The manifest records
both values (see [Output](#output)).

`plsr` was added in version 2 of the `standard` set. Version 1 (`standard@1`) is the set without it. In an
existing database, version 2 is added by rerunning the main cell of `setup/setup_db.ipynb` after updating
xspatula_ai4sh; existing records are skipped. Until then `"default"` gives version 1, and `plsr` runs with
the scikit-learn defaults (2 components, `scale: true`).
{: .notice--info}

`random_state: 42` makes the stochastic regressors give the same model every run. To change
hyper-parameters, create a custom regressor set ([ML settings][ml_settings]) rather than editing a file.

## Covariates

Besides the spectra, a model can use **covariates**: other observations of the same samples, selected with
`select_covariate_only` and compiled with the indicators and spectra (see
[Compile dataframe][compile]). A covariate is either numeric (e.g. a field pH) or nominal, a class
with unit `class` (e.g. soil texture).

**Process file**: `project_neospectra/process/ml_model/regression_modeling_texture.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_texture",
        "dataframe": "raw",
        "indicator_array": ["c-org", "n-tot"],
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
        "regressor_array": ["ols", "randfor"],
        "model_parameters_fp": "./project_neospectra/regressors/regression_model_parameters.json",
        "traintest": true,
        "test_size": 0.3,
        "kfold": true,
        "folds": 5
      }
    }
  ]
}
```

- **`covariate_array`**: empty means spectra only, `all` means every covariate of the compiled dataset (the
  covariates listed in its companion JSON), or list the covariate names. Nominal covariates become one 0/1
  column per class (one-hot encoding).
- **Numeric covariates are standardised** with the mean and standard deviation of the training samples:
  (value − mean) / std. The mean and std are fitted with the model, saved in it and applied unchanged when
  predicting. Why: without it, a
  covariate's weight in `plsr`, `knn` and `svr` would depend on its unit (clay in % or in g/kg) and its
  variance. Consequences:
  - the unit of a numeric covariate does not change the predictions;
  - for linear regressors a numeric covariate's coefficient, and its feature importance (|coefficient|), is
    per standard deviation, not per unit;
  - tree-based regressors (`dectree`, `randfor`) are unaffected.

  A standardised covariate has variance 1. A band of meancentred reflectance spectra has a far smaller
  variance, so in `plsr`, `knn` and `svr` one numeric covariate can weigh as much as many bands. Choose
  autoscaling in the chain to put the bands on the same footing.
- **A covariate that does not vary in the training samples has no effect**: a numeric covariate with one
  value (it is only centred, an all-zero column) or a class covariate with one class. The run prints a note
  once per indicator and mode:
  `⚠️  <indicator> [tt|kf]: covariate(s) <labels> do not vary in the training samples — no effect`.
- **`class_grouping`** pools the classes of a covariate before encoding: an array of objects
  `{"covariate": <name>, "groups": [[class, class, …], …]}`. Every class in the data must be placed in a group;
  otherwise the run stops and lists the classes it could not place
  (`class_grouping for "soil texture usda" does not place the class(es): …`). A group is named by its classes
  joined with `|`, e.g. `sand|loamy sand|sandy loam`.
- **Predefined groupings** need no `class_grouping`: they are separate covariates. When observations are
  imported, texture is derived into four nominal classifications: `soil texture usda` (12 classes),
  `soil texture usda major group` (4 groups), `soil texture iuss` and `soil texture iuss major group`
  (see [Derived soil texture classes][obs_texture]). LUCAS has the same four. Select the one you want as
  covariate.
- **Samples without a value for each covariate of the model are skipped**, both when modelling and when
  predicting (they get no prediction, and the number is printed). A sample missing one of two covariates is
  skipped too.
- **A class the model has not seen cannot be predicted.** The model learns one column per class from its
  training samples; a class without training samples has no column, and the model has nothing to base a
  prediction on. In train/test, test samples of such a class are skipped, and the run names the class:
  `<indicator>: <n> test sample(s) with a <covariate> class not in the training samples (<classes>), skipped`.
  With k-fold, a class with fewer samples than folds is missing from some training folds; its samples in
  those folds are predicted without a class column, which biases the k-fold metrics. The run warns
  (`… class(es) with fewer samples than folds (<folds>): <class> (<n>) … — consider class_grouping`). The cure
  in both cases is the same: pool the rare classes with a `class_grouping`, or use a major-group covariate.
- **The same train-only fitting as the spectra:** the spectra's preprocessing chain is still refitted on the
  training samples only, now together with the covariates, in one saved model. That model needs only the
  unprocessed spectra plus the covariates to predict.
- A covariate of the model is never an indicator. Other covariates of the dataset are only modelled as
  indicators when named in `indicator_array`.

`covariate_array` and `class_grouping` are registered by `setup_processes.ipynb`; rerun it after updating
xspatula_ai4sh.
{: .notice--info}

**Which classification?** With few samples (e.g. a few hundred samples over the 12 USDA texture classes), use a
coarse grouping or a major-group covariate, so that every class has enough samples. For linear regressors a
class shifts the prediction by a fixed amount; tree-based regressors can use the class columns in their splits.

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

Three steps are not refitted:

- **Outlier detection** (`.olif`, `.olee`, `.ollof`, `.ol1csvm`): indicator outliers were set to missing using
  all samples.
- **[Indicator range][pp1_range]** (`.ir`): the limits are fixed, nothing is learned from the samples, so
  there is nothing to refit.
- **Permutation selection**: the selected bands are taken as fixed. They are clean only if the selection
  used the same `test_size` (see above).

When outlier detection or permutation selection is in the chain of the dataframe you model, the run prints a
note at the start, e.g. for a chain that starts with isolation forest outlier detection:

```
    ⚠️  01_olif: indicator outliers were set to missing using all samples (not refitted)
```

For a compiled dataset the outlier step belongs to an input: it is recorded under `inputs` in the compiled
companion JSON (see [Compile dataframe][compile]), and the model prints no note about it.

The indicator transformation and standardisation are part of the pipeline too, so predictions are returned
in the native unit of the indicator.

## Output

Written under `project_root_fp`, named after the input dataframe. With covariates, the model, plot and results
names get the suffix `_cv-<covariates>` (the names in lower case without spaces or punctuation, e.g.
`_cv-soiltextureusdamajorgroup`; replaced by an 8-character hash when longer than 40 characters), so they do
not overwrite the spectra-only models. The tag names the covariates, not the `class_grouping`: a rerun with
the same covariates but another grouping overwrites the earlier models, plots and results. Move them first if
you want to compare groupings. For the texture example above the names are e.g.
`models/neospectra_10nm_texture_cv-soiltextureusda_tt/neospectra_10nm_texture_cv-soiltextureusda_c-org_randfor_tt.joblib`
and `neospectra_10nm_texture_cv-soiltextureusda_regression.json`.


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
- **`plot/<name>_tt/`**: the observed-predicted and importance plots, if saved. With covariates the feature
  importance plot gets a second panel with the covariate columns (e.g.
  `soil texture usda=sand|loamy sand|sandy loam`), and the
  permutation importance lists them next to the wavelengths.

For `plsr` the manifest keeps the requested value in `hyper_parameters` and the value used in
`fitted_hyper_parameters`, e.g. `"hyper_parameters": {"n_components": 10}` and
`"fitted_hyper_parameters": {"n_components": 5}` after a `pca5` chain (`{}` for the other regressors). In
`<name>_regression.json` the `plsr` metrics also have `fitted_hyper_parameters`.

With covariates the manifest's `input` also has `covariates`: `numeric`, `nominal`, `class_grouping`,
`levels` (the classes, after grouping, seen in training), `numeric_standardisation`, the training mean and
standard deviation of each numeric covariate (6 significant digits; `std: 0` for a covariate that did not
vary), e.g. `{"clay": {"mean": <mean>, "std": <std>}}`, and `numeric_units`, the unit of each numeric
covariate, e.g. `{"clay": "weight percent"}`. Prediction checks the unit (see
[Predict and register][ml_predict]).

The run prints the main test metrics per indicator, one column per regressor:

```
    ── c-org ──
                  ols/tr              huber/tr
    r2            <value>             <value>
    rmse          <value>             <value>
    rpiq          <value>             <value>
```

For the meancentred dataframe `neospectra_10nm.mc` the models are saved as, e.g.,
`models/neospectra_10nm.mc_tt/neospectra_10nm.mc_c-org_huber_tt.joblib`, each with its manifest. An
excerpt of a manifest (`c-org`, OLS):

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

**Reading the numbers.** OLS on all bands has nearly as many unknowns as training samples and overfits;
a negative `r2` means it predicts the test samples worse than their mean. Robust and regularised
regressors, or fewer inputs (PCA, band reduction, band selection), usually do better. `rpiq` (the
interquartile range of the observed values divided by the RMSE) is the common yardstick for soil
spectroscopy: the higher, the better.

A model without a manifest is refused by `predict_independent` and `register_model`
(`no manifest for … rerun regression_modeling`). Rerun `regression_modeling`.
{: .notice--warning}

## Next step

[Predict and register][ml_predict]: apply a saved model to new spectra, and register the final model in
the database.

[compile]: {{ site.baseurl }}/machine_learning/compile/
[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
[ml_preprocess]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[obs_texture]: {{ site.baseurl }}/observation/#derived-soil-texture-classes
[pp1_range]: {{ site.baseurl }}/machine_learning/ml_preprocess1/range/
