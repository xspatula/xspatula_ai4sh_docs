---
title: "ML Modeling"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Training and evaluating regressors on AI4SoilHealth spectra and covariates with regression_modeling: how a run works, the notebook ml_model.ipynb, and every parameter with a link to the page that explains it."
permalink: /machine_learning/ml_model/
author_profile: false
date: 2026-10-26 08:00:00 +0200
last_modified_at: 2026-11-06 12:00:00 +0100
---

The process `regression_modeling` trains and evaluates regressors on compiled datasets that must include both indicators and input features (spectra and/or other covariates) (see
[Compile dataframe][compile]). Every saved
model also saves a complete pipeline of the processes leading up to the model, including any pre-processing steps. A saved model can then be applied to independent feature data for predicting indicators from e.g. spectra and/or other covariates (see [Predict and register][ml_predict]).

## Notebook

**Path**: `xspatula_ai4sh/ai4sh/project_neospectra/ml_model.ipynb`

Set `scheme_file = '../scheme_ai4sh.json'` in the setup cell (the default), then run the cell you need. Each
cell runs one process file from `project_neospectra/process/ml_model/`. The pages of ML modeling manual collection follow the
blocks of the notebook:

| Page | Cells | Content |
|---|---|---|
| [1 Pre-processing effects][ml_model_pp] | 1.1.1–1.1.3 | `ols` and `huber` on raw, range-limited and pre-processed data; train and test samples; the pre-processing refitted inside the model; reading the metrics |
| [2.1 Covariates][ml_model_cov] | 2.1.1–2.1.2 | Soil texture as covariate, on raw and on PCA-reduced spectra |
| [2.2 Regressors][ml_model_regr] | 2.2.1–2.2.5 | `plsr`, `svr`, `cubist`, `randfor` and `mlp` on the full pre-processed spectra; hyper-parameters |
| [3 Arrays and plot layouts][ml_model_plots] | 3.1.1, 3.2.1 | Many indicators with one regressor, one indicator with many regressors; grid figures |
| [4 Feature importance][ml_model_imp] | 4.1.1–4.2.4 | Feature and permutation importance, `importance_only`, importance next to the model results |
| [5 Output and manifest][ml_model_out] | | Saved models, manifests, results and plots |

## How a run works

`regression_modeling` fits one model per **indicator** in `indicator_array` and **regressor** in
`regressor_array`, in one or two evaluation modes:

- **train/test** (`traintest: true`, tag `tt`): the samples are split into training and test samples
  (`test_size`); the model is fitted on the training samples and scored on the test samples;
- **k-fold** (`kfold: true`, tag `kf`): every sample is predicted once by a model fitted on the other folds
  (`folds`); the saved model is then fitted on all samples.

Two indicators, three regressors and both modes give 2 × 3 × 2 = 12 models.

## Models always start with unprocessed spectra

In the process file, `dataframe` can name the compiled dataset with unprocessed spectra (`raw`) or a
pre-processed dataframe, e.g. `neospectra_10nm_olif_texture.snv.d1.mc`. With a pre-processed dataframe the
model does not train on its spectra:

- The companion JSON of the dataframe records the unprocessed parquet the chain started from
  (`source_parquet`) and every step of the chain (`preprocessing_chain`).
- `regression_modeling` reads the spectra from `source_parquet` and the indicators and covariates from the
  dataframe you named. Indicator steps of the chain (outliers set to missing, indicator range) therefore carry
  over. The two parquets must hold the same samples in the same order, otherwise the run stops.
- The steps of the chain become steps of the model's pipeline, and are fitted on the training samples of each
  model (each fold in k-fold). The run prints
  `Refitting <n> preprocessing step(s) on the training samples, from <source>.parquet`.

This gives two guarantees:

- No information from the test samples leaks into the model through the pre-processing (the mean of
  meancentring, the PCA components, …), so the test and k-fold metrics are honest.
- The saved model holds the whole chain. It takes unprocessed spectra and applies the pre-processing itself when
  predicting new samples ([Predict and register][ml_predict]).

With `refit_preprocessing: false` the model trains on the pre-processed spectra as they are, fitted on all
samples, and the run warns that the test metrics are optimistic. With `raw` there is no chain to refit. Details
and the steps that are not refitted: [The pre-processing is refitted inside the model][ml_model_refit].

## Models results

Each run prints, per indicator, the main metrics with one column per regressor and mode, e.g.:

```
    ── c-org ──
                  ols/tt              randfor/tt          ols/kf              randfor/kf
    r2            <value>             <value>             <value>             <value>
    rmse          <value>             <value>             <value>             <value>
    rpiq          <value>             <value>             <value>             <value>
```

and saves the models, their manifests and a results JSON. If asked, plots are displayed and/or saved with each model ([Output and manifest][ml_model_out]).

## Parameters

**Input and targets**

| Parameter | Description |
|---|---|
| `project_root_fp` | The folder with the compiled dataframe |
| `dataframe` | The dataframe to model: `raw` (the compiled dataset, default), a dataframe name, or `previous` ([The input dataframe][input_df]) |
| `indicator_array` | The target indicators; empty = every indicator of the dataframe |
| `covariate_array` | Covariates next to the spectra: empty = none, `all`, or names. See [Covariates][ml_model_cov] |
| `class_grouping` | Pool the classes of a nominal covariate before encoding. See [Covariates][ml_model_cov] |
| `indicator_transformation` | Transformation of the indicator before fitting: `none` (default), `default`/`standard`, a set name or a JSON file |
| `indicator_standardisation` | Z-score standardisation of the indicator: same forms, default `none` |

