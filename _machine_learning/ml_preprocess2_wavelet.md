---
title: "ML Pre-processing 2: Wavelet coefficients"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Wavelet coefficients of compiled spectra in a chemometric chain, and band selection on the coefficients instead of the wavelengths."
permalink: /machine_learning/ml_preprocess2/wavelet/
author_profile: false
date: 2026-11-06 16:00:00 +0100
last_modified_at: 2026-11-06 16:00:00 +0100
---

Section 2 of the notebook `ml_preprocess_compiled.ipynb`. The wavelet transform itself is unsupervised and is
explained with the spectral pre-processing ([Wavelets][pp1_wavelet]): `"mode": "denoise"` keeps the bands
(suffix `.wd`), `"mode": "coefficients"` replaces them with the coefficients `wt1 … wtn` (suffix `.wt`). Here
the coefficients are computed on the compiled dataset and then selected like bands.

## 2.1.1 Wavelet coefficients in a chemometric chain

**Process file**: `project_neospectra/process/ml_preprocess_compiled/chemometric_chain_snv-d1-mc-wavelet-coefficients.json`

```json
{
  "process": [
    {
      "process": "chemometric_chain",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_olif_texture",
        "dataframe": "neospectra_10nm_olif_texture",
        "chemometrics_array": [
          "snv",
          "derivative",
          "meancentring",
          {"wavelet": {"mode": "coefficients", "wavelet": "db4", "level": 3, "details": 1}}
        ]
      }
    }
  ]
}
```

The chain starts from the compiled dataset and ends with the wavelet item
([chemometrics_array syntax][pp1_chain_syntax]). The 115 bands after the derivative become 40 coefficients:
20 approximation coefficients (`wt1`–`wt20`) and 20 detail coefficients of level 3 (`wt21`–`wt40`).

![Chain snv, derivative, meancentring and wavelet coefficients]({{ "/assets/media/ml_preprocesses_compiled/2-1-1-chain-wavelet-coefficients.png" | relative_url }})

The bottom panel marks the blocks: `A3` the approximation coefficients, `D3` the detail coefficients of level 3,
with a dashed line between them. After the derivative the spectrum has no baseline left, so the detail coefficients are no longer close to zero
as they are for raw spectra ([2.6.4][pp1_wt]): the sharp absorption features show in both blocks. The output is
`neospectra_10nm_olif_texture.snv.d1.mc.wt`.

In a model the wavelet is replayed unchanged: it works on each spectrum on its own and learns nothing from the
other samples.

## 2.2.1 Select wavelet coefficients

**Process file**: `project_neospectra/process/ml_preprocess_compiled/band_selection_rfe_c-org_plsr_wavelet-coefficients_k20.json`,
the RFE selection of [1.2.1][pp2_imp] with `"dataframe": "neospectra_10nm_olif_texture.snv.d1.mc.wt"`,
`"selector_array": ["rfe"]`, `"n_features": 20` and `"n_top_features_in_plot": 40`.

![RFE ranking of the wavelet coefficients, c-org, plsr]({{ "/assets/media/ml_preprocesses_compiled/2-2-1-rfe-wavelet-coefficients-c-org-plsr.png" | relative_url }})

The plot shows all 40 coefficients, ranked as in [1.2.1][pp2_imp]; the selection keeps the top 20:
`neospectra_10nm_olif_texture.snv.d1.mc.wt.c-org-plsr-rfe`. The axis shows coefficient numbers instead of
wavelengths: `wt1`–`wt20` are approximation, `wt21`–`wt40` detail coefficients. An approximation coefficient
covers a broad wavelength region, a detail coefficient a narrower one; neither maps to a single band. The
selection mixes both blocks: RFE keeps detail coefficients where they carry information the approximation
lacks.

## Next step

[3 Model the selections][pp2_model]: compare the selections in `regression_modeling`.

[pp1_wavelet]: {{ site.baseurl }}/machine_learning/ml_preprocess1/pca/#wavelets
[pp1_wt]: {{ site.baseurl }}/machine_learning/ml_preprocess1/pca/#264-wavelet-coefficients
[pp1_chain_syntax]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/#chemometrics_array-syntax
[pp2_imp]: {{ site.baseurl }}/machine_learning/ml_preprocess2/selection/#121-permutation-rfe-and-tree-based-importance-without-cross-validation
[pp2_model]: {{ site.baseurl }}/machine_learning/ml_preprocess2/model/
