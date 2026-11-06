---
title: "ML Modeling 1: Pre-processing effects"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "The first regression models: OLS and Huber on raw, range-limited and pre-processed data, the train and test samples, the pre-processing refitted inside the model, and how to read r², RMSE and RPIQ."
permalink: /machine_learning/ml_model/preprocessing/
author_profile: false
date: 2026-11-06 08:00:00 +0100
last_modified_at: 2026-11-06 12:00:00 +0100
---

Block 1 of the notebook `ml_model.ipynb` fits the same two regressors, `ols` (ordinary least squares) and
`huber` (robust to outliers), on three versions of the same samples: raw, with an indicator range, and
pre-processed. The comparison shows what indicator cleaning and spectral pre-processing do to a model.

## 1.1.1 Raw data

**Process file**: `project_neospectra/process/ml_model/regression_modeling_raw_tt.json`

```json
{
  "process": [
    {
      "process": "regression_modeling",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm",
        "dataframe": "raw",
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

`raw` is the compiled dataset itself, with the unprocessed spectra (116 bands of 10 nm). `model_parameters_fp`
points to a JSON file that holds the same hyper-parameters as the `standard` regressor set in the database
([Available regressors][ml_model_avail]); `"model_parameters_fp": "default"` gives the same results. The file is
there to copy and edit.

![OLS and Huber on raw data, train/test]({{ "/assets/media/ml_model/1-1-1_raw_tt.png" | relative_url }})

Each panel plots the predicted against the observed values of the test samples, with r², RMSE, RPIQ and the
number of training and test samples. The dashed line is the 1:1 line.

- **OLS** has nearly as many unknowns (116 bands plus intercept) as training samples, so it fits the training
  samples almost exactly and predicts the test samples badly. A negative r² means the model predicts worse than
  the mean of the test samples.
- **Huber** is held back by its regularisation and its robust loss, and comes closer to the line. A few test
  samples with very high observed values (the indicator outliers) are far off for both regressors.
- On the unscaled raw spectra Huber may stop before it converges:
  `ConvergenceWarning: lbfgs failed to converge after 100 iteration(s)`. The model is still saved. Pre-processing
  (cell 1.1.3) removes the warning; a custom regressor set with a higher `max_iter` is the other way.

## 1.1.2 Indicator range

**Process file**: `project_neospectra/process/ml_model/regression_modeling_range_tt.json`, the same as above with
`"dataframe": "neospectra_10nm.ir"`: the compiled dataset after the [indicator range][pp1_range] (cell 1.1.3 of
`compile_dataframe.ipynb`, see [Indicator steps before or after compiling][compile_ind]).

![OLS and Huber on range-limited indicators]({{ "/assets/media/ml_model/1-1-2_range_tt.png" | relative_url }})

The values outside the range are missing, so the axes span only the range. Huber now fits the bulk of the
samples; OLS still overfits the raw spectra.

## 1.1.3 Pre-processed indicators and spectra

**Process file**: `project_neospectra/process/ml_model/regression_modeling_pre-processed_tt.json`, with
`"project_root_fp": "./project_neospectra/data/compiled/neospectra_olif_10nm"` and
`"dataframe": "neospectra_olif_10nm.snv.d1.mc.pca5"`: the indicators cleaned of outliers before compiling, and the
spectra after SNV, first derivative, meancentring and PCA to 5 components (see
[Pre-process the compiled dataset][compile_chain]).

![OLS and Huber on pre-processed data]({{ "/assets/media/ml_model/1-1-3_pre-processed_tt.png" | relative_url }})

With 5 components instead of 116 bands OLS no longer overfits, and the two regressors give similar results:
the pre-processing, not the regressor, made the difference.

## Train and test samples

The test samples are chosen from the `sample_id`, not from the row order: the `round(test_size × n)` samples
with the lowest hash of their `sample_id` are the test samples. The same sample is therefore always a test
sample for the same `test_size`, whatever order or subset the dataframe has.

Band selection (permutation and univariate selection in ML pre-processing) uses the same rule. Give the
selection and the modeling the same `test_size`, otherwise the model's test samples include samples the
band selection has already seen, and the test metrics are too optimistic. The model prints a note when the two
differ.

**Train/test or k-fold.** A single train/test split of about 150 samples scores 40–50 test samples, and another
split can give a clearly different result. K-fold predicts every sample once and is the more reliable estimate
on a small dataset; use it to choose between models. The train/test model is the one to keep when you want an
untouched test set.

## The pre-processing is refitted inside the model

With `refit_preprocessing: true` (the default), the model does not train on the pre-processed dataframe
directly. It starts from the unprocessed source spectra (`source_parquet` in the dataframe's companion JSON) and
replays its `preprocessing_chain` as steps of a scikit-learn pipeline, fitted on the training samples only.
Meancentring, scaling, the MSC reference spectrum, PCA, Ward clusters, variance threshold and univariate band
selection therefore never see the test samples, and their fitted values are saved with the model.

Three steps are not refitted:

- **Outlier detection**: detected indicator outliers were set to missing using
  all samples.
- **[Indicator range][pp1_range]** (`.ir`): the limits are fixed, nothing is learned from the samples.
- **Permutation selection**: the selected bands are taken as fixed. They are clean only if the selection used
  the same `test_size` (see above).

When outlier detection or permutation selection is in the chain of the dataframe you model, the run prints a
note at the start, e.g.:

```
    ⚠️  01_olif: indicator outliers were set to missing using all samples (not refitted)
```

The note appears when the outlier step ran on the compiled dataset itself, e.g. for
`neospectra_10nm.olif.snv.sg5.d1.mc.wc40.pca5`. Outliers removed in the indicator folder before compiling, as in
cell 1.1.3, are recorded under `inputs` in the compiled companion JSON (see [Compile dataframe][compile]), and
the model prints no note about them.

The indicator transformation and standardisation are part of the pipeline. They are fitted on the training
samples, and the model converts its predictions back to the native unit of the indicator: the metrics, the
plots and the predictions of new samples are all in the native unit.

## Reading the metrics

| Metric | Meaning |
|---|---|
| `r2` | Share of the variance of the observed values that the predictions explain; 1 is perfect, 0 is no better than the mean, negative is worse than the mean |
| `rmse` | Root mean square error, in the unit of the indicator |
| `rpiq` | Interquartile range of the observed values divided by the RMSE: the common yardstick in soil spectroscopy, the higher the better |

RMSE and RPIQ depend on the spread of the samples: compare models of the same indicator on the same samples,
not across datasets. The results JSON also has `mae`, `mape`, `medae`, `bias` and `ccc` ([Output and manifest][ml_model_out]).

## Next step

[2.1 Covariates][ml_model_cov]: add soil texture next to the spectra.

[compile]: {{ site.baseurl }}/machine_learning/compile/
[compile_ind]: {{ site.baseurl }}/machine_learning/compile/#indicator-steps-before-or-after-compiling
[compile_chain]: {{ site.baseurl }}/machine_learning/compile/#pre-process-the-compiled-dataset
[pp1_range]: {{ site.baseurl }}/machine_learning/ml_preprocess1/range/
[ml_model_cov]: {{ site.baseurl }}/machine_learning/ml_model/covariates/
[ml_model_out]: {{ site.baseurl }}/machine_learning/ml_model/output/
[ml_model_avail]: {{ site.baseurl }}/machine_learning/ml_model/regressors/#available-regressors
