---
title: "ML Pre-processing 1: Band Reduction"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Reducing the number of spectral bands without the target indicator: discarding low-variance bands with a variance threshold, and merging similar neighbouring bands with Ward clustering."
permalink: /machine_learning/ml_preprocess1/bands/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-10-31 08:00:00 +0100
---

Section 2.5 of the notebook `ml_preprocess_separated.ipynb`. See [ML pre-processing 1][pp1] for input
dataframes, naming and the companion JSON.

The 10 nm selection from 1350 to 2500 nm has 116 bands. Neighbouring bands are strongly correlated, and a
model with many correlated inputs and few samples easily overfits. Band reduction gives a more
parsimonious input. The two methods here do it without the target indicator or the regressor:

- **Variance threshold** (2.5.1) *discards* bands whose variance is low.
- **Ward clustering** (2.5.2) *merges* similar bands into clusters and keeps the cluster means.

Selection that uses the target indicator (univariate, permutation, recursive feature elimination and
tree-based) is part of [ML pre-processing 2][pp2]. Apply band reduction after derivatives: a derivative needs neighbouring bands,
and after Ward clustering the columns are clusters, not wavelengths.

## 2.5.1 Variance select spectral bands to keep/discard

**Process file**: `project_neospectra/process/ml_preprocess/spectra_variance_threshold_selection.json`

```json
{
  "process": [
    {
      "process": "select_variance_threshold",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw"
      }
    }
  ]
}
```

