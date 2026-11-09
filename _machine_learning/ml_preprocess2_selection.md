---
title: "ML Pre-processing 2: Band selection"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Supervised band selection on compiled dataframes: univariate F-test (SelectKBest), permutation importance, recursive feature elimination with and without cross-validation (RFE, RFECV) and tree-based importance; parameters, plots and the number of bands kept."
permalink: /machine_learning/ml_preprocess2/selection/
author_profile: false
date: 2026-11-06 16:00:00 +0100
last_modified_at: 2026-11-06 16:00:00 +0100
---

Section 1 of the notebook `ml_preprocess_compiled.ipynb`. See [ML pre-processing 2][pp2] for the training
samples, covariates, output names and how a model replays a selection.

## The methods

| Selector | Process | Ranks a band by | Regressor |
|---|---|---|---|
| Univariate | `spectra_indicator_univariate_selection` | The F-test of its linear relation to the indicator (scikit-learn `SelectKBest`, `f_regression`) | none |
| `permutation` | `spectra_indicator_permutation_selection` | The drop in r² when the band is shuffled, out-of-fold on the training samples | yes |
| `rfe` | `spectra_indicator_permutation_selection` | The round in which recursive feature elimination removes it: the regressor is refitted and the weakest bands removed until one is left | yes |
| `tree_based` | `spectra_indicator_permutation_selection` | Impurity importance of an extra-trees ensemble | none (its own) |

- **Univariate** scores each band on its own, is fast and needs no model. Two bands that matter only
  together, e.g. the two sides of an absorption feature, can both score low.
- **Permutation** asks what the fitted model loses without each band. In each fold a model
  fitted on the other folds of the training samples is scored with the band shuffled; the repeats of all
  folds are pooled.
- **RFE** ranks by the regressor's own weights: the absolute coefficients (`ols`, `huber`, linear `svr`, `plsr`)
  or the impurity importance (`dectree`, `randfor`). `knn`, `mlp` and `svr` with `kernel: rbf` have neither,
  and use permutation importance instead (message displayed).
- **Tree-based** fits an extra-trees ensemble (`n_estimators`, default 20) and ranks by how much the splits on
  each band reduce the error.

Correlated bands share importance: when two neighbouring bands carry the same information, permutation and
tree-based importance split it between them, and shuffling one barely hurts the model. A band low in the
ranking is not necessarily without information.

## 1.1.1 Univariate selection with SelectKBest

**Process file**: `project_neospectra/process/ml_preprocess_compiled/univariate_selection_c-org+n-tot_k30.json`

```json
{
  "process": [
    {
      "process": "spectra_indicator_univariate_selection",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_olif_texture",
        "dataframe": "neospectra_10nm_olif_texture.snv.d1.mc",
        "indicator_array": ["c-org", "n-tot"],
        "separate_selections": true,
        "n_features": 30,
        "show": true,
        "save": true
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `project_root_fp` | The compiled folder |
| `dataframe` | Input dataframe: a file name, `raw` (the compiled dataset) or `previous` |
| `indicator_array` | Indicators to select for; empty = all indicators, not the covariates |
| `separate_selections` | `true`: one selection per indicator; `false` (default): one for all ([Indicators and covariates][pp2_ind]) |
| `n_features` | Number of bands to keep; the result is saved without asking |
| `univariate_selector_fp` | Optional JSON `{"SelectKBest": {"n_features": 20}}`, used when `n_features` is 0; you are then asked before saving |
| `test_size` | Test samples held out, default 0.3; the same as in `regression_modeling` |
| `name_extension` | Optional, added after `#` |
| `colormap`, `max_spectra` | Colour ramp and number of spectra in the plot |
| `show`, `save` | Show the plot, save it under `plot/` |

One figure per indicator: the input spectra, the kept bands (highest F-score) and the discarded bands. The dots
under the input panel mark kept (green) and discarded (red) bands; `valid` is the number of training samples
with a value for the indicator. The lines break where bands are left out, so each run of neighbouring bands
shows as its own segment.

