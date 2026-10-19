---
title: "Parquet Files"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "What an AI4SoilHealth selection saves: Parquet file and companion JSON, output names, columns, units, invalid values, the .previous_dataframe history, inspecting and compiling."
permalink: /machine_learning/parquet_files/
author_profile: false
date: 2026-10-19 08:00:00 +0200
last_modified_at: 2026-10-19 08:00:00 +0200
---

Every [select process][select_data] saves its result locally as a Parquet file. This page explains what
is in it, how it is named, and how to inspect it and compile several selections into one dataset. The
code is the same as for LUCAS ([LUCAS Parquet Files][lucas_parquet]).

## What a select process saves

Two files with the same name:

- `<name>.parquet`: the data, with the unit of each column embedded as column metadata.
- `<name>.json`: the companion file, recording how the selection was made.

The selection is also registered in the folder's [.previous_dataframe](#the-previous-dataframe-previous_dataframe).

## Output names

Output paths are built from tags (`_Ml_output_paths` in `select.py`):

```
{output_root_fp}/{role}/{provision tag}/{preparation alias}/{scope tag}/{begin}-{end}_{bandwidth}[__{filter}].parquet   spectra
{output_root_fp}/{role}/{scope tag}/{aliases}_{min}-{max}[__{filter}].parquet                                        indicator, covariate
{output_root_fp}/compiled/{output_name}/{output_name}.parquet                                                         compile
```

- The **scope tag** is the most specific one given: sampling_log > campaign > dataset. For all AI4SH
  samples it is `ai4sh`, for one site e.g. `ai4sh_dk_foulum`.
- An indicator or covariate list longer than 60 characters is cut and given an 8-character hash of the
  full list.
- `output_root_fp` is the AI4SH project's data folder, e.g. `./project_neospectra/data`.
- Set `output_sub_fp` and/or `output_name` to override the defaults.

**Filter part.** A class filtered selection gets `__<filter part>` appended to the default name: per
filter its abbreviation and entries, lower-cased, every character outside `a-z0-9.` replaced by `-`,
filters joined by `_`. Longer than 60 characters, it is cut and given an 8-character hash. Examples:

| Filter | Name part |
|---|---|
| `juxtaposition_array: ["furrow", "plant row"]` | `jx-furrow-plant-row` |
| `composition_array: ["individual"]` | `cp-individual` |
| `proximity_array: ["central", "peripheral"]` | `px-central-peripheral` |
| `composition_array` + `proximity_array` above | `cp-individual_px-central-peripheral` |

The indicator selection on [Select data][select_indicators] (`cec`, `clay`, 0–100 cm, furrow/plant row,
all sites) is saved as:

```
data/indicator/ai4sh/cec-clay_0-100__jx-furrow-plant-row.parquet
```

`output_suffix` replaces the automatic filter part. With a given `output_name` there is no filter part,
but an `output_suffix` is still appended (`__<suffix>`). `none` leaves it out. Default `default`.

## Columns

**Key columns**, in every output: `sample_id` (the join key; sample names are only unique within a
sampling log), `sample_name`, `campaign_tag`, `sampling_log_tag`, `latitude_dd`, `longitude_dd`,
`profile_min`, `profile_max`.

**Class columns**: one `class_<filter>` column per class filter, e.g. `class_juxtaposition`. The value is
the filter entry the sample matched (`furrow` or `plant row`).

**Data columns**: the spectral bands, indicators or covariates.

`campaign_tag` and `sampling_log_tag` let a model pooled over all sites be plotted or evaluated per site
and survey round. ML preprocessing and modelling treat key and `class_*` columns as metadata, never as
features.

## The companion JSON

Records the scope, the filters and the result. Shortened (the real file also has the `*_name` fields,
`covariate_array`, `covariate_rank`, `class_labels` and `n_samples`, the number of samples saved):

