---
title: "Compile Dataframe"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Joining the unprocessed spectra, indicator and covariate selections of AI4SoilHealth samples into one compiled dataset: the rules, the companion JSON, covariates, inspecting the result and pre-processing the compiled dataset."
permalink: /machine_learning/compile/
author_profile: false
date: 2026-11-04 08:00:00 +0100
last_modified_at: 2026-11-04 08:00:00 +0100
---

The select processes save spectra, indicators and covariates as separate selections, each in its own folder
(see [Select data][select_data]). A model needs them side by side, one row per sample. The process
`compile_dataset` joins the selections into one **compiled dataset**, saved in a folder of its own. That
folder is the `project_root_fp` of every later step: ML pre-processing 2, modelling and prediction.

Compile comes after [ML pre-processing 1][ml_preprocess] and before the steps that need spectra and
indicators together:

```
select spectra, indicators, covariates
  → ML pre-processing 1 (explore in the selection folders)
  → compile the unprocessed selections          ← this page
  → pre-process the compiled dataset (chain_from)
  → ML pre-processing 2, ML modeling
```

## Notebook

**Path**: `xspatula_ai4sh/ai4sh/project_neospectra/compile_dataframe.ipynb`

Three steps: compile spectra and indicators, compile with covariates (select the covariates first), and
inspect the compiled dataframe.

## Compile dataframe

**Process file**: `project_neospectra/process/compile/compile_neospectra_10nm.json`

```json
{
  "process": [
    {
      "process": "compile_dataset",
      "overwrite": false,
      "parameters": {
        "input_array": [
          "./project_neospectra/data/indicator/ai4sh/cec-clay-ca2-ec-n-tot-c-org-ph-water_0-20.parquet",
          "./project_neospectra/data/spectra/neospectra/ds2/ai4sh/1350-2500_10.parquet"
        ],
        "output_root_fp": "./project_neospectra/data",
        "output_name": "neospectra_10nm"
      }
    }
  ]
}
```

| Parameter (`compile_dataset`) | Description |
|---|---|
| `input_array` | The selections to join, at least two; paths relative to the project root (`.parquet` optional) |
| `output_root_fp` | Root folder of the output; default `./data` |
| `output_sub_fp` | Sub folder under `output_root_fp`; default `compiled` |
| `output_name` | Name of the compiled dataset, required |

The run prints the number of compiled samples and the number of samples in each input, in the order of
`input_array`:

```
    Compiled <n> samples (inputs: <n1>, <n2>) → …/project_neospectra/data/compiled/neospectra_10nm/neospectra_10nm.parquet
```

**Output**: `<output_root_fp>/<output_sub_fp>/<output_name>/<output_name>.parquet` and its companion JSON,
here `data/compiled/neospectra_10nm/`. The output is registered in the folder's `.previous_dataframe`
([The previous dataframe][previous_df]).

### How the inputs are joined

- The inputs are **inner joined** on `sample_id`: only samples present in **every** input are kept. If the
  compiled count is much lower than the input counts, the selections cover different samples, e.g. other
  depths (`min_profile`/`max_profile`), campaigns or sample filters. Compare the input counts in the printed
  line, and the scope in the inputs' companion JSONs.
- **Key columns** (`sample_id`, `sample_name`, `campaign_tag`, `sampling_log_tag`, `latitude_dd`,
  `longitude_dd`, `profile_min`, `profile_max`) are taken from the first input and kept once. `class_*`
  columns already taken from an earlier input are kept once too.
- Any other column found in two inputs stops the compile:
  `❌ ERROR: columns in <input> already in an earlier input: [...]`. Each indicator, covariate and band must
  come from one input only.
- An input without a `sample_id` column stops the compile: rerun the select process that made it.
- An indicator value that is missing in a sample stays missing. The sample is kept, and a model of that
  indicator drops it, while models of other indicators still use it.

### Inputs must be unprocessed

