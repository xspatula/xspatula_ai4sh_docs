---
title: "ML Pre-processing 1: PCA"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Principal Component Analysis of spectra: what the components and scores are, the share of variance they explain, and comparing PCA of differently pre-processed inputs."
permalink: /machine_learning/ml_preprocess1/pca/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-10-31 08:00:00 +0100
---

Section 2.6 of the notebook `ml_preprocess_separated.ipynb`. See [ML pre-processing 1][pp1] for input
dataframes, naming and the companion JSON.

## What PCA does

Principal Component Analysis rotates the data into new axes, the **principal components**:

1. The data are centred: the mean of each band is subtracted (PCA does this itself, so a meancentred input
   gives the same result as a raw one).
2. The first component is the direction (a weighted combination of all bands, the *loadings*) along which
   the samples vary most. The second is the direction of largest remaining variance perpendicular to the
   first, and so on.
3. Each sample's spectrum is projected on the first `n_components` components. These coordinates, the
   *scores*, are the output columns `pc1 … pcN`.

Think of the samples as a cloud of points with one dimension per band. PCA finds the few directions in
which the cloud is stretched. Because neighbouring bands are strongly correlated, a handful of components
usually capture nearly all variation. The model gets, say, 5 uncorrelated inputs instead of 116 correlated
ones.

The components are chosen for **variance, not for relevance** to the target indicator. A component with
little variance may still be the one that predicts, say, nitrogen. That is why the explained variance alone
does not decide the number of components.

The process uses scikit-learn `PCA`. `n_components` is capped at the number of samples and bands. In a
model the components are refitted on the training samples ([Exploration versus modelling][explore]).

| Parameter (`spectra_decomposition`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `dataframe_array` | Compare mode: two or more input dataframes (see 2.6.2); overrides `dataframe` |
| `method` | `pca` (the only method) |
| `n_components` | Number of components, default 5 |
| `colormap` | Colour ramp of the plots |

## 2.6.1 Principal Component Analysis (PCA) Decomposition

**Process file**: `project_neospectra/process/ml_preprocess/spectra_decomposition.json`

```json
{
  "process": [
    {
      "process": "spectra_decomposition",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "method": "pca",
        "n_components": 5
      }
    }
  ]
}
```

The plot shows the input spectra and the scores of every sample on the 5 components (x-axis = component).
The title gives the share of the total spectral variance the components explain: the sum of the score
variances divided by the sum of the band variances.

![PCA of the raw spectra]({{ "/assets/media/ml_preprocesses_separate/2-6-1-pca.png" | relative_url }})

For raw spectra a few components explain nearly all the variance, and most of it is in pc1. Raw spectra
differ mainly in brightness (the scatter effect described in [scatter correction][pp1_scatter]), and one
component describes that. The scores of the higher components are close to zero.

```
    Accept and save pca5 result? [y/n]:
```

`y` saves `1350-2500_10.pca5`, `n` discards the result.

suffix: `.pca5` (`.pca` + the number of components)

## 2.6.2 Compare PCA decompositions

**Process file**: `project_neospectra/process/ml_preprocess/spectra_decomposition_compare_select.json`

```json
{
  "process": [
    {
      "process": "spectra_decomposition",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe_array": ["raw", "1350-2500_10.snvdt", "1350-2500_10.wc40"],
        "method": "pca",
        "n_components": 5
      }
    }
  ]
}
```

With two or more entries in `dataframe_array` the PCA of every input is computed and plotted, each labelled
with its explained variance. The inputs here are the raw spectra, the snv + detrend corrected spectra of
[2.2.2][pp1_scatter] and the 40 Ward clusters of [2.5.2][pp1_bands]:

![PCA of three inputs]({{ "/assets/media/ml_preprocesses_separate/2-6-2-compare-pca.png" | relative_url }})

- **raw** and **wc40**: nearly all variance in pc1. The Ward cluster means keep the brightness differences
  of the raw spectra, so the PCA looks the same with fewer input columns.
- **snvdt**: the variance is spread over several components. After scatter correction the brightness and
  baseline are gone, and the components describe differences in spectral shape: the absorption features
  that relate to soil composition. A lower explained variance is not a worse result: the variance left out
  is mostly noise, and the variance kept is more likely to relate to the indicators.

```
    Input dataframe to apply pca 5 to (0 = none, 1 = 1350-2500_10 (<n> % variance), 2 = 1350-2500_10.snvdt (<n> % variance), 3 = 1350-2500_10.wc40 (<n> % variance)):
```

The PCA of the chosen input is saved directly, with the suffix added to that input's name (e.g.
`1350-2500_10.snvdt.pca5`). `0` saves nothing (`No pca 5 applied.`).

suffix: `.pca5` (`.pca` + the number of components) added to the selected input dataframe

## Next step

[Chemometric chain][pp1_chain]: run a complete pre-processing pipeline in one go.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[explore]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#exploration-versus-modelling
[pp1_scatter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scatter/
[pp1_bands]: {{ site.baseurl }}/machine_learning/ml_preprocess1/bands/
[pp1_chain]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/
