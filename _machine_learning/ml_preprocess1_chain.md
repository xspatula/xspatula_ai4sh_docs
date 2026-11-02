---
title: "ML Pre-processing 1: Chemometric Chain"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Running a complete spectral pre-processing pipeline in one process with chemometric_chain: the chemometrics_array syntax, a simple and a complete pipeline, and rerunning a saved chain with chain_from."
permalink: /machine_learning/ml_preprocess1/chain/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-10-31 08:00:00 +0100
---

Section 2.7 of the notebook `ml_preprocess_separated.ipynb`. See [ML pre-processing 1][pp1] for input
dataframes, naming and the companion JSON.

The single pre-processes of sections 2.1–2.6 are for exploring: you look at each step and choose its
settings. Once you know the pipeline you want, `chemometric_chain` runs it in one go.

## How the chain works

- The steps in `chemometrics_array` are applied in the listed order, each to the output of the step before.
- The whole array is **validated first**. An unknown method or a missing setting stops the process with an
  error that names the item, before anything is computed.
- The chain is **not interactive**: no threshold or cluster prompts. All steps are plotted together, and the
  result is saved after one `Accept and save <suffix> result? [y/n]`.
- Only the **final** dataframe is saved, not the intermediate steps.
- The output has the **same file name and the same companion `preprocessing_chain`** as running the steps
  one by one. Modelling therefore replays it the same way ([Exploration versus modelling][explore]).

| Parameter (`chemometric_chain`) | Description |
|---|---|
| `project_root_fp` | The spectra selection folder |
| `dataframe` | Input dataframe: `raw` (default), `previous` or a file name |
| `chemometrics_array` | The steps, see the syntax below |
| `chain_from` | Instead of `chemometrics_array`: rerun the chain of a saved dataframe (2.7.3) |
| `name_extension` | Optional, see [Name extension][name_extension] |
| `colormap` | Colour ramp of the plots |

## chemometrics_array syntax

Each item is either a **method name**, applied with its default settings, or an **object** with one key
and custom settings:

| Step | Method name (default settings) | Object (custom settings) | Suffix |
|---|---|---|---|
| Filter | `moving-average`, `gauss`, `savitzky-golay`, `lowess` (the [filter presets][filter_presets]) | `{"filter": "<parameter set name or JSON file>"}` | `ma5`, `gf2`, `sg5`, `lw21` … |
| Multi-filter | — | `{"multi_filter": "<parameter set name or JSON file>"}` | `mf` |
| Scatter correction | `l1`, `l2`, `max`, `snv`, `msc`, `detrend` | `{"scatter_correction": ["snv", "detrend"]}` | `snv`, `dt`, `snvdt` … |
| Derivative | `derivative` (1st order) | `{"derivative": 2}` | `d1`, `d2` |
| Variable scaling | `meancentring`, `autoscaling`, `paretoscaling`, `poissonscaling` | `{"scaling": "autoscaling"}` | `mc`, `as`, `ps`, `poi` |
| Variance threshold | — | `{"variance_threshold": {"threshold": 0.03}}` | `vt03` |
| Ward clustering | — | `{"ward_clustering": {"n_clusters": 40, "connectivity": true}}` | `wc40` |
| PCA | `pca` (5 components) | `{"pca": 5}` or `{"pca": {"n_components": 5}}` | `pca5` |

- The filter object also takes the settings inline, e.g.
  `{"filter": {"savitzky-golay": {"window_length": 7, "polyorder": 2, "deriv": 1}}}`.
- Variance threshold and Ward clustering have no defaults: `threshold` and `n_clusters` must be given.
  Find them with the interactive cells [2.5.1 and 2.5.2][pp1_bands] first. The variance threshold is always
  computed on minmax-scaled bands; a `scaler` other than `minmax` is an error. Ward `connectivity` defaults
  to `true`. Ward clustering also accepts the path of a settings file with `n_clusters`.
- An appended derivative (`d1app`) cannot be part of a chain.

The same syntax is used for `chemometrics_array` in the spectra plots ([Plot data][plot_data]).

## 2.7.1 Simple pre-processing chain

**Process file**: `project_neospectra/process/ml_preprocess/chemometric_default_chain.json`

```json
{
  "process": [
    {
      "process": "chemometric_chain",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "chemometrics_array": ["snv", "derivative", "meancentring", "pca"]
      }
    }
  ]
}
```

Four method names with default settings: SNV, first derivative, meancentring and PCA with 5 components. Every
step is plotted under the one before:

![Simple chain: snv, derivative, meancentring, PCA]({{ "/assets/media/ml_preprocesses_separate/2-7-1-simple-chain.png" | relative_url }})

The plot shows what each step does to the spectra. SNV removes the brightness differences. The derivative
leaves the absorption features as swings around zero. Meancentring shows how each sample differs from the
average derivative. After these steps the PCA scores spread over more components than the PCA of raw
spectra (compare [2.6.1][pp1_pca]).

```
    Accept and save snv.d1.mc.pca5 result? [y/n]:
```

`y` saves `1350-2500_10.snv.d1.mc.pca5`.

suffix: the suffixes of the steps, e.g. `.snv.d1.mc.pca5` (+ optional `#<name_extension>`)

## 2.7.2 Complete pre-processing pipeline

**Process file**: `project_neospectra/process/ml_preprocess/chemometric_pipeline_snv+sgd1+mc+wc+pca.json`