A spectral pre-processing chain is fitted on all samples of its dataframe. The model refits the chain on its
training samples only, starting from the unprocessed source parquet in the same folder ([Exploration versus
modelling][explore]). A chain run in a selection folder points to a source in that folder, so it can not
be refitted from the compiled one. Compile therefore refuses a pre-processed input:

```
    ❌ ERROR: <input> is pre-processed (spectra_scatter_correction → …). Compile the unprocessed selection and pre-process the compiled dataset, e.g. chemometric_chain with chain_from = "<input path>.json"
```

The two **indicator steps** are allowed: [outlier removal][pp1_outliers] (`.olif`, `.olee`, `.ollof`,
`.ol1csvm`) and the [indicator range][pp1_range] (`.ir`). They only set indicator values to missing and leave
the spectra untouched. Their chain is recorded under that input in the compiled companion JSON (see below).
The model does not print a note about them for a compiled dataset; the record in the companion JSON is the
trace.

### Overwrite

With `"overwrite": false`, an existing compiled dataset is left as it is
(`Compiled dataset already exists, skipping: …`), **even if the inputs have changed**. After a new selection
or a new outlier removal, set `"overwrite": true`, or give the compiled dataset a new `output_name`. Models
already trained on the old compiled dataset are not updated; rerun them.

## The compiled companion JSON

The companion JSON of a compiled dataset merges those of its inputs and adds what the later steps read.
Shortened:

```json
{
  "role": "compiled",
  "output_name": "neospectra_10nm",
  "n_samples": "<n>",
  "dataset_tag": "ai4sh",
  "min_profile": 0,
  "max_profile": 20,
  "indicator_array": ["cec", "clay", "ca2+", "ec", "n-tot", "c-org", "ph-water"],
  "indicator_units": {"cec": "cmol*kg^-1", "clay": "weight percent", "…": "…"},
  "covariate_array": [],
  "begin_wavelength": 1350,
  "end_wavelength": 2500,
  "output_bandwidth": 10,
  "column_blocks": {
    "indicator": ["cec", "clay", "ca2+", "ec", "n-tot", "c-org", "ph-water"],
    "covariate": [],
    "spectra": ["wl_nm_1350", "…", "wl_nm_2500"]
  },
  "inputs": [
    {"file": "./project_neospectra/data/indicator/ai4sh/cec-clay-ca2-ec-n-tot-c-org-ph-water_0-20.parquet",
     "role": "indicator", "n_samples": "<n1>"},
    {"file": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh/1350-2500_10.parquet",
     "role": "spectra", "n_samples": "<n2>"}
  ]
}
```

- **Merged from the inputs:**
  - `indicator_array`, `covariate_array` and `class_columns` are joined lists;
  - `indicator_units`, `covariate_rank` and `class_labels` are joined maps;
  - the spectra parameters (`begin_wavelength`, `end_wavelength`, `output_bandwidth`, `output_unit`,
    `output_wavelengths`, provision and preparation) come from the spectra input;
  - other keys come from the first input that has them.
- **`column_blocks`** says which columns are indicators, covariates and spectra. [ML modeling][ml_model]
  reads the covariates from here.
- **`inputs`** lists each input with its file, role and sample count. An input with an indicator step also
  has its `preprocessing_chain` here, e.g. the outlier removal of a `.olif` input.
- **Not taken over:** `preprocessing_chain`, `source_parquet` and `output_data`. The compiled dataset is a new,
  unprocessed starting point.

## Compile with covariates

Covariates are selected on their own with `select_covariate_only` (see
[Select covariates][select_covariates]) and compiled like any other input, next to the indicators and the
spectra. The notebook has two cells for soil texture: the selection, then the compile.

**Process file**: `project_neospectra/process/select/select_ai4sh_texture.json` selects
`soil texture usda` and `soil texture usda major group` into `data/covariate/ai4sh/texture-usda_0-20.parquet`.

**Process file**: `project_neospectra/process/compile/compile_neospectra_10nm_texture.json`

