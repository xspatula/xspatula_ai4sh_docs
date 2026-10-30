---
title: "Plot Data"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Plotting selected AI4SoilHealth indicators, class distributions and spectra before machine learning preprocessing."
permalink: /machine_learning/plot_data/
author_profile: false
date: 2026-10-19 08:00:00 +0200
last_modified_at: 2026-10-26 08:00:00 +0200
---

Look at the selection before preprocessing: the distribution of each indicator, the class counts of a
filter, and the spectra under different filters and scalers. The plots help decide which transformation,
standardisation and spectral enhancement to apply in ML preprocessing. The code is the same as for LUCAS
([LUCAS Plot Data][lucas_plot]).

The AI4SH plot process files are in `xspatula_ai4sh/ai4sh/project_neospectra/process/plot/`
(`plot_ai4sh_indicators.json`, `plot_neospectra*.json`, `plot_filter_neospectra.json`). Their
`project_root_fp` points at the selections `data/indicator/ai4sh` and `data/spectra/neospectra/ds2/ai4sh`;
update it if you select into other folders (see [Select data][select_data]).

## Which file is plotted

`project_root_fp` points at a folder, not a file. The plot reads the latest selected parquet registered in
the folder's `.previous_dataframe`, i.e. normally the selection you just ran (see
[The previous dataframe][previous_dataframe]).

## Plot indicators and covariates

`plot_indicators` reads `indicator_array` and `covariate_array` from the selection in `project_root_fp`:

```json
{
  "process": [
    {
      "process": "plot_indicators",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/indicator/ai4sh",
        "indicator_array": ["cec", "clay"],
        "targetfeaturesymbols": "default",
        "standardisation": "default",
        "transformation": "default",
        "single_canvas_plot": true,
        "individual_canvas_plot": false,
        "boxplot": true,
        "histogram": true,
        "show": true,
        "save": true
      }
    }
  ]
}
```

- Without `indicator_array`, every column except the key, `class_*` and band columns is plotted.
- Numeric indicators get a boxplot and a histogram. `single_canvas_plot` draws all indicators on one
  canvas, `individual_canvas_plot` one canvas per indicator.
- Saved to `<project_root_fp>/plot/`, e.g. `boxplot_all-indicators_z_log.png` (the suffix names the
  standardisation and transformation), or `boxplot_<indicator>.png` per indicator.
- `"default"` for `targetfeaturesymbols`, `standardisation` and `transformation` means the parameter set
  `targetfeaturesymbols` (family `plot_symbol`) and the `standard` standardisation and transformation sets.
  Give a set name (`"ai4sh_log_carbon"`, `"name@2"`) to use another set, or `"none"` for no
  transformation or standardisation. See [ML settings][ml_settings].
- Colours, labels and axis units come from the `plot_symbol` set `targetfeaturesymbols`, the same set used
  for [indicator units][indicator_units]. `ai4sh/default/plot/targetfeaturesymbols.json` is only the
  fallback before `setup_db` has created the `ml_setting` schema. In the plot no unit translation happens.
- The transformation shown in a plot is not passed on to the model: set `indicator_transformation` in
  [ML modeling][ml_model].

## Plot a filter's class distribution

To see how a class filtered selection splits, put its `class_*` column in `covariate_array`:

```json
"indicator_array": ["clay"],
"covariate_array": ["class_juxtaposition"]
```

Nominal columns can't be histogrammed or transformed. They are drawn as horizontal bars of class counts,
largest on top, saved as `classes_<name>.png` (here `classes_class_juxtaposition.png`).

| Parameter | Description |
|---|---|
| `class_label` | `display_name` (default), `tag`, `name` or `alias`. Only changes the plot labels |
| `max_classes` | Max number of classes per covariate, default 15; the rest are summed as `other (N classes)` |

The `targetfeaturesymbols` set has no entry for `class_*` names, so the default bar style applies.

### Coming next

Colouring the indicator plots by class, and marker shape by campaign or sampling log, are planned. With
many AI4SH sites this matters more than for LUCAS: a pooled model is only trustworthy if no single site
dominates or behaves differently. The columns needed (`class_*`, `campaign_tag`, `sampling_log_tag`) are
already in every parquet. There are no plot parameters for it yet.

## Spectra plots

The four spectra plots read the spectra selection. Set `project_root_fp` in all four to the spectra
folder, e.g. `./project_neospectra/data/spectra/<provision tag>/ds2/ai4sh`.

| Process file | Plots |
|---|---|
| `plot_neospectra.json` | the selected spectra, no processing |
| `plot_neospectra_scattercorrection_scalers.json` | raw spectra against each scatter correction (SNV, MSC, ...) |
| `plot_neospectra_scaling_methods.json` | raw spectra against each scaling method (meancentring, autoscaling, paretoscaling, poissonscaling) |
| `plot_filter_neospectra.json` | raw spectra against each smoothing filter (moving-average, gauss, Savitzky-Golay, lowess) |

`colormap` sets the colour ramp of the spectra. Its default is `"default"`, the `spectra_plot` set
`standard` (= `jet`); `"none"` gives the matplotlib default (viridis), and any matplotlib name
(`"plasma"`, `"jet"`) also works. The `chemometrics` and `filter` parameters take set names in the same way
(see [ML settings][ml_settings]).

## Next step

An AI4SH page for preprocessing is still to be written. Until then, follow the LUCAS
[ML preprocessing][lucas_preprocess] page with the compiled AI4SH dataset as `project_root_fp` (see
[Compile the project dataset][compile]). Then name your settings ([ML settings][ml_settings]), train the
models ([ML modeling][ml_model]) and apply or register them ([Predict and register][ml_predict]).

[select_data]: {{ site.baseurl }}/machine_learning/select_data/
[previous_dataframe]: {{ site.baseurl }}/machine_learning/parquet_files/#the-previous-dataframe-previous_dataframe
[indicator_units]: {{ site.baseurl }}/machine_learning/parquet_files/#indicator-units
[compile]: {{ site.baseurl }}/machine_learning/parquet_files/#compile-the-project-dataset
[lucas_plot]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/plot_data/
[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
[lucas_preprocess]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/ml_preprocess/