```json
{
  "process": [
    {
      "process": "chemometric_chain",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "chemometrics_array": [
          "snv",
          {"filter": "./project_neospectra/filter/savitzky-golay_d1.json"},
          "meancentring",
          {"ward_clustering": {"n_clusters": 40, "connectivity": true}},
          {"pca": 5}
        ]
      }
    }
  ]
}
```

A complete pipeline in the [recommended order][order]:

```
raw → scatter correction (snv) → Savitzky-Golay derivative (window 5, D1) → meancentring → Ward clustering (40) → PCA (5)
```

![Complete pipeline]({{ "/assets/media/ml_preprocesses_separate/2-7-2-complete-pipeline.png" | relative_url }})

Compared with 2.7.1, the Savitzky-Golay derivative is smoother than the plain derivative. Ward clustering
reduces the meancentred derivative to 40 cluster means (x-axis = cluster number) and keeps the swings of
the absorption features.

suffix: `.snv.sg5.d1.mc.wc40.pca5`

## 2.7.3 Load existing pre-processing chain

**Process file**: `project_neospectra/process/ml_preprocess/chemometric_pipeline_load.json`

```json
{
  "process": [
    {
      "process": "chemometric_chain",
      "parameters": {
        "project_root_fp": "./project_neospectra/data/spectra/neospectra/ds2/ai4sh",
        "dataframe": "raw",
        "chain_from": "1350-2500_10.snv.sg5.d1.mc.wc40.pca5",
        "name_extension": "rerun"
      }
    }
  ]
}
```

Instead of writing a chain, point `chain_from` at a saved pre-processed dataframe. The chain recorded in
its companion JSON is rebuilt with the recorded settings and run on the input dataframe. Give
`chain_from` as:

- the name of a dataframe in `project_root_fp` (`.json` optional), e.g.
  `1350-2500_10.snv.sg5.d1.mc.wc40.pca5`;
- or a path relative to the project root, to use the chain of another dataset, e.g.
  `./project_neospectra/data/spectra/neospectra/ds2/ai4sh/1350-2500_10.snv.sg5.d1.mc.wc40.pca5.json`.

Give either `chain_from` or `chemometrics_array`, not both.

**Settings, not fitted results.** The recorded settings are reused: the filter settings themselves (not the
file, which may have changed), the scatter corrections, the variance threshold and the applied number of
Ward clusters. Every step is **refitted on the new input**, so the kept bands, the cluster borders, the
scaling means and the PCA loadings are computed from the new dataframe. This is right for building a new
training dataset with a proven pipeline. To predict independent samples, use the saved model instead
([Predict and register][ml_predict]). It applies the chain as fitted on the training samples.

**Same spectral range and resolution.** The input must have the same band range and band width as the
dataset the chain was made for (`begin_wavelength`, `end_wavelength`, `output_bandwidth` in the companion
JSON). Otherwise the process stops before running, e.g. a 5 nm input against a 10 nm chain:

```
    ERROR: the input has the bands 1350-2500 nm at 5 nm, the chain was made for 1350-2500 nm at 10 nm
```

The example reruns the chain of 2.7.2 on the same raw dataframe. `"name_extension": "rerun"` keeps the
result apart from the original: it is saved as `1350-2500_10.snv.sg5.d1.mc.wc40.pca5#rerun`.

### From exploration to the model

`chain_from` is how an explored pipeline reaches the model. The model needs indicators and spectra in one
compiled dataset, and it refits the chain on its training samples starting from the unprocessed source in
the same folder (see [Exploration versus modelling][explore]). A chain run in the spectra folder was fitted
on all samples there and points to a source in another folder, so it cannot be refitted. `compile_dataset`
therefore refuses a pre-processed input:

```
    ❌ ERROR: <input> is pre-processed (spectra_scatter_correction → …). Compile the unprocessed selection and pre-process the compiled dataset, e.g. chemometric_chain with chain_from = "<input path>.json"
```

The workflow is:

1. Explore and choose the pipeline in the spectra folder (sections 2.1–2.7).
2. Compile the unprocessed spectra and indicator selections ([Compile the project dataset][compile]).
3. Run `chemometric_chain` with `project_root_fp` set to the compiled folder and `chain_from` set to the
   companion JSON of the explored chain, as a path relative to the project root.
4. Model the result ([ML modeling][ml_model]).

suffix: the suffixes of the loaded chain, e.g. `.snv.sg5.d1.mc.wc40.pca5` (+ optional `#<name_extension>`)

## Next step

[Delete dataframes][pp1_delete]: remove the intermediate dataframes you no longer need.

[pp1]: {{ site.baseurl }}/machine_learning/ml_preprocess1/
[explore]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#exploration-versus-modelling
[order]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#recommended-order
[overwrite]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#overwrite
[companion]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#the-companion-json--the-recipe
[name_extension]: {{ site.baseurl }}/machine_learning/ml_preprocess1/#name-extension
[filter_presets]: {{ site.baseurl }}/machine_learning/ml_preprocess1/filter/#filter-settings
[pp1_bands]: {{ site.baseurl }}/machine_learning/ml_preprocess1/bands/
[pp1_pca]: {{ site.baseurl }}/machine_learning/ml_preprocess1/pca/
[pp1_delete]: {{ site.baseurl }}/machine_learning/ml_preprocess1/delete/
[plot_data]: {{ site.baseurl }}/machine_learning/plot_data/
[ml_predict]: {{ site.baseurl }}/machine_learning/ml_predict/
[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[compile]: {{ site.baseurl }}/machine_learning/parquet_files/#compile-the-project-dataset
