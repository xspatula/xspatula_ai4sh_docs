---
title: "ML Pre-processing 1: Scatter Correction"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Per-spectrum scatter correction with L1, L2 or max normalisation, Standard Normal Variate (SNV), Multiplicative Scatter Correction (MSC) or detrending, alone or in sequence."
permalink: /machine_learning/ml_preprocess1/scatter/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-10-31 08:00:00 +0100
---

Section 2.2 of the notebook `ml_preprocess_separated.ipynb`. See [ML pre-processing 1][pp1] for input
dataframes, naming and the companion JSON.

## Why scatter correction

Diffuse reflectance from a soil sample depends on the chemistry of the sample, but also on how light
scatters in it. Particle size, packing density, surface roughness and path length shift a whole spectrum up
or down (an **additive** effect, offset) and stretch or compress it (a **multiplicative** effect, slope). Two
subsamples of the same soil, one ground finer, give spectra of different brightness but with the same
absorption features. A scatter correction removes these effects so that the remaining differences between
spectra reflect composition.

Think of photographs of the same page taken in different light. Scatter correction adjusts brightness and
contrast so that only the text differs.

All six methods work **per spectrum**: each row of the data table is corrected using only its own values
(MSC also uses the mean spectrum as reference). This is the opposite of
[variable scaling][pp1_scaling], which works per band across all samples (see
[Per spectrum or per band][per_band]).

## The six methods

For a spectrum *x* with bands *x*<sub>1</sub> … *x*<sub>n</sub>:

| `scaler` | Name | Computation | Removes | Suffix |
|---|---|---|---|---|
| `l1` | L1 norm | *x* / Σ\|*x*<sub>j</sub>\| — divide by the sum of absolute values | multiplicative (total intensity) | `l1` |
| `l2` | L2 norm | *x* / √(Σ*x*<sub>j</sub>²) — divide by the Euclidean length | multiplicative (spectral energy) | `l2` |
| `max` | Max norm | *x* / max\|*x*<sub>j</sub>\| — divide by the largest value | multiplicative (peak normalisation) | `max` |
| `snv` | Standard Normal Variate | (*x* − mean(*x*)) / std(*x*) — centre on the spectrum's own mean, divide by its own standard deviation | additive and multiplicative | `snv` |
| `msc` | Multiplicative Scatter Correction | Regress the spectrum on a reference spectrum, *x* ≈ *a* + *b*·*ref*; correct as (*x* − *a*) / *b* | additive and multiplicative | `msc` |
| `detrend` | Detrending | Fit a 2nd-order polynomial to the spectrum against band position and subtract it | curved (quadratic) baseline | `dt` |

- **l1, l2, max** (scikit-learn `normalize`) only rescale. A spectrum keeps its shape and offset but gets a
  common total, length or maximum.
- **snv** gives every spectrum mean 0 and standard deviation 1. It is the most used scatter correction for
  soil NIR and needs no reference.
- **msc** first subtracts each spectrum's own mean, then uses the mean spectrum of all samples as the
  reference *ref*. Each spectrum is fitted to it by a straight line (offset *a*, slope *b*), and that line is
  removed. SNV and MSC usually give very similar results. MSC depends on the samples through the reference;
  in a model the reference is recomputed from the training samples ([Exploration versus modelling][explore]).
- **detrend** removes the curvature of the baseline that SNV leaves in diffuse reflectance spectra, mainly
  caused by particle size. What remains are the absorption features around a flat baseline. Like SNV it
  learns nothing from other samples, so a model applies it as it is. It is normally used after SNV
  (Barnes et al. 1989).

## Corrections in sequence

A list applies the corrections in sequence, each on the output of the one before. The abbreviations are
joined without a dot: `"scaler": ["snv", "detrend"]` → suffix `snvdt`. The order matters: `snv` then
`detrend` is not the same as `detrend` then `snv`.

A sequence is only useful if a later step removes something the earlier one leaves. SNV and MSC already
remove the offset and scale of each spectrum, so an offset or scale step before them is wasted:

