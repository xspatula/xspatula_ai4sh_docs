---
title: "ML Pre-processing 1: Variable Scaling"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Per-band variable scaling of spectra: meancentring, autoscaling, Pareto scaling and Poisson scaling, what each computes and when to use it."
permalink: /machine_learning/ml_preprocess1/scaling/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-10-31 08:00:00 +0100
---

Section 2.4 of the notebook `ml_preprocess_separated.ipynb`. See [ML pre-processing 1][pp1] for input
dataframes, naming and the companion JSON.

## What variable scaling does

Variable scaling works **per band (column) across all samples**. For every band it subtracts a centre and
divides by a scale, both computed from the values of that band in all samples:

scaled value = (*x* − centre<sub>band</sub>) / scale<sub>band</sub>

This is the opposite direction of [scatter correction][pp1_scatter], which works per spectrum (see
[Per spectrum or per band][per_band]). Scatter correction makes spectra comparable with each other; variable
scaling makes bands comparable with each other.

| `method` | Centre | Scale | Effect | Suffix |
|---|---|---|---|---|
| `meancentring` | band mean | 1 | Every band gets mean 0. The data show variation around the average spectrum, not absolute values. Large-variance bands keep their weight. | `mc` |
| `autoscaling` | band mean | band standard deviation | Every band gets mean 0 and variance 1 (unit variance), so all bands weigh equally in a model. Also amplifies noisy bands with little real variation. | `as` |
| `paretoscaling` | band mean | √(band standard deviation) | Between meancentring and autoscaling: reduces the dominance of large-variance bands without blowing up noisy bands. | `ps` |
| `poissonscaling` | 0 (no centring) | √(band mean) | Divides each band by the square root of its mean (Poisson noise model, where variance grows with the signal). Values stay positive. | `poi` |

A band with zero scale (constant values) is divided by 1 instead.

Meancentring is the most common default. Many methods (PLS, PCA) assume centred data. PCA centres the data
itself, so meancentring right before PCA changes nothing. Autoscaling suits data where the bands have
different units, which is rarely the case for spectra.

The centre and scale depend on the samples. In a model they are recomputed from the training samples only
([Exploration versus modelling][explore]).

| Parameter (`spectra_scaling`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `method` | `meancentring` (default), `autoscaling`, `paretoscaling`, `poissonscaling`, or `select` to compare all four |
| `colormap` | Colour ramp of the spectra plots |

## 2.4.1 Select variable scaling

**Process file**: `project_neospectra/process/ml_preprocess/spectra_variable_scaling_select.json`

```json
{
  "process": [
    {
      "process": "spectra_scaling",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "method": "select"
      }
    }
  ]
}
```

All four scalings are applied and plotted under the raw spectra:

![Raw spectra and four variable scalings]({{ "/assets/media/ml_preprocesses_separate/2-4-1-select-variable-scaling.png" | relative_url }})

- **meancentring**: the average spectrum is subtracted. The spectra lie around 0, and the absorption
  features show only where samples differ from the average.
- **autoscaling**: the same pattern, but every band has unit variance. Bands with a large variance, e.g.
  at the water features, lose their prominence.
- **paretoscaling**: in between.
- **poissonscaling**: no centring. The spectra keep their shape and are only rescaled.

Applied to raw spectra, all four are dominated by the brightness differences between samples, i.e. by
scatter. That is why variable scaling comes after scatter correction (and derivative) in the
[recommended order][order].

```
    Variable scaling to apply (0 = none, 1 = meancentring, 2 = autoscaling, 3 = paretoscaling, 4 = poissonscaling):
```

The chosen scaling is saved directly (e.g. `1350-2500_10.mc`).

suffix: `.mc`, `.as`, `.ps` or `.poi` for the selected variable scaling

## 2.4.2 Apply predefined variable scaling

**Process file**: `project_neospectra/process/ml_preprocess/spectra_meancentring.json`

```json
{
  "process": [
    {
      "process": "spectra_scaling",
      "parameters": {
        "method": "meancentring",
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw"
      }
    }
  ]
}
```

One scaling, set in `method`. Meancentring of the raw spectra:

![Meancentring]({{ "/assets/media/ml_preprocesses_separate/2-4-2-apply-meancentring.png" | relative_url }})

```
    Accept and save mc result? [y/n]:
```

`y` saves `1350-2500_10.mc`. In a pipeline set
`dataframe` to the output of the step before, e.g. `1350-2500_10.snv.sg5.d1` → `1350-2500_10.snv.sg5.d1.mc`.

suffix: `.mc` (meancentring), `.as` (autoscaling), `.ps` (paretoscaling) or `.poi` (poissonscaling)

## Next step

[Band reduction][pp1_bands]: discard low-variance bands or merge similar bands.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[per_band]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#per-spectrum-or-per-band
[explore]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#exploration-versus-modelling
[order]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#recommended-order
[pp1_scatter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scatter/
[pp1_bands]: {{ site.baseurl }}/machine_learning/ml_preprocess1/bands/
