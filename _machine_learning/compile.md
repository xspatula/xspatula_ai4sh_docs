---
title: "Compile Dataframe"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Compiling AI4SoilHealth indicators with spectra and/or covariates into one dataset, from saved selections or directly from the database: the order of the steps, what each input may have been through, the companion JSON, covariates, indicator steps before or after compiling, and pre-processing the compiled dataset."
permalink: /machine_learning/compile/
author_profile: false
date: 2026-11-04 08:00:00 +0100
last_modified_at: 2026-11-05 14:00:00 +0100
---

The select processes save spectra, indicators and covariates as separate selections, each in its own folder
(see [Select data][select_data]). A model needs them side by side, one row per sample. The process
`compile_dataset` joins the selections into one **compiled dataset**, saved in a folder of its own. That
folder is the `project_root_fp` of every later step: ML pre-processing 2, modelling and prediction. The process
`compile_dataset_from_db` makes the same kind of compiled dataset directly from the database
([Compile directly from the database](#compile-directly-from-the-database)).

A compiled dataset holds **spectra and/or covariates**, and **indicators** to model on. Without indicators it is
saved as a dataset for prediction ([Compile for prediction](#compile-for-prediction)). Without spectra and
covariates both compile processes stop without saving anything.

Compile comes after [ML pre-processing 1][ml_preprocess] and before the steps that need spectra and
indicators together. The order:

1. Select spectra, indicators and covariates ([Select data][select_data]).
2. Optional, in the **indicator folder**: [outlier removal][pp1_outliers] and/or the
   [indicator range][pp1_range] (`.olif`, `.ir`). Explore the spectral pipeline in the spectra folder.
3. **Compile** the unprocessed spectra, the indicators (as selected or after step 2) and the covariates
   (this page). Or select and compile in one step with `compile_dataset_from_db`, which replaces steps 1–3.
4. Optional, in the **compiled folder**: outlier removal and/or indicator range on the compiled dataset,
   instead of step 2 ([Indicator steps before or after compiling](#indicator-steps-before-or-after-compiling)).
5. Spectral chain in the compiled folder: `chemometric_chain` with `chain_from` the explored chain
   ([Load existing pre-processing chain][chain_from]), on the result of step 4, or on the compiled dataset
   itself when step 4 is skipped ([Pre-process the compiled dataset](#pre-process-the-compiled-dataset)).
6. ML pre-processing 2 (to be completed) and [ML modeling][ml_model] on the result of step 5.

Indicator steps come before the spectral chain. The chain's output then carries them in its companion JSON,
and the model reads the cleaned indicators from it while refitting the spectral steps on the training
samples.

## Notebook

**Path**: `xspatula_ai4sh/ai4sh/project_neospectra/compile_dataframe.ipynb`

Five blocks:

| Block | Cells | Content |
|---|---|---|
| 1 Compile dataframes | 1.1.1–1.1.3 | Compile raw indicators and spectra, compile outlier-cleaned indicators, indicator range on a compiled dataset |
| 2 Compile with covariates | 2.1.1–2.1.3 | Select soil texture, compile with raw or outlier-cleaned indicators |
| 3 Pre-process spectra | 3.1.1–3.1.4 | `chain_from` on the compiled datasets; SNV, derivative and mean centring without PCA |
| 4 Single process file | 4.1.1 | `compile_dataset_from_db` and `chemometric_chain` stacked in one process file |
| 5 Inspect | 5.1.1 | Inspect the latest compiled dataframe |

## Compile dataframe

Notebook cell 1.1.1.

**Process file**: `project_neospectra/process/compile/compile_neospectra_10nm_raw.json`

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
- The result must hold spectra and/or covariates, otherwise nothing is saved:
  `❌ ERROR: a compiled dataset needs spectra and/or covariates (and indicators to model on); the inputs give <n> indicator(s), <n> band(s), <n> covariate(s) — nothing saved`.
- An indicator value that is missing in a sample stays missing. The sample is kept, and a model of that
  indicator drops it, while models of other indicators still use it.

### Spectra must be unprocessed

What each input may have been through:

- **Spectra:** nothing. A spectral pre-processing chain is fitted on all samples of its dataframe. The model
  refits the chain on its training samples only, starting from the unprocessed source parquet in the same
  folder ([Exploration versus modelling][explore]). A chain run in a selection folder points to a source in
  that folder, so it can not be refitted from the compiled one.
- **Indicators:** as selected, or after [outlier removal][pp1_outliers] (`.olif`, `.olee`, `.ollof`,
  `.ol1csvm`) and/or the [indicator range][pp1_range] (`.ir`). These two indicator steps only set indicator
  values to missing and leave the spectra untouched. Outlier detection is fitted on all samples, but it is
  data cleaning (which values to trust), not part of the model. The steps are recorded under that input in
  the compiled companion JSON (`inputs[…].preprocessing_chain`).
- **Covariates:** as selected; there are no pre-processes for them. A model standardises numeric covariates
  and pools classes itself ([Covariates][ml_model_covariates]).

Compile refuses an input with any other step:

```
    ❌ ERROR: <input> is pre-processed (spectra_scatter_correction → …). Compile the unprocessed selection and pre-process the compiled dataset, e.g. chemometric_chain with chain_from = "<input path>.json"
```

Example with outlier-cleaned indicators (cell 1.1.2): `project_neospectra/process/compile/compile_neospectra_10nm_olif.json`
has the same inputs as `compile_neospectra_10nm_raw.json`, except the indicators
`cec-clay-ca2-ec-n-tot-c-org-ph-water_0-20.olif.parquet`, and saves `neospectra_olif_10nm`.

The model prints its outlier note (see [ML modeling][ml_model_refit]) only for steps in the chain of the
dataframe it models. For outliers removed **before** compiling, the record under `inputs` in the compiled
companion JSON is the only trace.

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
spectra. The notebook cell 2.1.1 selects soil texture; 2.1.2 and 2.1.3 compile it with raw and with
outlier-cleaned indicators.

**Process file**: `project_neospectra/process/select/select_ai4sh_texture.json` selects
`soil texture usda` and `soil texture usda major group` into `data/covariate/ai4sh/texture-usda_0-20.parquet`.

**Process file**: `project_neospectra/process/compile/compile_neospectra_10nm_texture_olif.json`

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
        "output_name": "neospectra_10nm_olif_texture"
      }
    }
  ]
}
```

- The indicators here are the outlier-cleaned selection (`.olif`), which compile accepts.
  `compile_neospectra_10nm_texture_raw.json` compiles the same with the indicators as selected, into
  `neospectra_10nm_texture`.
- The texture columns are nominal (unit `class`). How a model uses them is explained under
  [Covariates][ml_model_covariates].
- `column_blocks.covariate` lists `soil texture usda` and `soil texture usda major group`.
- The inputs are inner joined, so the compiled dataset only holds the samples that have a texture class. To
  see what the covariate adds, compare a model with covariates against a spectra-only model of the **same**
  compiled dataset (`covariate_array` empty), not against a dataset compiled without the covariates.

### Compile for prediction

A model with covariates needs the covariates next to the spectra of the samples to predict
([Predict and register][ml_predict]). Compile the spectra selection with the same covariate selection, or run
`compile_dataset_from_db` with an empty `indicator_array`. The same rules apply: unprocessed spectra and at least
two inputs. Without indicators the dataset is saved for prediction, with a note:

```
    ⚠️  no indicators — saved as a dataset for prediction (purpose: prediction), not for modeling
```

The companion JSON records `"purpose": "prediction"` (`"modeling"` when it holds indicators).
`predict_independent` takes such a dataset; `regression_modeling` refuses it. Add the indicator selection when
the new samples have indicator values: `predict_independent` then also validates the model on them.

## Compile directly from the database

`compile_dataset_from_db` selects indicators, spectra and/or covariates for the samples in scope in one call
and saves a compiled dataset, without separate selections. It takes the parameters of the select processes
([Select data][select_data]): the scope, `indicator_array`, `data_range`, the spectra parameters,
`covariate_array`, `observation_log_array` and `aggregate`.

The notebook cell 4.1.1 also shows that one process file can **stack several processes**, run in order. Here the
compile is followed by the spectral chain.

**Process file**: `project_neospectra/process/compile/compile_wetlab+neospectra_10nm_from_db.json` (shortened)

```json
{
  "process": [
    {
      "process": "compile_dataset_from_db",
      "overwrite": false,
      "parameters": {
        "dataset": "ai4sh",
        "campaign": "",
        "sampling_log": "",
        "indicator_array": ["clay", "n-tot", "c-org", "ph-water"],
        "data_range": "{\"c-org\": {\"min\": 0, \"max\": 10}, \"n-tot\": {\"min\": 0, \"max\": 1}}",
        "spectra_provision": "neospectra",
        "spectra_preparation_name": "ds2",
        "observation_log_array": [
          "ai4sh_dk_foulum_2024@neospectra.ds2",
          "ai4sh_es_arkaute_2024@neospectra.ds2",
          "…"
        ],
        "begin_wavelength": 1350,
        "end_wavelength": 2500,
        "output_bandwidth": 10,
        "output_spectra_signal": "reflectance",
        "output_root_fp": "./project_neospectra/data",
        "output_name": "neospectra_10nm_db"
      }
    },
    {
      "process": "chemometric_chain",
      "overwrite": false,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm_db",
        "dataframe": "neospectra_10nm_db",
        "chain_from": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh/1350-2500_10.snv.d1.mc.pca5.json"
      }
    }
  ]
}
```

- **Output:** `<output_root_fp>/compiled/<output_name>/<output_name>.parquet` and its companion JSON, here
  `data/compiled/neospectra_10nm_db/`, registered in the folder's `.previous_dataframe`. The companion has
  `role: compiled`, `column_blocks` and `inputs` with `"source": "database"`. The folder is used as
  `project_root_fp` exactly like one from `compile_dataset`. Without `output_name`, the name is built from the
  wavelength range, the indicator and covariate aliases and the depth.
- **Spectra and/or covariates, and indicators to model on.** `spectra_provision` is optional; leave it empty
  for a dataset of indicators and covariates only (the wavelength parameters are then ignored). An empty
  `indicator_array` gives a dataset for prediction ([Compile for prediction](#compile-for-prediction)). With
  neither spectra nor covariates the process stops before retrieving anything:
  `❌ ERROR: compile_dataset_from_db requires spectra (spectra_provision) and/or covariates (covariate_array), and indicators (indicator_array) for a dataset to model on`.
  If the samples in scope have no spectra or covariate values:
  `❌ ERROR: no spectra or covariates with data for the samples in scope — nothing saved`.
- **Indicator steps:** only `data_range`, applied while selecting: a value outside its range is set to
  missing ([Select indicators][select_indicators]). It is recorded as `data_range` in the companion JSON.
  There is no outlier removal in `compile_dataset_from_db`; for that, run `detect_outliers` in the compiled
  folder and give its result to the chain
  ([Indicator steps before or after compiling](#indicator-steps-before-or-after-compiling)).
- **`observation_log_array`** lists only the spectral logs here. A listed log restricts only its own kind of
  data, so every laboratory value is kept ([Observation logs][select_obs_logs]).
- **The chain must exist.** `chain_from` points at a chain explored and saved before, in
  [ML pre-processing 1][ml_preprocess]. The result is `neospectra_10nm_db.snv.d1.mc.pca5`.

**Which compile process.** Use `compile_dataset_from_db` when the selection is settled and the indicators need no
outlier removal before compiling: one process file from the database to a pre-processed dataset, easy to rerun
with another scope. Use separate selections and `compile_dataset` while exploring: plotting and pre-processing the
indicators and spectra on their own ([ML pre-processing 1][ml_preprocess]), and reusing one selection in several
compiled datasets.

**Covariates without spectra.** A dataset of indicators and covariates only (empty `spectra_provision`, or
`compile_dataset` without a spectra input) can be modelled with `covariate_array` set
([Covariates][ml_model_covariates]). The spectral processes (chains, band selection) need spectra.

## Inspect the compiled dataframe

Notebook cell 5.1.1.

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

## Indicator steps before or after compiling

Outlier removal and the indicator range work on any dataframe with indicator columns, the compiled one
included. Run them either:

- **before compiling**, in the indicator folder, and compile the `.olif`/`.ir` file (above); or
- **after compiling**, with `project_root_fp` set to the compiled folder. The steps then see only the samples
  that have spectra (and covariates), and you can add a step without compiling again.

For example, the indicator range on the compiled dataset (notebook cell 1.1.3).

**Process file**: `project_neospectra/process/compile/indicator_pre-process_compiled_neospectra_10nm.json`

```json
{
  "process": [
    {
      "process": "indicator_range",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm",
        "dataframe": "neospectra_10nm",
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

The result, `neospectra_10nm.ir`, is saved in the compiled folder with the compiled dataset as its
`source_parquet`. Outlier removal works the same way, with the parameters of [Outlier removal][pp1_outliers]
(result `neospectra_10nm.olif`).

- **Covariates are skipped.** With an empty `indicator_array`, the indicator steps process every numeric
  column except the covariates of the dataset (`column_blocks.covariate`, or `covariate_array` of a
  selection), and say so:
  `Covariates not processed (name them in indicator_array to include): <covariates>`. A covariate named in
  `indicator_array`, or in `data_range` of the indicator range, is processed; a value set to missing then
  drops that sample from every model that uses the covariate. Class covariates are never processed.
- **No `chain_from` for indicator steps.** To repeat them on another dataset, run the process again. The
  indicator range repeats exactly with `"interactive": false` and the same `data_range` (the applied
  ranges are in the companion JSON). Outlier detection is fitted on the data, so on another sample set it
  flags other values, by design.
- **Before the spectral chain.** Run the indicator steps first and give their result to `chemometric_chain`
  (below). The reverse order, an indicator step on the chain's output (e.g. `….pca5.olif`), gives the same
  model, but each spectral pipeline then needs its own outlier run, and the models of different pipelines
  no longer share the same cleaned indicators.

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

- `"dataframe": "raw"` is the compiled dataset itself. After an indicator step in the compiled folder,
  name its result instead, e.g. `"dataframe": "neospectra_10nm.olif"`, which gives
  `neospectra_10nm.olif.snv.sg5.d1.mc.wc40.pca5`. `"previous"` works too, but it is the dataframe most
  recently saved (or found already saved) in the folder by any process, so name the file when several
  processes have run there. The values are described in [The input dataframe][input_df].
- The input must not be spectrally pre-processed. The chain is applied on top of the input without a
  warning, e.g. `neospectra_10nm.snv` with an SNV chain gives SNV twice. The model then refits the same
  doubled steps.
- The chain is refitted on the compiled dataset with the recorded settings. The compiled dataset must have the
  same band range and band width as the explored one; it has, when the same spectra selection was compiled.
- For PLSR and the other multivariate regressors, leave out the final PCA, e.g.
  `project_neospectra/process/compile/pre-process_snv-d1-mc_neospectra_10nm_olif_texture.json` with
  `"chemometrics_array": ["snv", "derivative", "meancentring"]` on `neospectra_10nm_olif_texture`, which saves
  `neospectra_10nm_olif_texture.snv.d1.mc` (see [Choosing a regressor][ml_model_choose]).
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
[select_indicators]: {{ site.baseurl }}/machine_learning/select_data/#select-indicators
[select_obs_logs]: {{ site.baseurl }}/machine_learning/select_data/#observation-logs-subsamples-and-replicates
[ml_model_choose]: {{ site.baseurl }}/machine_learning/ml_model/
[ml_model_refit]: {{ site.baseurl }}/machine_learning/ml_model/#the-preprocessing-is-refitted-inside-the-model
[input_df]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#the-input-dataframe
[ml_model_covariates]: {{ site.baseurl }}/machine_learning/ml_model/#covariates
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