**Regressors and evaluation**

| Parameter | Description |
|---|---|
| `regressor_array` | The regressors to fit, see [Available regressors][ml_model_avail] |
| `model_parameters_fp` | Hyper-parameters: `default` (the `standard` set), `none` (scikit-learn defaults), a set name or a JSON file ([ML settings][ml_settings]) |
| `traintest` | Train/test evaluation (default `true`) |
| `test_size` | Fraction of test samples (default 0.3) |
| `kfold` | K-fold cross-validation (default `false`) |
| `folds` | Number of folds (default 10) |
| `refit_preprocessing` | Refit the pre-processing chain inside the model on the training samples (default `true`) |
| `model_fp` | Folder for the saved models, relative to the project root. Empty = `<project_root_fp>/models` |

**Plots and importance**

| Parameter | Description |
|---|---|
| `show_observed_predicted` / `save_observed_predicted` | Observed/predicted scatter (default shown, not saved) |
| `show_feature_importance` / `save_feature_importance` | Feature importance (default neither) |
| `show_permutation_importance` / `save_permutation_importance` | Permutation importance (default neither) |
| `n_top_features_in_plot` | Most important features in a saved permutation figure (default 8; 0 = all) |
| `targetfeaturesymbols` | Colour per indicator in the scatter plots: `default` (the `plot_symbol` set `targetfeaturesymbols`), a set name or a JSON file. See [Plot symbols](#plot-symbols) |
| `regressionmodelsymbols` | Marker per regressor: `default` (the set `regressionmodelsymbols`), a set name or a JSON file |
| `plot_layout` | Compose the shown plots into grid figures: `default`, a preset, a set name, a JSON file or `none`. See [Plot layout][ml_model_layout] |
| `save_plot_layout` | Also save the grid figures (default `false`) |
| `importance_only` | Fit the regressors and map the importance only: no metrics, models or results. See [Importance without the model tests][ml_model_imp_only] |

The indicator transformation and standardisation are set only in `regression_modeling`. A compiled dataframe
always holds the indicators in their native unit: no process writes transformed or standardised indicators
(the plot processes transform for display only), so the transformation can not be applied twice. Both default
to `none`; a transformation used in a plot is not taken over. The model fits them on the training samples and
returns predictions in the native unit ([ML settings][ml_settings_trans]).

### Plot symbols

`targetfeaturesymbols` and `regressionmodelsymbols` take a JSON file the same way as the other settings, e.g.
`"regressionmodelsymbols": "./project_neospectra/plot_symbol/regressor_markers.json"`:

```json
{
  "regressionModelSymbols": {
    "plsr":    {"marker": "o", "size": 50},
    "randfor": {"marker": "h", "size": 50}
  }
}
```

and for the indicators `{"targetFeatureSymbols": {"c-org": {"color": "black", "alpha": 0.6}, …}}`. The keys are
the regressor keys and the indicator names of the dataframe. A file replaces the whole set: a regressor not in it
is drawn with marker `o`, size 40, an indicator in `steelblue`, alpha 0.6. To change a few entries and keep the
rest, create a custom `plot_symbol` set on the preset ([Create a custom set][ml_settings_custom]).

A dataframe compiled without indicators is a dataset for prediction ([Compile for prediction][compile_predict])
and can not be modelled:
`ERROR: no valid indicator columns. <name> is a dataset for prediction (compiled without indicators).`

`plot_layout`, `save_plot_layout` and `importance_only` are registered by `setup_processes.ipynb`; rerun it after
updating xspatula_ai4sh.
{: .notice--info}

## Next step

[1 Pre-processing effects][ml_model_pp]: the first models, and how pre-processing changes them.

[compile]: {{ site.baseurl }}/machine_learning/compile/
[compile_predict]: {{ site.baseurl }}/machine_learning/compile/#compile-for-prediction
[input_df]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#the-input-dataframe
[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[ml_settings_trans]: {{ site.baseurl }}/machine_learning/ml_settings/#indicator-transformation-and-standardisation
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
[ml_model_pp]: {{ site.baseurl }}/machine_learning/ml_model/preprocessing/
[ml_model_refit]: {{ site.baseurl }}/machine_learning/ml_model/preprocessing/#the-pre-processing-is-refitted-inside-the-model
[ml_model_cov]: {{ site.baseurl }}/machine_learning/ml_model/covariates/
[ml_model_regr]: {{ site.baseurl }}/machine_learning/ml_model/regressors/
[ml_model_avail]: {{ site.baseurl }}/machine_learning/ml_model/regressors/#available-regressors
[ml_model_plots]: {{ site.baseurl }}/machine_learning/ml_model/plots/
[ml_model_layout]: {{ site.baseurl }}/machine_learning/ml_model/plots/#plot-layout
[ml_model_imp]: {{ site.baseurl }}/machine_learning/ml_model/importance/
[ml_model_imp_only]: {{ site.baseurl }}/machine_learning/ml_model/importance/#importance-without-the-model-tests
[ml_model_out]: {{ site.baseurl }}/machine_learning/ml_model/output/
[ml_settings_custom]: {{ site.baseurl }}/machine_learning/ml_settings/#create-a-custom-set
