---
title: "ML Modeling 2.1: Covariates"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Covariates next to the spectra in regression_modeling: numeric covariates and classes such as soil texture, one-hot encoding, class grouping, standardisation, unseen classes, and models on covariates without spectra."
permalink: /machine_learning/ml_model/covariates/
author_profile: false
date: 2026-11-06 08:00:00 +0100
last_modified_at: 2026-11-06 12:00:00 +0100
---

Besides the spectra, a model can use **covariates**: other observations representing the same samples, selected with
`select_covariate_only` and compiled with the indicators and spectra (see [Compile dataframe][compile]). A
covariate is either numeric (e.g. a field pH) or nominal, a class with unit `class` (e.g. soil texture).

Section 2.1 of the notebook `ml_model.ipynb` adds the USDA soil texture class a covariate to the modeling. The texture class is derived from the clay, silt and sand content when an
observation is imported ([Derived soil texture classes][obs_texture]).

## 2.1.1 Raw spectra with texture

**Process file**: `project_neospectra/process/ml_model/regression_modeling_texture_raw_tt+kf5.json`

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
In the process file, soil texture is grouped into coarse, medium and fine. Both evaluation modes run, so every indicator and regressor gives a
train/test and a k-fold panel:

![OLS and random forest with texture on raw spectra]({{ "/assets/media/ml_model/2-1-1_texture_raw_tt-kf5.png" | relative_url }})

On the raw spectra OLS overfits as in [cell 1.1.1][ml_model_pp]; random forest copes better. The k-fold results
are lower than train/test: they score every sample, including the difficult ones a single test set may miss.

## 2.1.2 Pre-processed spectra with texture

**Process file**: `project_neospectra/process/ml_model/regression_modeling_texture_pre-processed_tt+kf5.json`,
has the same model settings as the model above, but reads another dataframe with outlier-cleaned indicators and spectra pre-processing (SNV, first derivative, meancentring and 5 PCA components).

![OLS and random forest with texture on pre-processed spectra]({{ "/assets/media/ml_model/2-1-2_texture_pre-processed_tt-kf5.png" | relative_url }})

Pre-processing improves both regressors far more than the covariate does. To see what the covariate adds,
compare a model with covariates against a model of the **same** dataframe with `covariate_array` empty (cells
4.2.3 and 4.2.4 do that, see [Feature importance][ml_model_imp_side]).

## How covariates are used

- **`covariate_array`**: empty means spectra only, `all` means every covariate of the compiled dataset (the
  covariates listed in its companion JSON), or list the covariate names. Nominal covariates become one 0/1
  column per class (one-hot encoding).
- **`class_grouping`** pools the classes of a covariate before encoding: an array of objects
  `{"covariate": <name>, "groups": [[class, class, …], …]}`. Every class in the data must be placed in a group;
  otherwise the run stops and lists the classes it could not place
  (`class_grouping for "soil texture usda" does not place the class(es): …`). A group is named by its classes
  joined with `|`, e.g. `sand|loamy sand|sandy loam`; in plots it is labelled by its first class and the number
  of others, e.g. `sand +2`.
- **Predefined groupings** need no `class_grouping`: they are separate covariates. Texture is derived into four
  nominal classifications: `soil texture usda` (12 classes), `soil texture usda major group` (4 groups),
  `soil texture iuss` and `soil texture iuss major group`. 
- **Numeric covariates are standardised** with the mean and standard deviation of the training samples:
  (value − mean) / std. The mean and std are fitted with the model, saved in it and applied unchanged when
  predicting. Without it, a covariate's weight in `plsr`, `knn` and `svr` would depend on its unit (clay in % or
  in g/kg) and its variance. Consequences:
  - the unit of a numeric covariate does not change the predictions;
  - for linear regressors a numeric covariate's coefficient, and its feature importance (|coefficient|), is per
    standard deviation, not per unit;
  - tree-based regressors (`dectree`, `randfor`) are unaffected.

  A standardised covariate has variance 1. A band of meancentred reflectance spectra has a far smaller
  variance, so in `plsr`, `knn` and `svr` one numeric covariate can weigh as much as many bands. Choose
  autoscaling in the chain to put the bands on the same footing.
- **A covariate that does not vary in the training samples has no effect**: a numeric covariate with one value
  (it is only centred, an all-zero column) or a class covariate with one class. The run prints a note once per
  indicator and mode:
  `⚠️  <indicator> [tt|kf]: covariate(s) <labels> do not vary in the training samples — no effect`.
