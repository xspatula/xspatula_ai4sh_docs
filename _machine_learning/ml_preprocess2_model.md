---
title: "ML Pre-processing 2: Model the selections"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Comparing band selections and wavelet coefficients with regression_modeling: one regression block per dataframe, train/test, and how to read the result."
permalink: /machine_learning/ml_preprocess2/model/
author_profile: false
date: 2026-11-06 16:00:00 +0100
last_modified_at: 2026-11-06 16:00:00 +0100
---

Section 3 of the notebook `ml_preprocess_compiled.ipynb`. A selection is only as good as the model it gives.
This cell models c-org on the full pre-processed spectra and on every selection of sections 1 and 2, with the
same regressors and the same test samples.

## 3.1.1 Model the selections

**Process file**: `project_neospectra/process/ml_preprocess_compiled/regression_modeling_c-org_band-selections_tt.json`

The file holds one `regression_modeling` block per dataframe. The first block:

```json
{
  "process": "regression_modeling",
  "overwrite": true,
  "parameters": {
    "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_olif_texture",
    "dataframe": "neospectra_10nm_olif_texture.snv.d1.mc",
    "indicator_array": ["c-org"],
    "regressor_array": ["plsr", "randfor", "svr"],
    "model_parameters_fp": "./project_neospectra/regressors/regression_model_parameters_tuned.json",
    "traintest": true,
    "test_size": 0.3,
    "kfold": false,
    "show_observed_predicted": true,
    "plot_layout": "default"
  }
}
```

The other blocks differ only in `dataframe`:

| `dataframe` | Input |
|---|---|
| `neospectra_10nm_olif_texture.snv.d1.mc` | All 115 bands, the reference |
| `….snv.d1.mc.uv-c-org` | Univariate, 30 bands ([1.1.1][pp2_uv]) |
| `….snv.d1.mc.c-org-plsr-pms` | Permutation, 30 bands ([1.2.1][pp2_imp]) |
| `….snv.d1.mc.c-org-plsr-rfe` | RFE, 30 bands |
| `….snv.d1.mc.c-org-tree` | Tree-based, 30 bands |
| `….snv.d1.mc.wt` | All 40 wavelet coefficients ([2.1.1][pp2_wt]) |
| `….snv.d1.mc.wt.c-org-plsr-rfe` | RFE, 20 wavelet coefficients ([2.2.1][pp2_wt_sel]) |

Run the selection cells first. A missing dataframe is reported and its block skipped; the other blocks run.

Each block gives one row of observed-versus-predicted plots, one per regressor, titled with the dataframe. The
box holds the test metrics and the number of training and test samples; all rows have the same test samples.
The metrics are also printed per block.

![All 115 bands (reference), c-org, train/test]({{ "/assets/media/ml_preprocesses_compiled/3-1-1-tt-all-bands.png" | relative_url }})

![Univariate selection, c-org, train/test]({{ "/assets/media/ml_preprocesses_compiled/3-1-1-tt-uv-c-org.png" | relative_url }})

![Permutation selection, c-org, train/test]({{ "/assets/media/ml_preprocesses_compiled/3-1-1-tt-c-org-plsr-pms.png" | relative_url }})

![RFE selection, c-org, train/test]({{ "/assets/media/ml_preprocesses_compiled/3-1-1-tt-c-org-plsr-rfe.png" | relative_url }})

![Tree-based selection, c-org, train/test]({{ "/assets/media/ml_preprocesses_compiled/3-1-1-tt-c-org-tree.png" | relative_url }})

![All 40 wavelet coefficients, c-org, train/test]({{ "/assets/media/ml_preprocesses_compiled/3-1-1-tt-wt.png" | relative_url }})

![RFE selection of 20 wavelet coefficients, c-org, train/test]({{ "/assets/media/ml_preprocesses_compiled/3-1-1-tt-wt-c-org-plsr-rfe.png" | relative_url }})


## Reading the result

- **Use train/test.** All selections except the univariate one are replayed as fixed bands, chosen from the
  training samples. Train/test metrics are then clean; k-fold metrics would include the selection
  ([How a model uses a selection][pp2_replay]).
- **Compare with the reference.** A selection is worth keeping when it predicts as well as, or better than, all
  bands with the same regressor. Fewer inputs with the same accuracy is a gain: faster models and a clearer
  link to absorption features.
- **One split, one dataset.** About 40 test samples give a noisy estimate, and another split or another dataset
  can rank the selectors differently. The ranking here is an example, not a general result. Treat small
  differences as noise.
- **Selector and regressor go together.** A selection made with `plsr` suits a `plsr` model best; the same
  bands may help `randfor` less.

## Next step

[ML modeling][ml_model]: model the chosen dataframe with more indicators, regressors and k-fold.

[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[pp2_replay]: {{ site.baseurl }}/machine_learning/ml_preprocess2/#how-a-model-uses-a-selection
[pp2_uv]: {{ site.baseurl }}/machine_learning/ml_preprocess2/selection/#111-univariate-selection-with-selectkbest
[pp2_imp]: {{ site.baseurl }}/machine_learning/ml_preprocess2/selection/#121-permutation-rfe-and-tree-based-importance-without-cross-validation
[pp2_wt]: {{ site.baseurl }}/machine_learning/ml_preprocess2/wavelet/#211-wavelet-coefficients-in-a-chemometric-chain
[pp2_wt_sel]: {{ site.baseurl }}/machine_learning/ml_preprocess2/wavelet/#221-select-wavelet-coefficients
