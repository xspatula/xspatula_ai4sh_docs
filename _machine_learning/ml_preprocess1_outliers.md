---
title: "ML Pre-processing 1: Indicator Outliers"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Detecting outlying indicator values with isolation forest, elliptic envelope, local outlier factor or one-class SVM, comparing the detectors, and setting the outliers to missing."
permalink: /machine_learning/ml_preprocess1/outliers/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-11-03 08:00:00 +0100
---

Section 1.1 of the notebook `ml_preprocess_separated.ipynb`. Indicators (the laboratory, home or field observed
values) have two pre-processes: outlier removal (this page) and the [indicator range][pp1_range] (1.2). Both run
on the indicator selection, not on the spectra, and both set single values to missing, never whole samples. See
[ML pre-processing 1][pp1] for input dataframes, naming and the companion JSON.

## How outliers are detected

The process `detect_outliers` works on **each indicator separately and in one dimension**: the detector
sees only the values of that indicator, not the spectra, the other indicators or the regressor. A value is
an outlier when it lies outside the bulk of the indicator's own distribution.

Four detectors from scikit-learn are available:

| `detector` | Method | How it decides |
|---|---|---|
| `iforest` | IsolationForest | Builds random trees that split the values at random points. Values that are isolated after few splits (short path) are outliers. Makes no assumption about the distribution. |
| `ee` | EllipticEnvelope | Fits a robust Gaussian (mean and variance from the most central values) and flags the values farthest from it. Assumes a roughly normal distribution. |
| `lof` | LocalOutlierFactor | Compares the density around each value with the density around its neighbours. Values in sparse regions are outliers, which also catches isolated values inside the range. |
| `1csvm` | OneClassSVM | Fits a boundary around the values with a kernel support vector machine. Values outside the boundary are outliers. |

`threshold` is the expected share of outliers, between 0 and 1 (exclusive). It is `contamination` for
iforest, ee and lof, and `nu` for 1csvm. With 0.1, about 10 % of the values of each indicator are flagged.

- Only the indicators in `indicator_array` are tested. Without it, every numeric column that is not
  metadata or a band is tested.
- An indicator with fewer than 5 values is skipped with a warning.
- If a detector fails on an indicator, isolation forest is used instead, marked `[fallback]` in the plot.

**Outliers are set to missing (NaN).** The rows stay, and so do the spectra and the other indicators of the
sample. Only the outlying value of the approved indicator is removed. A model of that indicator then drops
the sample, while models of other indicators still use it.

The threshold is a share, not a distance. With 0.1 the detector always flags about 10 % of the values,
whether or not they are real errors. For a skewed indicator like organic carbon, a high but correct value
from a peat sample is flagged as easily as a typing error. Look at the plot before you accept.
{: .notice--warning}

## 1.1.1 Compare and select outlier removal with fixed contamination

**Process file**: `project_neospectra/process/ml_preprocess/indicator_outlier_detection_select.json`

```json
{
  "process": [
    {
      "process": "detect_outliers",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/indicator/ai4sh",
        "indicator_array": ["n-tot", "c-org"],
        "detector": "select",
        "threshold": 0.1
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `project_root_fp` | The indicator selection folder |
| `dataframe` | Input dataframe, default `raw` (the selected indicator dataset) |
| `indicator_array` | Indicators to test; empty = all numeric indicators |
| `detector` | `iforest`, `ee`, `lof`, `1csvm`, or `select` to compare all four |
| `threshold` | Expected share of outliers (contamination, nu for 1csvm), default 0.1 |
| `name_extension` | Optional, see [Name extension][name_extension] |

With `"detector": "select"`, all four detectors run at the same threshold. The counts per indicator are
printed and plotted in a grid (row = indicator, column = detector, outliers in red):

```
    iforest n-tot: <n>  c-org: <n>
    ee     n-tot: <n>  c-org: <n>
    lof    n-tot: <n>  c-org: <n>
    1csvm  n-tot: <n>  c-org: <n>
```

![Outlier detectors compared]({{ "/assets/media/ml_preprocesses_separate/1-1-1-compare-outlier-detectors.png" | relative_url }})

For a skewed indicator the detectors usually agree on the long tail. Where they differ, look at which
values each one flags: ee assumes a normal distribution, lof also flags isolated values inside the range,
and 1csvm can flag values in the bulk of the distribution. Then the prompt asks for a detector:

```
    Outlier detector to apply (0 = none, 1 = iforest (<n>), 2 = ee (<n>), 3 = lof (<n>), 4 = 1csvm (<n>)):
```

The number in brackets is the total count over all indicators. `0` skips outlier detection. Any other
number continues as in 1.1.2 with the chosen detector: the threshold can still be changed, and you choose
which outliers to remove. 

suffix: `.olif`, `.olee`, `.ollof` or `.ol1csvm` for the chosen detector (+ optional `#<name_extension>`)

## 1.1.2 Apply predefined outlier removal with interactive contamination

**Process file**: `project_neospectra/process/ml_preprocess/indicator_outlier_detection.json`

```json
{
  "process": [
    {
      "process": "detect_outliers",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/indicator/ai4sh",
        "indicator_array": ["n-tot", "c-org"],
        "detector": "iforest",
        "threshold": 0.1
      }
    }
  ]
}
```

One detector, set in the process file. The outliers are printed and plotted, one panel per indicator
(the points are spread vertically only to make them visible):

![Isolation forest outliers]({{ "/assets/media/ml_preprocesses_separate/1-1-2-apply-outlier-detection.png" | relative_url }})

Then two prompts:

```
    New threshold? (0=skip outlier detection, Enter=accept 0.100):
```

- A number between 0 and 1 reruns the detector with that threshold and redraws the plot. Repeat until
  satisfied.
- Enter accepts the current threshold.
- `0` skips outlier detection; nothing is saved.

```
    Remove outliers? [all / none / individual]:
```

Write the answer in full:

- `all`: the outliers of every indicator are set to missing.
- `none`: nothing is removed or saved.
- `individual`: asks `Remove <n> outliers for "<indicator>"? [y/n]` for each indicator.

The result is saved as `<indicator dataset>.<suffix>`, e.g.
`cec-clay-ca2-ec-n-tot-c-org-ph-water_0-20.olif.parquet`, with the message
`Removed <n> outlier values across <k> indicators. Saved: …`. The companion JSON records the detector,
threshold and the indicators whose outliers were removed.

suffix: `.olif`, `.olee`, `.ollof` or `.ol1csvm` for the detector applied (+ optional `#<name_extension>`)

## In a model

Outlier removal is not refitted on the training samples in [ML modeling][ml_model]. The values were set to
missing using all samples, and the model prints a note that says so. That is acceptable for gross errors.
It is a reason to keep the threshold low rather than to use outlier removal to trim the distribution. To limit
the model to a population (e.g. agricultural soils with c-org up to 10 %), use the [indicator range][pp1_range]
instead: fixed limits, nothing learned from the samples.

Outlier detection and the indicator range are the two indicator steps: they only set indicator values to missing
and are never refitted in a model. An indicator file with either step can be [compiled][compile].

## Next step

[Indicator range][pp1_range]: set indicator values outside a [min, max] range to missing.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[name_extension]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#name-extension
[pp1_filter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/filter/
[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[pp1_range]: {{ site.baseurl }}/machine_learning/ml_preprocess1/range/
[compile]: {{ site.baseurl }}/machine_learning/parquet_files/#compile-the-project-dataset