```json
{
  "process": [
    {
      "process": "compile_dataset",
      "overwrite": false,
      "parameters": {
        "input_array": [
          "./project_neospectra/data/indicator/ai4sh/cec-clay-ca2-ec-n-tot-c-org-ph-water_0-20.olif.parquet",
          "./project_neospectra/data/covariate/ai4sh/texture-usda_0-20.parquet",
          "./project_neospectra/data/spectra/neospectra/ds2/ai4sh/1350-2500_10.parquet"
        ],
        "output_root_fp": "./project_neospectra/data",
        "output_name": "neospectra_10nm_texture"
      }
    }
  ]
}
```

- The indicators here are the outlier-cleaned selection (`.olif`), which compile accepts.
- The texture columns are nominal (unit `class`). How a model uses them is explained under
  [Covariates][ml_model_covariates].
- `column_blocks.covariate` lists `soil texture usda` and `soil texture usda major group`.
- The inputs are inner joined, so the compiled dataset only holds the samples that have a texture class. To
  see what the covariate adds, compare a model with covariates against a spectra-only model of the **same**
  compiled dataset (`covariate_array` empty), not against a dataset compiled without the covariates.

### Compile for prediction

A model with covariates needs the covariates next to the spectra of the samples to predict
([Predict and register][ml_predict]). Compile the spectra selection with the same covariate selection;
indicators are not needed. The same rules apply: unprocessed spectra, at least two inputs.

## Inspect the compiled dataframe

**Process file**: `project_neospectra/process/inspect/inspect_latest_compiled_df.json`

```json
{
  "process": [
    {
      "process": "inspect_pandas_dataset",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm",
        "parquet_file": "",
        "column_array": "",
        "max_rows": 10
      }
    }
  ]
}
```

With an empty `parquet_file` it reads the previous dataframe of the compiled folder, i.e. the compiled
dataset just saved. It prints the columns with their units, a sample of the rows and the full size. Check
that every indicator, covariate and the expected bands are there. The parameters are described under
[Inspect the dataset][inspect].

## Pre-process the compiled dataset

The pipeline you chose in [ML pre-processing 1][ml_preprocess] was explored in the spectra folder. To model
it, run it again in the compiled folder with `chemometric_chain` and `chain_from` pointing to the companion
JSON of the explored chain ([Load existing pre-processing chain][chain_from]), e.g.:

```json
{
  "process": [
    {
      "process": "chemometric_chain",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm",
        "dataframe": "raw",
        "chain_from": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh/1350-2500_10.snv.sg5.d1.mc.wc40.pca5.json"
      }
    }
  ]
}
```

- `"dataframe": "raw"` is the compiled dataset itself.
- The chain is refitted on the compiled dataset with the recorded settings. The compiled dataset must have the
  same band range and band width as the explored one; it has, when the same spectra selection was compiled.
- The result, `neospectra_10nm.snv.sg5.d1.mc.wc40.pca5`, is saved in the compiled folder with the compiled
  dataset as its `source_parquet`, so the model can refit the chain on its training samples.

## Next step

ML pre-processing 2 (to be completed): supervised band selection on the compiled dataset. Until then, go to
[ML modeling][ml_model]: train regressors on the compiled or pre-processed compiled dataset.

[select_data]: {{ site.baseurl }}/machine_learning/select_data/
[select_covariates]: {{ site.baseurl }}/machine_learning/select_data/#select-covariates
[previous_df]: {{ site.baseurl }}/machine_learning/parquet_files/#the-previous-dataframe-previous_dataframe
[inspect]: {{ site.baseurl }}/machine_learning/parquet_files/#inspect-the-dataset
[ml_preprocess]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[explore]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#exploration-versus-modelling
[pp1_outliers]: {{ site.baseurl }}/machine_learning/ml_preprocess1/outliers/
[pp1_range]: {{ site.baseurl }}/machine_learning/ml_preprocess1/range/
[chain_from]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/#273-load-existing-pre-processing-chain
[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[ml_model_covariates]: {{ site.baseurl }}/machine_learning/ml_model/#covariates
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
