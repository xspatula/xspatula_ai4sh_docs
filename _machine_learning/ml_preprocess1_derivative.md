---
title: "ML Pre-processing 1: Derivatives"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Converting spectra to finite-difference derivatives, comparing derivatives of differently pre-processed inputs, and the Savitzky-Golay derivative that smooths and differentiates in one step."
permalink: /machine_learning/ml_preprocess1/derivative/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-10-31 08:00:00 +0100
---

Section 2.3 of the notebook `ml_preprocess_separated.ipynb`. See [ML pre-processing 1][pp1] for input
dataframes, naming and the companion JSON.

## What a derivative does

A derivative replaces each spectrum by its rate of change along the wavelength axis:

- The **first derivative (D1)** removes constant baseline offsets. A spectrum shifted up by 0.1 has the same
  D1. Peaks and valleys turn into zero crossings, and the steepest flanks become the extremes.
- The **second derivative (D2)** also removes linear baseline trends (a sloping baseline has a constant D1
  and zero D2). D2 has its minima at the original absorption maxima and is narrower than the original
  feature. Overlapping absorption bands are thereby separated (sharpened).

The cost is noise. Differencing neighbouring bands amplifies band-to-band noise, and every order more so.
That is why derivatives are often combined with smoothing, most simply as a
[Savitzky-Golay derivative](#234-savitzky-golay-derivative).

## How the framework computes it

The process `spectra_derivative` takes the **difference between neighbouring bands**, repeated `derivative`
times:

- D1 at the midpoint of bands *j* and *j*+1 is *x*<sub>j+1</sub> − *x*<sub>j</sub>. The difference is per
  band step, not divided by the band width in nm: with the 10 nm selection the values are 10 times larger
  than a derivative per nm. For modelling this makes no difference.
- Each order loses one band: *n* bands give *n* − 1 D1 columns and *n* − 2 D2 columns.
- The new columns are named `d<midpoint wavelength>`, e.g. `d1355` between 1350 and 1360 nm. The companion
  JSON records them as derivative columns.
- With `"append": true` the derivatives are added to the original bands instead of replacing them (suffix
  `d1app`). This is not recommended for modelling, but can be used to explore whether bands or derivatives
  carry more information.

| Parameter (`spectra_derivative`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `dataframe_array` | Compare mode: two or more input dataframes (see 2.3.3); overrides `dataframe` |
| `derivative` | Order of the derivative, default 1 |
| `append` | `true` = append the derivatives to the original bands, default `false` |
| `colormap` | Colour ramp of the spectra plots |

## 2.3.1 Continuous derivatives

**Process file**: `project_neospectra/process/ml_preprocess/spectra_derivatives.json`

```json
{
  "process": [
    {
      "process": "spectra_derivative",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe_array": ["raw"],
        "derivative": 1
      }
    }
  ]
}
```

A `dataframe_array` with one entry is the same as `"dataframe": "raw"`. The first derivative of the raw
spectra:

![First derivative]({{ "/assets/media/ml_preprocesses_separate/2-3-1-first-derivative.png" | relative_url }})

The brightness differences of the raw spectra are gone: all derivatives lie around zero. What remains are
the absorption features, e.g. water near 1400 and 1900 nm (a negative then a positive swing) and clay
minerals (Al-OH) near 2200 nm.

```
    Accept and save d1 result? [y/n]:
```

`y` saves `1350-2500_10.d1`.

suffix: `.d1`, `.d2` … for the derivative set (`.d1app` etc. if the derivatives are appended)

## 2.3.2 Discrete derivatives

Planned: derivatives between specified pairs of bands (not neighbours), giving a dataframe with only those
derivatives. Not yet implemented; the cell is disabled in the notebook.

## 2.3.3 Compare derivatives

**Process file**: `project_neospectra/process/ml_preprocess/spectra_derivatives_compare_select.json`

```json
{
  "process": [
    {
      "process": "spectra_derivative",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe_array": ["raw", "1350-2500_10.snvdt", "1350-2500_10.lw21"],
        "derivative": 1
      }
    }
  ]
}
```

With two or more entries in `dataframe_array` the derivative is applied to every input and the results are
plotted together. The list should represent different levels of pre-processing; here the raw spectra, the
snv + detrend corrected spectra of [2.2.2][scatter] and the LOWESS-filtered spectra (filter preset `lowess`, 21
points, saved earlier with [2.1.1][filter]). All entries must exist in `project_root_fp`.

![First derivative of three inputs]({{ "/assets/media/ml_preprocesses_separate/2-3-3-compare-derivatives.png" | relative_url }})

- `snvdt → derivative 1` has the same features as the raw derivative but on the SNV scale. Because SNV
  removed the differences in brightness and slope between the spectra before differencing, the curves of
  the different samples lie closer together, and the differences that remain are in the absorption features.
- `lw21 → derivative 1` shows how strong a 21-point LOWESS smoothing is. Narrow features such as the water
  absorption near 1400 nm are largely smoothed away, and the 1900 nm feature becomes a broad swing.
  Smoothing before a derivative helps against noise, but this much smoothing removes information.

```
    Input dataframe to apply derivative 1 to (0 = none, 1 = 1350-2500_10, 2 = 1350-2500_10.snvdt, 3 = 1350-2500_10.lw21):
```

The derivative of the chosen input is saved directly, with the suffix added to that input's name, e.g.
`1350-2500_10.snvdt.d1`.

suffix: `.d1`, `.d2` … added to the selected input dataframe

## 2.3.4 Savitzky-Golay derivative

**Process file**: `project_neospectra/process/ml_preprocess/spectra_savitzky-golay_derivative.json`

```json
{
  "process": [
    {
      "process": "filter_spectra",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "filter_fp": "./project_neospectra/filter/savitzky-golay_d1.json"
      }
    }
  ]
}
```

with the filter file `project_neospectra/filter/savitzky-golay_d1.json`:

```json
{
  "savitzky-golay": {
    "window_length": 5,
    "polyorder": 2,
    "deriv": 1,
    "mode": "nearest"
  }
}
```

The Savitzky-Golay derivative is the [Savitzky-Golay filter][filter] with `deriv` 1 (or 2). In each window
of 5 bands a second-order polynomial is fitted by least squares, and the derivative of that polynomial at
the centre band is the output. Smoothing and derivative happen in one step. The derivative is taken of a
smooth local fit, not of two noisy neighbours, so noise is amplified much less than with the plain finite
difference of 2.3.1. Unlike the finite difference, no band is lost: the output has the same wavelength
columns as the input.

![Savitzky-Golay first derivative]({{ "/assets/media/ml_preprocesses_separate/2-3-4-savitzky-golay-derivative.png" | relative_url }})

Compared with 2.3.1 the curves are smoother and the extremes somewhat lower, because the 5-band window
averages the slope over 40 nm.

The Savitzky-Golay derivative is a common choice for soil NIR spectra pre-processing. In a pipeline it goes after scatter
correction and before variable scaling, and it replaces both the filter (2.1) and the derivative (2.3.1)
step:

```
raw → scatter correction → Savitzky-Golay derivative → meancentring → band reduction → PCA
```

The cell runs it on the raw spectra as an example. To follow the recommended order, set `"dataframe"` to a
scatter-corrected file (e.g. `1350-2500_10.snv`). [Chemometric chain][pp1_chain] 2.7.2 has it as one step of
a complete pipeline. Never apply a scatter correction (snv, msc) after a derivative.

```
    Accept and save sg5.d1 result? [y/n]:
```

`y` would save `1350-2500_10.sg5.d1`.

suffix: `.sg<window_length>.d1` (`.d2` for deriv 2), e.g. `.sg5.d1` (+ optional `#<name_extension>`)

## Next step

[Variable scaling][pp1_scaling]: centre and scale each band.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[filter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/filter/
[scatter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scatter/
[pp1_scaling]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scaling/
[pp1_chain]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/