```json
{
  "role": "indicator",
  "dataset_tag": "ai4sh",
  "campaign_tag": "",
  "sampling_log_tag": "",
  "min_profile": 0,
  "max_profile": 100,
  "indicator_array": ["cec", "clay"],
  "indicator_units": {"cec": "cmol*kg^-1", "clay": "weight percent"},
  "data_range": {},
  "sample_filter": {"juxtaposition": ["furrow", "plant row"]},
  "sample_source": "",
  "class_columns": ["class_juxtaposition"],
  "output_name": "cec-clay_0-100__jx-furrow-plant-row"
}
```

`sample_filter` is the record of how the file was made (with `texture_classification` added when texture
is filtered). A selection made with [`sample_source`][sample_source] inherits the source's
`sample_filter`.

## Indicator units

Different provisions and laboratories can record the same indicator in different units. The select
processes harmonise them at extraction, so every value in the saved `.parquet` is in one unit per
indicator.

**Where the target unit comes from**: `ai4sh/default/plot/targetfeaturesymbols.json`, keyed by indicator
name, e.g.:

```json
"c-org": {
  "color": "dimgray",
  "alpha": 0.2,
  "label": "Organic carbon (C)",
  "unit": "native"
}
```

All indicators in the default file use `"unit": "native"`. Point `targetfeaturesymbols` at a different
file (a path, instead of `"default"`) to use your own settings. The same file drives the axis units and
colours of the [plots][plot_data].

**Two ways to set an indicator's `unit`:**

- `"native"`: keep the unit the DB recorded, no translation. If the DB has mixed native units for one
  indicator, the process prints a warning and uses the first one as the saved unit label. The values are
  left untouched, so check that this isn't silently mixing units.
- A specific unit name (e.g. `"percent"`): every record is translated via
  `observation_utility.unit_translate`. If the translation is missing, the process raises an error naming
  the project folder and the row to add to
  `ai4sh/import_data/utility/observation/excel/unit_translate.xlsx` (source unit, destination unit). Add
  the row, then re-run cell 1.2 (Load observation utilities) of [insert_ai4sh_data.ipynb][all_data].

**Combining selections**: if you compile selections from different provisions or sites, set the *same*
target unit for every indicator they share.

