---
title: "ML Pre-processing 1: Indicator Range"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Setting indicator values outside a [min, max] range to missing, interactively from the plotted distributions or with fixed ranges, e.g. organic carbon above 10 % for a model of agricultural soils."
permalink: /machine_learning/ml_preprocess1/range/
author_profile: false
date: 2026-11-03 08:00:00 +0100
last_modified_at: 2026-11-05 14:00:00 +0100
---

Section 1.2 of the notebook `ml_preprocess_separated.ipynb`. Like [outlier removal][pp1_outliers], the
indicator range works on the indicator selection, not on the spectra. See [ML pre-processing 1][pp1] for input
dataframes, naming and the companion JSON.

## Why a range

Outlier detection flags values that lie outside the bulk of an indicator's own distribution. An indicator range
instead sets **fixed limits** that define the population the model is meant for. A model of organic carbon in
agricultural soils, where the span is usually up to 5 % (sometimes 8–10 %), is distorted by a few organic or
peat soils with 25 % or more: they dominate the model error and make the estimated error unrealistically high
for the soils the model is meant for. The range `c-org 0–10` removes them.

`data_range` in the select processes applies the same rule while selecting (see
[Select indicators][select_indicators]). `indicator_range` adds the interactive check of the limits, and can run on
a saved selection or a compiled dataset.

## How it works

The process `indicator_range`:

- **Start ranges** come from `data_range` in the process file, the same JSON format as in
  `select_indicator_only`: `"{\"c-org\": {\"min\": 0, \"max\": 10}}"` (a JSON string). Omit `min` or `max` for an
  open end. An indicator without an entry starts with no limits. `indicator_array` limits which indicators are
  shown and ranged (empty = all numeric indicators); a `data_range` entry for an indicator not in
  `indicator_array` is ignored.
- **What you see:** a table per indicator (n, min, 1 %, median, 99 %, max, current range, number of values
  outside) and a strip plot with the limits as dashed red lines and the values outside in red.
- **Changing a range:** type `<indicator> <min> <max>`, e.g. `c-org 0 8`, or `ph-water - 8.5` for no lower
  limit (`-` = open end). The table and the plot are redrawn. Enter accepts, `0` skips (nothing is saved). With
  `"interactive": false`, `data_range` is applied as given, without a prompt.
- **Result:** values outside the range become missing (NaN) for that indicator only; the rest of the sample
  (other indicators, spectra) is kept. A model of that indicator then drops the sample, while models of other
  indicators still use it. Nothing is saved if no value is outside.
- The file gets the suffix `.ir` (optional `#<name_extension>`), and the applied ranges are recorded in the
  preprocessing chain of the companion JSON (`process: indicator_range`, `parameters.data_range`). Only the
  ranges that set at least one value to missing are recorded.

## 1.2 Remove indicator values outside a range

**Process file**: `project_neospectra/process/ml_preprocess/indicator_range.json`

```json
{
  "process": [
    {
      "process": "indicator_range",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/indicator/ai4sh",
        "dataframe": "previous",
        "indicator_array": ["c-org", "n-tot"],
        "data_range": "{\"c-org\": {\"min\": 0, \"max\": 10}, \"n-tot\": {\"min\": 0, \"max\": 1}}",
        "interactive": true,
        "show": true,
        "save": true
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `project_root_fp` | The indicator selection folder |
| `dataframe` | Input dataframe: `raw` (the selected indicator dataset, default), `previous`, or a dataframe name |
| `indicator_array` | Indicators to show and range; empty = all numeric indicators |
| `data_range` | Start range per indicator, a JSON string `{"<indicator>": {"min": …, "max": …}}`; empty = all open |
| `interactive` | `true` (default): change ranges at the prompt; `false`: apply `data_range` as given |
| `show`, `save` | Show the plot (default `true`); save it as `plot/range_<indicators>.png` (default `false`) |
| `name_extension` | Optional, see [Name extension][name_extension] |

The process `indicator_range` is registered by `setup_processes.ipynb`; rerun it after updating xspatula_ai4sh.
{: .notice--info}

The example runs on the previous dataframe, i.e. after the outlier removal of 1.1, and starts from c-org 0–10 %
and n-tot 0–1 %. Set `"dataframe": "raw"` to range the selected indicators directly. When the cell runs, the
table is printed:

```
    indicator           n       min        p1    median       p99       max   range              outside
    c-org             <n>     <min>      <p1>  <median>     <p99>     <max>   [0, 10]            <n>
    n-tot             <n>     <min>      <p1>  <median>     <p99>     <max>   [0, 1]             <n>
```

and the prompt asks for changes:

```
    Change a range: <indicator> <min> <max> ("-" = open end); Enter = accept, 0 = skip:
```

The result is saved as `<indicator dataset>.ir`, e.g. `cec-clay-ca2-ec-n-tot-c-org-ph-water_0-20.olif.ir.parquet`
after outlier removal, with the message
`Set <n> values outside the range to missing across <k> indicators (c-org [0, 10], …). Saved: …`.

suffix: `.ir` (+ optional `#<name_extension>`)

## In a model

Unlike outlier detection, nothing is learned from the samples: the limits are fixed, so the step is the same for
training and independent data. A ranged indicator file can be [compiled][compile] and used for prediction, and
[ML modeling][ml_model] does not refit it. Outlier detection and indicator range
are the two **indicator steps**: they only set indicator values to missing.

Typical order: outlier detection (1.1), then indicator range (1.2) with `"dataframe": "previous"`; or the range
alone on the raw selection.

The indicator range also runs on a compiled dataset: set `project_root_fp` to the compiled folder. With an empty
`indicator_array` it skips the covariates of the compiled dataset; a covariate named in `indicator_array` or
`data_range` is ranged. See [Indicator steps before or after compiling][compile_indicator_steps].

## Next step

[Filters][pp1_filter]: smooth the spectra.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[pp1_outliers]: {{ site.baseurl }}/machine_learning/ml_preprocess1/outliers/
[name_extension]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#name-extension
[pp1_filter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/filter/
[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[compile]: {{ site.baseurl }}/machine_learning/compile/
[select_indicators]: {{ site.baseurl }}/machine_learning/select_data/#select-indicators
[compile_indicator_steps]: {{ site.baseurl }}/machine_learning/compile/#indicator-steps-before-or-after-compiling