| Parameter (`select_variance_threshold`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `name_extension` | Optional, see [Name extension][name_extension] |

### What it computes

1. Every band is minmax scaled across the samples: (*x* − min) / (max − min), so every band ranges from
   0 to 1. Bands with high and low reflectance are then compared on the same scale.
2. The variance of every scaled band is computed.
3. Bands with a variance **at or below** the threshold are discarded; the others are kept unchanged (the
   scaling is only used to compute the variance).

After minmax scaling a low variance means that most samples are bunched in a narrow part of the band's range,
while a few samples set the extremes.

The scaling is always minmax. Standard scaling (mean 0, standard deviation 1) would give every band the
variance 1, and robust scaling (median and interquartile range) nearly constant variances, so neither could
separate bands.
{: .notice--info}

On raw spectra the spread of the scaled band variances is narrow, and the threshold separates bands only
weakly. The variance threshold is more informative after scatter correction.

### Interactive threshold

The first plot shows the scaled variance of every band, with the minimum as a dashed line, over the spectra.
The minimum, median and maximum are printed:

```
    Band variance (minmax scaled): min <min>   median <median>   max <max>
```

![Band variance, first plot]({{ "/assets/media/ml_preprocesses_separate/2-5-1-variance-threshold-initial.png" | relative_url }})

```
    Variance threshold (0 = skip) [min <min>, max <max>]:
```

- A number between minimum and maximum redraws the plot with the discarded bands shaded red and prints how
  many are kept (`Threshold <t>: <n> bands retained, <m> discarded`). Enter new numbers until you are
  satisfied; the y-axis stays fixed, so the plots compare.
- After a threshold has been given the prompt adds `Enter = accept <threshold>`. Enter saves the kept bands
  (no further y/n).
- `0`, or a value below the minimum, skips: nothing is saved.
- A value at or above the maximum would discard every band, and is refused.

![Band variance with a threshold]({{ "/assets/media/ml_preprocesses_separate/2-5-1-variance-threshold-set.png" | relative_url }})

Look at where the discarded bands lie before you accept. Bands next to the water absorption features
(1400 and 1900 nm) are not necessarily noise: water absorption relates to clay content and organic matter,
which are often the target indicators. Check with a model before discarding them.

suffix: `.vt` + threshold without the leading `0.`, e.g. `.vt03` for 0.03 or `.vt028` for 0.028
(+ optional `#<name_extension>`)

## 2.5.2 Agglomerate similar bands with ward clustering

**Process file**: `project_neospectra/process/ml_preprocess/spectra_ward_clustering.json`

```json
{
  "process": [
    {
      "process": "ward_clustering",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "connectivity": true,
        "ward_clustering_fp": "./project_neospectra/agglomeration/ward_clustering_neospectra.json"
      }
    }
  ]
}
```

| Parameter (`ward_clustering`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `ward_clustering_fp` | Ward clustering settings file (required) |
| `connectivity` | `true` (default) = only neighbouring bands are merged; `false` = any bands |
| `name_extension` | Optional, see [Name extension][name_extension] |
| `colormap` | Colour ramp of the spectra plots |

### What it computes

Ward clustering (scikit-learn `FeatureAgglomeration` with Ward linkage) treats each band as a point, defined
by its values in all samples. It starts with every band as its own cluster and repeatedly merges the two
clusters whose merge increases the within-cluster variance least, until `n_clusters` remain. Bands that
vary alike across the samples end up together. Each output column `wc1 … wcN` is the **mean of its member
bands**, ordered by wavelength (`wc1` at the shortest wavelengths). Ward linkage always uses Euclidean
distance.

- `connectivity: true` (default and recommended) only allows merging neighbouring bands. Every cluster is
  then a contiguous wavelength range, i.e. a band of variable width: wide where the spectrum is flat and
  featureless, narrow where it changes fast.
- `connectivity: false` merges similar bands wherever they are, and a cluster can contain bands from
  different parts of the spectrum. It compresses more, but the clusters no longer have a spectral meaning.

The companion JSON records the wavelengths of each cluster (`cluster_bands`), so the clustering can be
interpreted and replayed.

### Settings file and tuning

`project_neospectra/agglomeration/ward_clustering_neospectra.json`:

```json
{
  "ward_clustering": {
    "n_clusters": 0,
    "tune_ward_clustering": {
      "apply": true,
      "kfolds": 3,
      "clusters": [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
    }
  }
}
```

- With `tune_ward_clustering.apply: true`, every candidate in `clusters` is tested by k-fold
  cross-validation over the samples (`kfolds`, default 3). The clustering is fitted on the training folds.
  The test samples are then reduced to cluster means and expanded back to one value per band, and the mean
  squared difference to the original spectra is the reconstruction error (MSE).
- The suggestion is the **elbow** of the MSE curve. With the number of clusters and the MSE both scaled to
  0–1, it is the candidate furthest below the straight line from the first to the last candidate: the
  point where adding more clusters stops paying off. More clusters always reconstruct better, so a rule
  based on the lowest MSE would always pick the largest candidate.
  With fewer than 3 candidates the one with the lowest MSE is suggested.
- With `apply: false`, `n_clusters` is the suggestion (must then be > 0).

### Interactive number of clusters

```
    Tuning Ward clustering over 10 candidate cluster counts (connectivity=True) ...
    Tuned: n_clusters at the elbow of the MSE curve = <n>
```

The tuning plot shows the MSE of every candidate, with the elbow marked (`elbow n_clusters = <n>`):

![Ward tuning curve]({{ "/assets/media/ml_preprocesses_separate/2-5-2-ward-tuning.png" | relative_url }})

```
    Number of clusters to save (0 = no clustering, Enter = <n>) [1-<bands − 1>]:
```

Enter accepts the elbow, another number sets a different count, and `0` saves nothing. The result is saved
directly (no further y/n), e.g. as `1350-2500_10.wc40.parquet` for 40 clusters.

Two more plots follow. The cluster means against the input (x-axis = cluster number):

![Input spectra and Ward clusters]({{ "/assets/media/ml_preprocesses_separate/2-5-2-ward-clusters.png" | relative_url }})

And the clusters on the wavelength axis: every band replaced by its cluster mean, with the cluster borders as
thin vertical lines:

![Ward clusters on the wavelength axis]({{ "/assets/media/ml_preprocesses_separate/2-5-2-ward-wavelength.png" | relative_url }})

With `connectivity: true` the clusters are narrow where the spectra change fast, e.g. around the water
absorption features, and wide in smooth regions. The absorption features are preserved with far fewer
columns.

suffix: `.wc<n_clusters>`, e.g. `.wc40` (+ optional `#<name_extension>`)

## Next step

[PCA][pp1_pca]: decompose the spectra into principal components.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[name_extension]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#name-extension
[pp1_pca]: {{ site.baseurl }}/machine_learning/ml_preprocess1/pca/
[pp2]: {{ site.baseurl }}/machine_learning/ml_preprocess2/