**Where it ends up**: in the `.json`'s `indicator_units` and as column metadata in the `.parquet` (see
[Inspect the dataset](#inspect-the-dataset)). Nominal columns get the unit `class`.

## Invalid values

Two safety nets keep invalid values out of the models.

**Selection** (`_Screen_values` in `select.py`): every numeric indicator or covariate is checked after
unit translation. A value is set to missing if it is

- a missing-value code: `-9999`, `-999`, `-888` (`MISSING_VALUE_CODES`), or
- outside the physical range of its unit (`Valid_unit_range`): percent 0–100, permill 0–1000,
  `g*kg^-1` 0–1000, `mg*kg^-1` 0–1e6, pH 0–14, and no negative contents, concentrations,
  conductivities, lengths or areas.

Each removal is printed per indicator, e.g.
`⚠️  clay: 3 value(s) are missing codes or outside the valid range for weight percent, set to missing: [-999.0]`.

**Transformation** (`chemometrics.py`): log of values ≤ 0, square root of values < 0 and the reciprocal
of 0 give a missing value with a warning, instead of ±inf that crashes the plots.

## The previous dataframe (`.previous_dataframe`)

Every select process, `compile_dataset` and every ML preprocessing step appends a record to the hidden
file `.previous_dataframe` in the folder it saved to: process, time, note and file.

- The last record is **the previous dataframe**. Inspect reads it when `parquet_file` is empty, plot
  reads the latest selected file in it, and ML preprocessing reads it with `"dataframe": "previous"`.
  A notebook thus runs select → inspect → plot → preprocess without repeating file names.
- The earlier records are the folder's history, oldest first.
- Spectra, indicator and covariate folders each keep their own history.

Example (from LUCAS, same format):

```json
{
  "history": [
    {"process": "select_indicator_only", "created_at": "2026-10-19T08:49:03",
     "note": "indicator: 199 samples",
     "entries": [{"dataframe": "clay-c-org_0-20.parquet", "indicator": null, "regressor": null, "selector": null}]},
    {"process": "select_indicator_only", "created_at": "2026-10-19T08:49:04",
     "note": "indicator: 23 samples, filter {'landcover': ['cropland']}",
     "entries": [{"dataframe": "clay-c-org_0-20__lc-cropland.parquet", "indicator": null, "regressor": null, "selector": null}]}
  ]
}
```

A folder written before this change has no history until a select runs there again. Inspect then says
`ERROR: no .previous_dataframe found in ...`.

[Plot data][plot_data] picks the latest *selected* (not derived) file from the history. Without a
history it falls back to the newest file by date, and lists the others in a warning
(`Find_selected_parquet`). Derived files (with step suffixes like `_ol`, `_mc`) are never candidates.

## Inspect the dataset

`inspect_pandas_dataset` shows any `.parquet` along the pipeline: column names and units, a sample of the
data, and the dataframe's full size. With an empty `parquet_file` (or `none`/`previous`) it reads the
previous dataframe:

```json
{
  "process": [
    {
      "process": "inspect_pandas_dataset",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/indicator/ai4sh",
        "parquet_file": "",
        "column_array": "",
        "max_rows": 10
      }
    }
  ]
}
```

It first names what it read, e.g.
`Previous: select_indicator_only — cec-clay_0-100__jx-furrow-plant-row.parquet (...)`. Give
`parquet_file` a file name to inspect a specific file.

| Parameter | Description |
|---|---|
| `project_root_fp` | Folder of the parquet file |
| `parquet_file` | File relative to `project_root_fp`; empty, `none` or `previous` reads `.previous_dataframe` |
| `column_array` | CSV list of columns to show; empty shows every column |
| `columns_units_data` | Show column names, units and data (default `true`) |
| `columns_data` | Show column names and data, without units |
| `columns_only` | Show only the column names |
| `columns_units_only` | Show only the column names and units |
| `max_rows` | Maximum number of rows to print; `0` lets Pandas decide (default) |
| `show_size` | Print the dataframe's full row/column count (default `true`) |

Set only one of the four `columns_*` views at a time. A name in `column_array` that isn't a column is
skipped with a warning, and the available columns are listed.

## Compile the project dataset

Joins saved selections into one dataset. The compiled folder is the `project_root_fp` of all later ML
steps.

```json
{
  "process": [
    {
      "process": "compile_dataset",
      "overwrite": false,
      "parameters": {
        "input_array": [
          "./project_neospectra/data/spectra/<provision tag>/ds2/ai4sh/400-2500_10__jx-furrow-plant-row.parquet",
          "./project_neospectra/data/indicator/ai4sh/cec-clay_0-100__jx-furrow-plant-row.parquet"
        ],
        "output_root_fp": "./project_neospectra/data",
        "output_name": "ai4sh_furrow_plant_row"
      }
    }
  ]
}
```

- `input_array` needs at least two files; `output_name` is required.
- The inputs are **inner joined** on `sample_id`: only samples present in all inputs are kept.
- Key columns are kept once. `class_*` columns repeated across inputs are kept once. Any other column
  found in two inputs is an error.
- Units, `class_columns`, `covariate_rank` and `class_labels` are merged. The companion JSON lists each
  input with its role and sample count.
- Compile registers its output in `.previous_dataframe`.

**Output**: `data/compiled/<output_name>/<output_name>.parquet` (+ `.json`).

## Next step

[Plot data][plot_data]: look at the selection before preprocessing.

[select_data]: {{ site.baseurl }}/machine_learning/select_data/
[select_indicators]: {{ site.baseurl }}/machine_learning/select_data/#select-indicators
[sample_source]: {{ site.baseurl }}/machine_learning/select_data/#reuse-a-selections-samples-sample_source
[plot_data]: {{ site.baseurl }}/machine_learning/plot_data/
[all_data]: {{ site.baseurl }}/all_data/
[lucas_parquet]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/parquet_files/
