---
title: "ML Pre-processing 1: Filters"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Smoothing spectra with moving average, Gaussian, Savitzky-Golay, LOWESS or LOESS filters, comparing filter settings, and region-wise multi-filters that also trim the spectral range."
permalink: /machine_learning/ml_preprocess1/filter/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-11-11 08:00:00 +0100
---

Section 2.1 of the notebook `ml_preprocess_separated.ipynb`. A filter smooths each spectrum along the
wavelength axis to reduce small-scale noise, e.g. sensor noise. The bands stay the same: a filtered
spectrum has the same columns as its input. Trimming the spectral range is normally done when selecting the
spectra (`select_spectra_only.ipynb`), but the [multi-filter](#216-multi-filter) can also trim. See
[ML pre-processing 1][pp1] for input dataframes, naming and the companion JSON.

## The five filters

All five filters slide a window along each spectrum and replace every band by a weighted local estimate. Window
sizes are counted in **bands, not nm**: with the 10 nm NeoSpectra selection a 5-band window spans 40 nm
(centre ± 2 bands).

| Filter | Settings key | Computation | Suffix |
|---|---|---|---|
| moving-average | `moving_average`: `kernel`, `mode` | Convolution with the kernel, normalised to sum 1. `[1,1,1,1,1]` is a plain 5-band mean; `[1,2,3,2,1]` weights the centre more. | `ma<kernel length>` |
| gauss | `gauss`: `sigma`, `mode` | Convolution with a Gaussian kernel; `sigma` is the standard deviation in bands. | `gf<sigma>` |
| savitzky-golay | `savitzky-golay`: `window_length`, `polyorder`, `deriv`, `mode` | Fits a polynomial of degree `polyorder` by least squares to the `window_length` bands around each band and takes its value at the centre. `deriv` 1 or 2 takes the derivative of the polynomial instead ([Savitzky-Golay derivative][sg_derivative]). | `sg<window_length>` (`sg<n>.d1` with deriv 1) |
| lowess | `lowess`: `n_points` | Local linear regression over the `n_points` nearest bands, weighted with a tricube function (weight falls from 1 at the centre to 0 at the farthest point). | `lw<n_points>` |
| loess | `loess`: `n_points` | As lowess, but a local quadratic (degree 2) regression. `n_points` must be at least 4. | `lo<n_points>` |

- Moving average and gauss flatten peaks: they average the peak with its shoulders. The wider the window,
  the more a narrow absorption feature is lowered and broadened.
- Savitzky-Golay preserves peak height and position better, because a polynomial can follow a curved peak.
  It is the standard smoothing filter for NIR spectra.
- LOWESS fits a straight line in each window. It is robust but with many points it smooths strongly:
  21 points span 200 nm at 10 nm bands.
- LOESS fits a second-order polynomial in each window, like Savitzky-Golay. Savitzky-Golay weights all
  bands of a symmetric window equally; LOESS weights the `n_points` nearest bands with the tricube function.
  LOESS reproduces a quadratic exactly, so it keeps curved peaks that LOWESS lowers.
- The tricube weight is 0 at the farthest point. With `n_points` 7 only 5 bands (centre ± 2) get a weight
  above 0 in the interior of the spectrum, for LOWESS and LOESS alike. A small `n_points` therefore smooths
  little: LOESS with `n_points` 4 or 5 leaves the interior bands unchanged (3 weighted bands, 3 polynomial
  coefficients), and an even `n_points` gives the same interior result as the next odd one.
- `mode` sets how the window is filled at the ends of the spectrum. The default `"nearest"` repeats the
  first and last band. LOWESS and LOESS have no `mode`: near the ends they use the `n_points` nearest bands,
  so the window becomes one-sided.

### Filter settings

A filter is given as either:

- a **parameter set name** from the database (family `filter`, see [ML settings][ml_settings]). The five
  presets `moving-average`, `gauss`, `savitzky-golay`, `lowess` and `loess` hold the defaults below.
- the **path to a filter JSON file**, relative to the project root (`.json` optional). Custom filters for
  this project are in `xspatula_ai4sh/ai4sh/project_neospectra/filter/`.

| Preset | Settings | Suffix |
|---|---|---|
| `moving-average` | `{"moving_average": {"kernel": [1, 1, 1, 1, 1], "mode": "nearest"}}` | `ma5` |
| `gauss` | `{"gauss": {"sigma": 2, "mode": "nearest"}}` | `gf2` |
| `savitzky-golay` | `{"savitzky-golay": {"window_length": 5, "polyorder": 2, "deriv": 0, "mode": "nearest"}}` | `sg5` |
| `lowess` | `{"lowess": {"n_points": 21}}` | `lw21` |
| `loess` | `{"loess": {"n_points": 7}}` | `lo7` |

The `loess` preset was added on 2026-11-11. A database set up before that has no `loess` record until the
setting records are reloaded (re-run [setup_db][setup_db_ml_setting]); until then give the default file
instead: `filter_fp: "ai4sh/default/filter/loess_default.json"`.

A custom filter file has the same form, e.g. `project_neospectra/filter/moving-average_11.json`:

```json
{
  "moving_average": {
    "mode": "nearest",
    "kernel": [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
  }
}
```

The companion JSON records the filter settings themselves (key `filter`), not only the name or path. The
`.previous_dataframe` provenance records every filter file or parameter set that was read, with its sha256.

| Parameter (`filter_spectra`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `filter_fp` | A filter parameter set name or JSON path, or `select` to compare the filters in `filter_array` |
| `filter_array` | Only with `select`: the filters to compare (names or paths); default the five presets |
| `name_extension` | Optional, see [Name extension][name_extension] |
| `colormap` | Colour ramp of the spectra plots, default the `spectra_plot` set (`jet`) |

## 2.1.1 Select filter smoothing

**Process file**: `project_neospectra/process/ml_preprocess/spectra_filter_select.json`

```json
{
  "process": [
    {
      "process": "filter_spectra",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "filter_fp": "select",
        "filter_array": ["./project_neospectra/filter/moving-average_5.json", "gauss",
                         "./project_neospectra/filter/savitzky-golay", "lowess"]
      }
    }
  ]
}
```

The example mixes files (`moving-average_5.json`, `savitzky-golay`) with preset names (`gauss`,
`lowess`). Without `filter_array` the five presets are compared. Every filter is applied to the input and
plotted under the raw spectra:

![Four filters compared]({{ "/assets/media/ml_preprocesses_separate/2-1-1-select-filter.png" | relative_url }})

Compare how much each filter lowers and broadens the absorption features, e.g. the water features at 1400
and 1900 nm. A filter that removes noise but leaves the features intact is the one to choose; one that
flattens the features removes information. Then:

```
    Filter to apply (0 = none, 1 = moving-average_5, 2 = gauss, 3 = savitzky-golay, 4 = lowess):
```

Without `filter_array` the labels are the five preset names:

```
    Filter to apply (0 = none, 1 = moving-average, 2 = gauss, 3 = savitzky-golay, 4 = lowess, 5 = loess):
```

The chosen filter is saved directly, e.g. `1350-2500_10.ma5`. `0` saves nothing.

suffix: `.ma<kernel length>`, `.gf<sigma>`, `.sg<window_length>`, `.lw<n_points>` or `.lo<n_points>` (+ optional `#<name_extension>`)

## 2.1.2 Compare (and select) filters with custom parameters

**Process file**: `project_neospectra/process/ml_preprocess/spectra_filter_moving-average_compare.json`

```json
{
  "process": [
    {
      "process": "filter_spectra",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "filter_fp": "select",
        "filter_array": ["./project_neospectra/filter/moving-average_5.json",
                         "./project_neospectra/filter/moving-average_11.json",
                         "./project_neospectra/filter/moving-average_15.json"]
      }
    }
  ]
}
```

The same process as 2.1.1, but comparing one filter at different strengths: moving averages over 5, 11 and
15 bands (40, 100 and 140 nm).

![Moving averages of 5, 11 and 15 bands]({{ "/assets/media/ml_preprocesses_separate/2-1-2-compare-custom-filters.png" | relative_url }})

The wider the window, the more the narrow absorption features are broadened and lowered. Choose the
narrowest window that removes the noise you see.

If two filters of the same kind and length differ only in their weights, they get the same suffix. Use
`name_extension` to tell them apart, e.g. `.ma5#12321` for the kernel `[1, 2, 3, 2, 1]` (see 2.1.5).

## 2.1.3 Apply predefined filter smoothing

**Process file**: `project_neospectra/process/ml_preprocess/spectra_savitzky-golay_filter.json`

```json
{
  "process": [
    {
      "process": "filter_spectra",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "filter_fp": "./project_neospectra/filter/savitzky-golay.json"
      }
    }
  ]
}
```

`savitzky-golay.json` replicates the database preset (window 5, polyorder 2, deriv 0). Note that `deriv` is
0, i.e. smoothing; the derivative variant is a separate file (`savitzky-golay_d1.json`, see
[2.3.4][sg_derivative]).

![Savitzky-Golay smoothing]({{ "/assets/media/ml_preprocesses_separate/2-1-3-apply-savitzky-golay-filter.png" | relative_url }})

```
    Accept and save sg5 result? [y/n]:
```

`y` saves `1350-2500_10.sg5`; `n` prints `Result discarded.`

suffix: `.sg5` here; in general `.ma<n>`, `.gf<sigma>`, `.sg<n>`, `.lw<n>` or `.lo<n>` (+ optional `#<name_extension>`)

## 2.1.4 LOWESS filter length and comparison with Savitzky-Golay and LOESS

**Process file**: `project_neospectra/process/ml_preprocess/spectra_filter_lowess_vs_sg.json`

```json
{
  "process": [
    {
      "process": "filter_spectra",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "filter_fp": "select",
        "filter_array": ["lowess",
                         "./project_neospectra/filter/lowess_7.json",
                         "./project_neospectra/filter/savitzky-golay_7.json",
                         "loess"]
      }
    }
  ]
}
```

with the custom filter files `project_neospectra/filter/lowess_7.json`:

```json
{
  "lowess": {
    "n_points": 7
  }
}
```

and `project_neospectra/filter/savitzky-golay_7.json`:

```json
{
  "savitzky-golay": {
    "window_length": 7,
    "polyorder": 2,
    "deriv": 0,
    "mode": "nearest"
  }
}
```

The same select process as 2.1.1, set up for two comparisons:

- **Filter length**: the LOWESS preset (21 points, 200 nm) against LOWESS over 7 points (60 nm).
- **Filter type at the same length**: LOWESS, Savitzky-Golay and LOESS, all over 7 bands. LOWESS fits a
  weighted straight line in each window, Savitzky-Golay an unweighted second-order polynomial and LOESS
  (the `loess` preset) a tricube-weighted second-order polynomial.

![LOWESS 21 and 7 points, Savitzky-Golay 7 and LOESS 7 compared]({{ "/assets/media/ml_preprocesses_separate/2-1-4-lowess-loess-vs-savitzky-golay.png" | relative_url }})

- With 21 points LOWESS flattens and broadens the absorption features. With 7 points the features are
  kept, and the spectra are only smoothed.
- At the same window length Savitzky-Golay follows the features more closely than LOWESS. A straight line
  cannot follow a curved absorption peak, so LOWESS lowers the peak; a polynomial can follow it.
- LOESS 7 and Savitzky-Golay 7 are hard to tell apart: both are quadratics, so both keep the peak height
  that LOWESS 7 loses. The tricube weights put more weight on the centre band, and only 5 of the 7 bands
  get a weight above 0, so `lo7` smooths about as much as `sg5`.

```
    Filter to apply (0 = none, 1 = lowess, 2 = lowess_7, 3 = savitzky-golay_7, 4 = loess):
```

The labels are the preset names and the file names. The chosen filter is plotted against the input and
saved directly; here `4` saves `1350-2500_10.lo7`. `0` saves nothing.

![LOESS 7 applied]({{ "/assets/media/ml_preprocesses_separate/2-1-4-apply-loess-filter.png" | relative_url }})

suffix: `.lw<n_points>`, `.sg<window_length>` or `.lo<n_points>`, e.g. `.lw21`, `.lw7`, `.sg7`, `.lo7` (+ optional `#<name_extension>`)

## 2.1.5 Moving average with name extension

**Process file**: `project_neospectra/process/ml_preprocess/spectra_filter_moving-average_12321.json`

```json
{
  "process": [
    {
      "process": "filter_spectra",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "filter_fp": "./project_neospectra/filter/moving-average_5_12321.json",
        "name_extension": "12321"
      }
    }
  ]
}
```

with the filter file `project_neospectra/filter/moving-average_5_12321.json`:

```json
{
  "moving_average": {
    "mode": "nearest",
    "kernel": [1, 2, 3, 2, 1]
  }
}
```

A 5-band moving average with the weights `[1, 2, 3, 2, 1]` (normalised to sum 1), which weights the centre
band most. Its suffix is `ma5`, the same as for the plain moving average `[1, 1, 1, 1, 1]` of 2.1.1, since
the suffix only records the kernel length. Without a name extension the two outputs would get the same
name, and the second would not be computed ([Overwrite][overwrite]). `"name_extension": "12321"` adds
`#12321` to the suffix and keeps them apart. The extension is 1–6 letters, digits or `-`
([Name extension][name_extension]).

![Weighted moving average]({{ "/assets/media/ml_preprocesses_separate/2-1-5-moving-average-name-extension.png" | relative_url }})

```
    Accept and save ma5#12321 result? [y/n]:
```

`y` saves:

```
    Saved: …/project_neospectra/data/spectra/neospectra/ds2/ai4sh/1350-2500_10.ma5#12321.parquet
```

suffix: `.ma5#12321` (`.ma<kernel length>` + `#<name_extension>`)

## 2.1.6 Multi-filter

**Process file**: `project_neospectra/process/ml_preprocess/spectra_multi_filter_example.json`

```json
{
  "process": [
    {
      "process": "multi_filter_spectra",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "multi_filter_fp": "./project_neospectra/filter/multi_filter_neospectra.json"
      }
    }
  ]
}
```

A multi-filter applies different filters to different wavelength regions. Use it for a sensor that is
noisier in some regions (often the ends of the range), or to emulate a coarser sensor from a finer one. The
regions are defined in `multi_filter_fp`, a JSON file with parallel lists, one position per region:

```json
{
  "multi_filter": {
    "mode": "nearest",
    "begin_wavelength": [1350, 1450, 2400],
    "end_wavelength":   [1450, 2400, 2510],
    "moving_average": {
      "kernel": [[1, 1, 1, 1, 1], [], []]
    },
    "savitzky-golay": {
      "window_length": [0, 5, 0],
      "polyorder":     [0, 2, 0]
    },
    "gauss": {
      "sigma": [0, 0, 2]
    }
  }
}
```

- Region *i* covers `begin_wavelength[i]` ≤ wavelength < `end_wavelength[i]`; the end is not included.
- For each region the **first** filter with a non-zero entry at that position is used, in the order
  moving average (`kernel`), Savitzky-Golay (`window_length`, with `polyorder`), gauss (`sigma`), lowess
  (`n_points`), loess (`n_points`). An empty kernel or a `0` means "not this filter here". A region with no filter keeps its
  bands unfiltered.
- Each region is filtered on its own: the window does not reach across a region border, and `mode` fills
  the window at the borders.
- **Bands outside every region are dropped.** A multi-filter therefore also trims the spectra, and gaps
  between regions remove bands.

The example applies a 5-band moving average to 1350–1440 nm, Savitzky-Golay (window 5, polyorder 2) to
1450–2390 nm and gauss (sigma 2) to 2400–2500 nm. Together the regions cover the full range, so no band
is dropped:

![Multi-filter]({{ "/assets/media/ml_preprocesses_separate/2-1-6-multi-filter.png" | relative_url }})

```
    Accept and save mf result? [y/n]:
```

| Parameter (`multi_filter_spectra`) | Description |
|---|---|
| `project_root_fp`, `dataframe` | As for `filter_spectra` |
| `multi_filter_fp` | Multi-filter parameter set name or JSON path (required) |
| `name_extension` | Optional; useful since `mf` says nothing about the regions |
| `colormap` | As for `filter_spectra` |

suffix: `.mf` (+ optional `#<name_extension>`)

## Next step

[Scatter correction][pp1_scatter]: remove the offset and slope of each spectrum.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[name_extension]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#name-extension
[overwrite]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#overwrite
[sg_derivative]: {{ site.baseurl }}/machine_learning/ml_preprocess1/derivative/#234-savitzky-golay-derivative
[pp1_scatter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scatter/
[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[setup_db_ml_setting]: {{ site.baseurl }}/setup_db/ml_setting/