![Univariate selection of 30 bands for c-org]({{ "/assets/media/ml_preprocesses_compiled/1-1-1-univariate-c-org.png" | relative_url }})

![Univariate selection of 30 bands for n-tot]({{ "/assets/media/ml_preprocesses_compiled/1-1-1-univariate-n-tot.png" | relative_url }})

The two indicators keep largely the same regions: organic carbon and total nitrogen are related in soil.

With `n_features` set, the selections are saved without asking, otherwise the run asks first:

```
    Accept and save uv-c-org result? [y/n]:
```

## 1.2 Permutation, RFE and tree-based importance

| Parameter | Description |
|---|---|
| `project_root_fp`, `dataframe`, `indicator_array`, `separate_selections`, `test_size`, `name_extension` | As above |
| `selector_array` | `permutation`, `rfe`, `tree_based`; one or more |
| `regressor_array` | Regressors for `permutation` and `rfe`, see [Available regressors][ml_model_avail]; one selection per regressor |
| `model_parameters_fp` | Hyper-parameters of the regressors: `default`, a set name or a JSON file ([Regressors][ml_model_regr]) |
| `selector_parameters` | Selector settings: `default`, a set name or a JSON file (see below) |
| `n_features` | Bands to keep. 0 = the number RFECV chose, otherwise you are asked after the plot |
| `n_top_features_in_plot` | Bars in the plot; 0 = all. Band labels are left out above 40 bars |
| `show`, `save` | Show the plots; save them, and the band scores, under `plot/selection_plot/` and `band_selection/` |

The selector settings, e.g. `project_neospectra/spectral_band_selector/selector_rfecv5.json`:

```json
{
  "permutation": {"permutation_repeats": 6, "folds": 5},
  "rfe": {"CV": 5, "step": 0.1},
  "tree_based": {"n_estimators": 100}
}
```

| Key | Description | Default |
|---|---|---|
| `permutation.permutation_repeats` | Shuffles per band and fold | 6 |
| `permutation.folds` | Folds of the out-of-fold permutation | 5 |
| `rfe.CV` | 0 = RFE; n > 0 = RFECV with n folds, which picks the number of bands | 0 |
| `rfe.step` | Bands removed per round: ≥ 1 a number, < 1 a fraction of all bands (rounded down) | 1 |
| `tree_based.n_estimators` | Trees of the extra-trees ensemble | 20 |

One run gives one selection per indicator × regressor × selector. With `n_features` 0 and no RFECV, the run
asks after each plot:

```
    How many top bands to retain for "c-org" / plsr / rfe? (0 = skip saving, 30 bands shown in plot):
```

With `n_features` set, the top bands are saved without asking. With `save: true` the plots go to
`plot/selection_plot/<name>.png`, and the scores of every band, most important first, to
`band_selection/<input>.<name>.json`, where `<name>` is the selection suffix, e.g. `c-org-plsr-rfe`.

### 1.2.1 Permutation, RFE and tree-based importance without cross-validation

**Process file**: `project_neospectra/process/ml_preprocess_compiled/band_selection_permutation+rfe+tree_c-org_plsr_k30.json`

```json
{
  "process": [
    {
      "process": "spectra_indicator_permutation_selection",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_olif_texture",
        "dataframe": "neospectra_10nm_olif_texture.snv.d1.mc",
        "indicator_array": ["c-org"],
        "selector_array": ["permutation", "rfe", "tree_based"],
        "regressor_array": ["plsr"],
        "model_parameters_fp": "./project_neospectra/regressors/regression_model_parameters_tuned.json",
        "separate_selections": true,
        "n_features": 30,
        "n_top_features_in_plot": 30,
        "show": true,
        "save": true
      }
    }
  ]
}
```

Three selections of 30 bands for c-org: `.c-org-plsr-pms`, `.c-org-plsr-rfe` and `.c-org-tree`. The
plots show the 30 kept bands, the most important at the top.

![Permutation importance, c-org, plsr]({{ "/assets/media/ml_preprocesses_compiled/1-2-1-permutation-c-org-plsr.png" | relative_url }})

