---
title: "ML Pre-processing 2"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Supervised pre-processing of compiled dataframes: band selection by univariate F-test, permutation importance, recursive feature elimination and tree-based importance, on the training samples only, and how regression_modeling replays a selection."
permalink: /machine_learning/ml_preprocess2/
author_profile: false
date: 2026-11-06 16:00:00 +0100
last_modified_at: 2026-11-06 16:00:00 +0100
---

[ML pre-processing 1][pp1] works on spectra and indicators separately and never looks at the target. ML
pre-processing 2 selects the bands that matter for a target indicator, and for some methods for a regressor.
It needs the spectra and the indicators in one dataframe: a compiled dataset ([Compile dataframe][compile]),
usually after an unsupervised chain such as `.snv.d1.mc`.

## Notebook

**Path**: `xspatula_ai4sh/ai4sh/project_neospectra/ml_preprocess_compiled.ipynb`

Set `scheme_file = '../scheme_ai4sh.json'` in the setup cell (the default). Every other cell runs one process
file from `project_neospectra/process/ml_preprocess_compiled/`. All cells use the compiled folder
`./project_neospectra/data/compiled/neospectra_10nm_olif_texture` and start from
`neospectra_10nm_olif_texture.snv.d1.mc` (115 bands after the derivative).

| Section | Page | Process | Suffix |
|---|---|---|---|
| 1.1 Univariate selection | [Band selection][pp2_sel] | `spectra_indicator_univariate_selection` | `.uv-<indicator>`, `.uv` |
| 1.2 Permutation, RFE and tree-based | [Band selection][pp2_imp] | `spectra_indicator_permutation_selection` | `.<indicator>-<regressor>-pms`, `-rfe`, `.<indicator>-tree` |
| 2 Wavelets | [Wavelet coefficients][pp2_wavelet] | `chemometric_chain`, `spectra_indicator_permutation_selection` | `.wt`, then a selection suffix |
| 3 Model the selections | [Model the selections][pp2_model] | `regression_modeling` | — |

## Why select bands

A spectrum of 115 bands has far more columns than a typical soil dataset has samples, and neighbouring bands
carry nearly the same information. Band selection keeps the bands that relate to the indicator and drops the
rest. A smaller input can predict as well or better, is faster to fit, and points to the absorption features
behind the prediction.

## Training samples only

- The bands are selected on the **training samples** only. The test samples are the same holdout as in
  `regression_modeling`: the `round(test_size × n)` samples with the lowest hash of their `sample_id`
  ([Train and test samples][ml_model_tt]). The run prints
  `Selecting from <n> training samples (test_size=0.30 held out)`.
- Give the selection and the model the same `test_size` (default 0.3). Otherwise the model's test samples
  include samples the selection has seen, and the model prints a note (see below). `test_size: 0` selects on all
  samples.
- Samples without a value for the indicator are left out of that indicator's selection.

## Indicators and covariates

`indicator_array` lists the indicators to select bands for; empty means every numeric indicator of the
dataframe. The covariates of a compiled dataset (e.g. soil texture) are then left out:

```
    Covariates not processed (name them in indicator_array to include): <covariates>
```

Covariates are never selected as bands. They are copied unchanged to the output dataframe, so a model on the
selection can still use them ([Covariates][ml_model_cov]).

`separate_selections` decides how several indicators are handled:

- `true`: one selection, and one output dataframe, per indicator (and per regressor and selector).
- `false` (default): one selection for all indicators together, from the scores averaged over the indicators.
  The scores of each indicator are first scaled to 0–1, so an indicator with larger scores does not dominate.
  Only useful when the indicators are related, e.g. organic carbon and total nitrogen.

## Output names

Every selection saves a new dataframe next to its input, with the selected bands, all other columns of the
input and a companion JSON whose `preprocessing_chain` gets one more entry: the selection, its
parameters, the `selected_bands` and the `holdout`.

| Suffix | Selection |
|---|---|
| `uv-<indicator>` | Univariate selection for one indicator (spaces → hyphens), e.g. `.uv-c-org` |
| `uv` | Univariate selection, all indicators together |
| `<indicator>-<regressor>-pms` | Permutation importance, e.g. `.c-org-plsr-pms` |
| `<indicator>-<regressor>-rfe` | Recursive feature elimination, e.g. `.c-org-randfor-rfe` |
| `<indicator>-tree` | Tree-based (extra trees) importance, e.g. `.c-org-tree` |
| `combined-<regressor>-pms`, `combined-<regressor>-rfe`, `combined-tree` | Permutation, RFE or tree-based, all indicators together |

`name_extension` (1-6 letters, digits or `-`) is added after a `#`, e.g. `.c-org-plsr-rfe#cv5`, to keep
selections with different settings apart ([Name extension][name_extension]).

## How a model uses a selection

`regression_modeling` on a selection does what it does for any pre-processed dataframe
([Models always start with unprocessed spectra][ml_model_refit]): it starts from the compiled dataset, refits
the unsupervised steps on its training samples, and then applies the selection.

- **Univariate selection for the same indicator** is refitted: the model selects its own `n_features` bands
  from its own training samples (each fold in k-fold).
- **All other selections keep their bands**: permutation, RFE, tree-based, a combined univariate selection,
  and a univariate selection made for another indicator. The model prints a note:

```
    ⚠️  04_c-org-plsr-rfe: band selection fixed from the training samples (k-fold metrics still include it)
```

Fixed bands are clean for the train/test metrics, because the selection never saw the test samples. They are
not clean for k-fold: every sample was a training sample of the selection, so k-fold metrics of a fixed
selection are optimistic. **Compare selections with train/test**, not k-fold. When the `test_size` of the
selection and the model differ, the note says so:

```
    ⚠️  04_c-org-plsr-rfe: band selection was made with test samples included (holdout 0.2, model test_size 0.3)
```

## Next step

[1 Band selection][pp2_sel]: univariate, permutation, RFE and tree-based selection.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[name_extension]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#name-extension
[compile]: {{ site.baseurl }}/machine_learning/compile/
[ml_model_tt]: {{ site.baseurl }}/machine_learning/ml_model/preprocessing/#train-and-test-samples
[ml_model_cov]: {{ site.baseurl }}/machine_learning/ml_model/covariates/
[ml_model_refit]: {{ site.baseurl }}/machine_learning/ml_model/#models-always-start-with-unprocessed-spectra
[pp2_sel]: {{ site.baseurl }}/machine_learning/ml_preprocess2/selection/
[pp2_imp]: {{ site.baseurl }}/machine_learning/ml_preprocess2/selection/#12-permutation-rfe-and-tree-based-importance
[pp2_wavelet]: {{ site.baseurl }}/machine_learning/ml_preprocess2/wavelet/
[pp2_model]: {{ site.baseurl }}/machine_learning/ml_preprocess2/model/