- **Samples without a value for each covariate of the model are skipped**, both when modelling and when
  predicting (they get no prediction, and the number is printed). A sample missing one of two covariates is
  skipped too.
- **A class the model has not seen cannot be predicted.** The model learns one column per class from its
  training samples; a class without training samples has no column. In train/test, test samples of such a class
  are skipped, and the run names the class:
  `<indicator>: <n> test sample(s) with a <covariate> class not in the training samples (<classes>), skipped`.
  With k-fold, a class with fewer samples than folds is missing from some training folds; its samples in those
  folds are predicted without a class column, which biases the k-fold metrics. The run warns
  (`… class(es) with fewer samples than folds (<folds>): <class> (<n>) … — consider class_grouping`). The cure in
  both cases is the same: pool the rare classes with a `class_grouping`, or use a major-group covariate.
- **The same train-only fitting as the spectra:** the spectra's pre-processing chain is refitted on the training
  samples, together with the covariates, in one saved model. That model needs only the unprocessed spectra plus
  the covariates to predict.
- A covariate of the model is never an indicator. Other covariates of the dataset are only modelled as
  indicators when named in `indicator_array`.

**Which classification?** With few samples (e.g. a few hundred samples over the 12 USDA texture classes), use a
coarse grouping or a major-group covariate, so that every class has enough samples.

**A covariate derived from the indicator.** The texture class is derived from clay, silt and sand, so a clay model
with texture as covariate is not independent of its target. The notebook models clay with texture as an
illustration of a texture entered by a soil expert in the field.
{: .notice--warning}

## How covariates affect the regressors

The covariate columns are inputs next to the bands; each regressor uses them in its own way. Numeric covariates
are standardised (variance 1), class covariates are 0/1 columns.

| Regressor | Numeric covariate | Class covariate |
|---|---|---|
| `ols`, `theil_sen`, `huber` | Adds a linear term; the coefficient is per standard deviation | Shifts the prediction by a fixed amount per class; the spectral relation is the same in every class |
| `svr` linear | As the linear regressors, within the `C` and `epsilon` regularisation | A fixed shift per class |
| `svr` rbf, `knn` | Enters the distance between samples; with variance 1 it weighs as much as many meancentred bands | Samples of different classes are further apart, so neighbours tend to share the class |
| `plsr` | Enters the components together with the bands; with variance 1 it can dominate meancentred bands | Enters the components; acts mainly as a shift per class |
| `dectree`, `randfor` | Split on like any feature; scale does not matter | Split on: the trees can model different spectral relations per class |
| `cubist` | Used in the rule conditions and in the linear model of each rule | Rules per class, each with its own linear model |
| `mlp` | All inputs are standardised inside the model, so bands and covariates start on the same footing | Can interact with the bands in the hidden layer |

- A fixed shift (linear regressors, linear `svr`, largely `plsr`) helps when a class raises or lowers the
  indicator at the same spectrum. Only the tree and rule based regressors, and `mlp`, can let the spectral
  relation differ between classes.
- The weight problem of `svr` rbf, `knn` and `plsr` is a matter of scale: choose autoscaling in the chain to put
  the bands on the same footing as the standardised covariates.

## Covariates without spectra

A compiled dataset may hold indicators and covariates only (see
[Compile directly from the database][compile_from_db]). With `covariate_array` set, `regression_modeling` then
models the indicators from the covariates alone; the model, its plots and `predict_independent` work as with
spectra. Without spectra and without covariates the run stops:
`ERROR: no spectral columns and no covariates (covariate_array) — nothing to model on.` The spectral processes
(chains, band selection) need spectra.

## Next step

[2.2 Regressors][ml_model_regr]: the advanced regressors on the full pre-processed spectra.

[compile]: {{ site.baseurl }}/machine_learning/compile/
[compile_from_db]: {{ site.baseurl }}/machine_learning/compile/#compile-directly-from-the-database
[obs_texture]: {{ site.baseurl }}/observation/#derived-soil-texture-classes
[ml_model_pp]: {{ site.baseurl }}/machine_learning/ml_model/preprocessing/#111-raw-data
[ml_model_regr]: {{ site.baseurl }}/machine_learning/ml_model/regressors/
[ml_model_imp_side]: {{ site.baseurl }}/machine_learning/ml_model/importance/#importance-next-to-the-model-results