Permutation importance is noisy on a small dataset: the standard deviation is often larger than the mean, and
the order of the lower bands changes with the folds. More `permutation_repeats` and `folds` steady it, at the
cost of time.

![RFE ranking, c-org, plsr]({{ "/assets/media/ml_preprocesses_compiled/1-2-1-rfe-c-org-plsr.png" | relative_url }})

The RFE score is the number of elimination rounds a band survived. RFE removes bands until one is left, so with
`step` 1 the 115 bands get the scores 1 to 115: the last band left scores highest. Bands still in the model at
the end get a fraction (0–0.99) from the final regressor's |coefficient| or importance, which orders them. The
score is an order, not a size: a bar twice as long does not mean a band twice as important.

![Tree-based importance, c-org]({{ "/assets/media/ml_preprocesses_compiled/1-2-1-tree-c-org.png" | relative_url }})

The three selectors agree only partly on the bands. They measure different things, and correlated bands can
stand in for each other. Bands high in all three are the robust ones; which selection predicts best is decided
by modelling them ([Model the selections][pp2_model]).

### 1.2.2 RFE with cross-validation (RFECV)

**Process file**: `project_neospectra/process/ml_preprocess_compiled/band_selection_rfecv_c-org_plsr+randfor.json`.

With `"CV": 5`, RFE scores every number of bands by 5-fold cross-validated r² on the training samples and keeps
the best number. Each round removes bands, so the cross-validated r² is computed once per round. With `n_features` 0 that number is saved without asking:

```
        RFECV: <n> bands give the best cross-validated r2
    Retaining the top <n> bands for "c-org" / plsr / rfe
```

`"step": 0.1` removes a tenth of the bands (11 of 115) per round, which makes RFECV with `randfor` feasible; with
`step` 1 it refits the regressor once per band and fold. The name extension `#cv5` keeps the results apart from the fixed-size selections of 1.2.1.

Each selection gives two plots. The bar plot ranks the bands as in 1.2.1, but with `step` 0.1 there are only a
few rounds: all bands RFECV keeps survived every round and are ordered by the fraction from the final
regressor.

![RFECV ranking, c-org, plsr]({{ "/assets/media/ml_preprocesses_compiled/1-2-2-rfecv-c-org-plsr.png" | relative_url }})

The curve shows the cross-validated r² (mean ± standard deviation of the folds) against the number of bands, with
the chosen number marked (saved as `plot/selection_plot/<name>_rfecv.png`):

![RFECV curve, c-org, plsr]({{ "/assets/media/ml_preprocesses_compiled/1-2-2-rfecv-curve-c-org-plsr.png" | relative_url }})

![RFECV ranking, c-org, random forest]({{ "/assets/media/ml_preprocesses_compiled/1-2-2-rfecv-c-org-randfor.png" | relative_url }})

![RFECV curve, c-org, random forest]({{ "/assets/media/ml_preprocesses_compiled/1-2-2-rfecv-curve-c-org-randfor.png" | relative_url }})

Read the curve, not only the marked maximum. A flat curve means many band numbers predict about equally well,
and a broad band of ± one standard deviation means the folds disagree: RFECV then picks the maximum of a noisy
curve. A smaller number of bands on the flat part is just as good and gives a simpler model. Use the curve to
choose `n_features` for a fixed-size selection. The curve values are also in the band score JSON
(`"rfecv"`).

## Next step

[2 Wavelet coefficients][pp2_wavelet]: select wavelet coefficients instead of bands.

[pp2]: {{ site.baseurl }}/machine_learning/ml_preprocess2/
[pp2_ind]: {{ site.baseurl }}/machine_learning/ml_preprocess2/#indicators-and-covariates
[pp2_wavelet]: {{ site.baseurl }}/machine_learning/ml_preprocess2/wavelet/
[pp2_model]: {{ site.baseurl }}/machine_learning/ml_preprocess2/model/
[ml_model_avail]: {{ site.baseurl }}/machine_learning/ml_model/regressors/#available-regressors
[ml_model_regr]: {{ site.baseurl }}/machine_learning/ml_model/regressors/