| Sequence | Result |
|---|---|
| `snv` + `snv` | identical to `snv` (after SNV the mean is 0 and the standard deviation 1, a second SNV changes nothing) |
| `l1`, `l2` or `max` + `snv` | identical to `snv` (the common factor cancels in SNV) |
| `msc` + `snv` | identical to `snv` (MSC is an offset and a scale per spectrum, SNV removes both) |
| `l1`, `l2` or `max` + `msc` | almost identical to `msc` (only the MSC reference spectrum changes slightly) |
| `snv` + `msc` | differs from `snv`, but SNV and MSC are near-equivalent corrections, so little is gained |
| `snv` + `detrend` | the classic two-step correction: SNV removes offset and scale, detrend the curved baseline |

Use `["snv", "detrend"]` when a single correction is not enough. For a step that has no effect, the process
prints a warning, e.g.:

```
    Warning: l2 before snv has no effect: snv removes the offset and scale of each spectrum, the result equals the sequence without l2.
```

and for `l1`, `l2` or `max` before `msc`: `… has almost no effect: msc removes the scale of each spectrum,
only the msc reference spectrum changes slightly.` The step still runs and stays in the suffix; drop it from
`scaler`.

| Parameter (`spectra_scatter_correction`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `scaler` | One method, a list applied in sequence, or `["select"]` to compare all six; default `["snv"]` |
| `colormap` | Colour ramp of the spectra plots |

Apply scatter correction to complete, unscaled spectra: before derivatives, variable scaling and band
reduction. After a derivative the baseline and slope that SNV, MSC and detrend model are gone (see
[Recommended order][order]).
{: .notice--info}

## 2.2.1 Select scatter correction

**Process file**: `project_neospectra/process/ml_preprocess/spectra_scatter_correction_select.json`

```json
{
  "process": [
    {
      "process": "spectra_scatter_correction",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "scaler": ["select"]
      }
    }
  ]
}
```

All six single corrections are applied and plotted under the input spectra:

![Raw spectra and six scatter corrections]({{ "/assets/media/ml_preprocesses_separate/2-2-1-select-scatter-correction.png" | relative_url }})

- **l1, l2, max** remove the brightness differences between the spectra. What remains are differences in
  shape: slope, curvature and the depth of the absorption features.
- **snv** and **msc** give nearly the same pattern, snv on a standardised scale (around 0), msc in
  reflectance units around 0.
- **detrend** flattens the baseline of every spectrum. The absorption features stand out around a common
  zero line.

```
    Scatter correction to apply (0 = none, 1 = l1, 2 = l2, 3 = max, 4 = snv, 5 = msc, 6 = detrend):
```

The chosen correction is saved directly (e.g. `1350-2500_10.snv`).

suffix: `.l1`, `.l2`, `.max`, `.snv`, `.msc` or `.dt` for the selected scatter correction

## 2.2.2 Apply predefined scatter correction

**Process file**: `project_neospectra/process/ml_preprocess/spectra_snv+detrend_scatter_correction.json`

```json
{
  "process": [
    {
      "process": "spectra_scatter_correction",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "scaler": ["snv", "detrend"]
      }
    }
  ]
}
```

One correction, or a sequence, set in `scaler`. Input and output are plotted:

![SNV + detrend]({{ "/assets/media/ml_preprocesses_separate/2-2-2-apply-snv-detrend.png" | relative_url }})

After SNV and detrend the spectra share a flat baseline, and the absorption features (water, clay minerals,
organic matter) are what differs between them.

```
    Accept and save snvdt result? [y/n]:
```

Answering `y` saves a new dataframe:

```
    Saved: …/project_neospectra/data/spectra/neospectra/ds2/ai4sh/1350-2500_10.snvdt.parquet
```

For a single correction write e.g. `"scaler": ["snv"]` (suffix `.snv`).

suffix: the scatter corrections joined, e.g. `.snvdt` for snv + detrend

## Next step

[Derivatives][pp1_derivative]: convert spectra to first or second derivatives.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[per_band]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#per-spectrum-or-per-band
[explore]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#exploration-versus-modelling
[order]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#recommended-order
[pp1_scaling]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scaling/
[pp1_derivative]: {{ site.baseurl }}/machine_learning/ml_preprocess1/derivative/
