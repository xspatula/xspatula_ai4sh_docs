---
title: "ML Pre-processing 1: Decomposition"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Decomposition of spectra: Principal Component Analysis (components, scores, explained variance, comparing inputs) and the discrete wavelet transform (denoising, wavelet coefficients as features)."
permalink: /machine_learning/ml_preprocess1/pca/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-11-06 16:00:00 +0100
---

Section 2.6 of the notebook `ml_preprocess_separated.ipynb`: PCA (2.6.1, 2.6.2) and wavelets (2.6.3, 2.6.4).
See [ML pre-processing 1][pp1] for input dataframes, naming and the companion JSON.

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

## Wavelets

The discrete wavelet transform (DWT) splits a spectrum by scale. At each level the spectrum is passed through
a pair of filters: a smoothing filter gives the **approximation** (the broad shape, at half the number of
points) and a difference filter gives the **detail** coefficients (the sharp, local changes at that scale).
The next level splits the approximation again. After three levels a 116-band spectrum is described by about
20 approximation coefficients plus three sets of details, from fine (noise, sharp edges) to coarse.

Think of a photo saved at decreasing resolution: each step keeps a smaller image (the approximation) and
stores what was lost (the details). PCA finds patterns across the samples; a wavelet works on each spectrum
on its own, and each coefficient covers a wavelength region.

| Parameter (`spectra_wavelet`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder, or the compiled folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `mode` | `denoise` (default): same bands, less noise. `coefficients`: the coefficients replace the bands |
| `wavelet` | A discrete wavelet: `db4` (default, Daubechies 4), `db1`–`db38` (`db1` = Haar), `sym2`–`sym20`, `coif1`–`coif17`, … |
| `level` | Decomposition levels, default 3; capped to what the number of bands allows |
| `details` | `coefficients` only: detail levels kept next to the approximation, coarsest first; default 0 (approximation only) |
| `name_extension` | Optional, see [Name extension][name_extension] |
| `colormap`, `max_spectra` | Colour ramp and number of spectra in the plot |
| `show`, `save` | Show the plot, save it under `plot/` |

The process needs the PyWavelets package (`pip install PyWavelets`); without it the run stops with
`ERROR: wavelets need the PyWavelets package: pip install PyWavelets`. The wavelet works on each spectrum on
its own and learns nothing from the other samples, so in a model it is replayed unchanged.

## 2.6.3 Wavelet denoise

**Process file**: `project_neospectra/process/ml_preprocess/spectra_wavelet_denoise.json`

```json
{
  "process": [
    {
      "process": "spectra_wavelet",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "mode": "denoise",
        "wavelet": "db4",
        "level": 3,
        "show": true,
        "save": false
      }
    }
  ]
}
```

The detail coefficients are shrunk towards zero (soft thresholding) and the spectrum is rebuilt from the
approximation and the shrunk details. The threshold is the universal threshold of Donoho & Johnstone (1994),
σ·√(2 ln n), with the noise σ estimated per spectrum from the finest details and n the number of bands.
Details smaller than the threshold are taken as noise and removed; larger ones, real absorption features, are
kept but reduced by the threshold.

![Wavelet denoise of the raw spectra]({{ "/assets/media/ml_preprocesses_separate/2-6-3-wavelet-denoise.png" | relative_url }})

The output has the same bands as the input. Compared with a smoothing filter ([Filters][pp1_filter]), the
wavelet removes noise where the spectrum is flat and keeps sharp features where it changes. The 10 nm
NeoSpectra spectra carry little high-frequency noise, so the effect is small here.

```
    Accept and save wd result? [y/n]:
```

suffix: `.wd`

## 2.6.4 Wavelet coefficients

**Process file**: `project_neospectra/process/ml_preprocess/spectra_wavelet_coefficients.json`, the same
with `"mode": "coefficients"` and `"details": 1`.

The approximation coefficients at `level`, plus `details` levels of detail coefficients (coarsest first),
replace the bands as the columns `wt1 … wtn`. Like PCA this compresses the spectrum, but each coefficient still
belongs to a wavelength region.

![Wavelet coefficients of the raw spectra]({{ "/assets/media/ml_preprocesses_separate/2-6-4-wavelet-coefficients.png" | relative_url }})

With 116 bands, `db4` and level 3 there are 20 approximation coefficients (`wt1`–`wt20`), a smoothed and
shortened copy of the spectrum, and 20 detail coefficients of level 3 (`wt21`–`wt40`). On raw spectra the
details are close to zero: the shape is in the approximation. Every level halves the number of approximation
coefficients; `details` adds one block per level.

suffix: `.wt`

The coefficients are features, not wavelengths: a later band selection selects coefficients, and importance
plots show coefficient numbers ([ML pre-processing 2: Wavelet coefficients][pp2_wavelet]).

## Next step

[Chemometric chain][pp1_chain]: run a complete pre-processing pipeline in one go.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[explore]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#exploration-versus-modelling
[pp1_scatter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scatter/
[pp1_bands]: {{ site.baseurl }}/machine_learning/ml_preprocess1/bands/
[pp1_chain]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/
[pp1_filter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/filter/
[name_extension]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#name-extension
[pp2_wavelet]: {{ site.baseurl }}/machine_learning/ml_preprocess2/wavelet/
