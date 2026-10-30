---
title: "Predict and Register"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Applying saved AI4SoilHealth models to new spectra with predict_independent, and keeping final models in the ml_model registry with register_model and retire_model."
permalink: /machine_learning/ml_predict/
author_profile: false
date: 2026-10-26 08:00:00 +0200
last_modified_at: 2026-10-26 08:00:00 +0200
---

A model saved by [ML modeling][ml_model] is a complete pipeline: give it unprocessed spectra and it returns
the indicator in its native unit. This page covers applying models to new spectra (`predict_independent`)
and keeping the final models in the database registry (`register_model`, `retire_model`).

## Notebook

**Path**: `xspatula_ai4sh/ai4sh/project_neospectra/ml_predict.ipynb`

Three cells: predict, register, retire. They are disabled with `%%script false --no-raise-error`; remove
that line in the cell you want to run.

## Predict independent spectra

`predict_independent` (user stratum 3 and up).

**Process file**: `project_neospectra/process/predict/predict_independent.json`

```json
{
  "process": [
    {
      "process": "predict_independent",
      "overwrite": true,
      "parameters": {
        "project_root_fp": "./project_neospectra/data/compiled/neospectra_10nm",
        "dataframe": "neospectra_10nm",
        "model_array": [
          "./project_neospectra/data/compiled/neospectra_10nm/models/neospectra_10nm_ol_mc_tt/neospectra_10nm_ol_mc_c-org_huber_tt.joblib",
          "c-org_huber_neospectra"
        ],
        "dst_fp": "./project_neospectra/data/predictions/neospectra_10nm"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `project_root_fp` | Folder with the spectra to predict |
| `dataframe` | The parquet with the spectra: its name, or `previous` |
| `model_array` | Models to apply: a `.joblib` path (relative to the project root, with its `.manifest.json` next to it) or the name of a [registered model](#register-a-final-model) |
| `dst_fp` | Output folder. Empty = `<project_root_fp>/predictions` |

**Give unprocessed spectra.** The models refit and apply their own preprocessing. A dataframe whose
companion JSON shows spectral preprocessing (anything but outlier detection) is refused
(`… holds preprocessed spectra … give the unprocessed spectra`): it has the same band names, so it would
give wrong predictions without any error. In the example, `neospectra_10nm` is accepted, but
`neospectra_10nm_ol_mc` is not.
{: .notice--warning}

Before predicting, each model is checked:

- the `.joblib` must match the sha256 in its manifest (and, for a registered model, the sha256 recorded at
  registration);
- the spectra must have all the model's bands (any column order) and the same signal type (the NeoSpectra
  models above use `refl`);
- a different version of scikit-learn, numpy or scipy than the model was saved with gives a warning.

**Output** in `dst_fp`:

- `<name>_pred.parquet`: the key columns of the input plus one column per model,
  `<indicator>__<regressor>_tt`, in the indicator's unit (e.g. `c-org__huber_tt` in weight percent);
- `<name>_pred.json`: the source parquet and its sha256, and per model its file, sha256 and the metrics
  from training.

**Independent validation.** If the parquet also holds observed values of the indicator, the metrics are
computed on the samples that were **not** used to train the model (the training sample ids are in the
manifest). Samples that were used for training are excluded, and the run says how many. Predicting the
model's own compiled dataset, as above, therefore validates on its test samples only. For a registered
model these metrics are also written to the registry (dataset `independent:<parquet name>`; a rerun on the
same parquet replaces them).

## Register a final model

`register_model` (user stratum 4 and 5) records a model as final in the database schema `ml_model`.
Register only models you intend to use: **a registered model is never changed or replaced, only retired.**

**Process file**: `project_neospectra/process/ml_model/register_model.json`

```json
{
  "process": [
    {
      "process": "register_model",
      "parameters": {
        "model_array": [
          "./project_neospectra/data/compiled/neospectra_10nm/models/neospectra_10nm_ol_mc_tt/neospectra_10nm_ol_mc_c-org_huber_tt.joblib"
        ],
        "name": "c-org_huber_neospectra",
        "abstract": "Organic carbon from NeoSpectra reflectance 1350-2500 nm, AI4SH partners 0-20 cm, outliers removed, mean-centred, Huber"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `model_array` | Saved models (`.joblib` with `.manifest.json`), relative to the project root |
| `name` | Registry name (one model only). Empty = the file name without `.joblib` |
| `abstract` | What the model is for, the data it was trained on, and its limits |

Registration stores (see [ML setting and model schemas][setup_db_ml] for the columns):

| Table | Content |
|---|---|
| `ml_model.model` | Name, indicator, regressor, mode, signal type, number of bands and samples, the model file path and sha256, the source data and its sha256, the full manifest, status, who registered it and when |
| `ml_model.model_step` | The preprocessing chain, one row per step with its parameters |
| `ml_model.model_parameter_set` | The [parameter sets][ml_settings] the model used, and in which step |
| `ml_model.model_metric` | The test (or k-fold) metrics, and later the independent metrics |

**The settings a registered model used are frozen.** Every parameter set it used, in the modeling run and
in the preprocessing steps behind its input, becomes frozen: the database refuses to change or delete it,
so the record of how the model was made stays true. To edit those settings, save a new version. Settings
read from files are not in the database; their content and checksum are kept in the manifest.

The model file stays where it is: the registry stores its path and sha256, not the file. Keep registered
model files in a folder you do not clean up (set `model_fp` in `regression_modeling`), and do not overwrite
them: a changed file no longer matches its sha256 and is refused. A rerun of `regression_modeling` with
`"overwrite": true` into the same folder does exactly that.

A file that is already registered, or a name that is already taken, is refused.

## Retire a model

`retire_model` (user stratum 4 and 5).

**Process file**: `project_neospectra/process/ml_model/retire_model.json`

```json
{
  "process": [
    { "process": "retire_model", "parameters": { "model_array": ["c-org_huber_neospectra"] } }
  ]
}
```

A retired model can no longer be applied by name. Its record, metrics and frozen parameter sets are kept.

[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[setup_db_ml]: {{ site.baseurl }}/setup_db/ml_setting/
