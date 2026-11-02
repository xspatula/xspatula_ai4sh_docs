---
title: "ML Pre-processing 1"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Machine learning pre-processes that work on indicators or spectra separately: what they are for, how to run them, how outputs are named, what the companion JSON and .previous_dataframe record, and the recommended order."
permalink: /machine_learning/ml_preprocess1/
author_profile: false
date: 2026-10-30 08:00:00 +0100
last_modified_at: 2026-11-02 08:00:00 +0100
---

ML pre-processing 1 cleans up indicators and spectra **separately**, before they are compiled into a
modelling dataset. These pre-processes remove indicator outliers and physical artefacts, instrument noise and
baseline shifts from the spectra. Some also compress the spectra into fewer, more informative columns.
Neither the target indicator nor the regressor plays any part, which makes these pre-processes fast and
suited to exploring options.

**ML pre-processing 2** (to be documented) works on the compiled dataset. It can use the relation between
spectra and target indicator, and the regressor (univariate, permutation and recursive feature
elimination selection). It is the notebook `ml_preprocess_compiled`.

## Notebook

**Path**: `xspatula_ai4sh/ai4sh/project_neospectra/ml_preprocess_separated.ipynb`

Set `scheme_file = '../scheme_ai4sh.json'` in the setup cell (the default). Every other cell runs one process
file from `project_neospectra/process/ml_preprocess/`. The section numbers on these pages are the cell
numbers of the notebook. Each pre-process adds a unique suffix to the saved dataframe, and a pipeline of pre-processes stacks the sequential suffixes separated by dots.

| Section | Page | Process | Suffix |
|---|---|---|---|
| 1.1 Indicator outliers | [Indicator outliers][pp1_outliers] | `detect_outliers` | `.olif`, `.olee`, `.ollof`, `.ol1csvm` |
| 2.1 Filters | [Filters][pp1_filter] | `filter_spectra`, `multi_filter_spectra` | `.ma5`, `.gf2`, `.sg5`, `.lw21`, `.mf` |
| 2.2 Scatter correction | [Scatter correction][pp1_scatter] | `spectra_scatter_correction` | `.l1`, `.l2`, `.max`, `.snv`, `.msc`, `.dt` |
| 2.3 Derivatives | [Derivatives][pp1_derivative] | `spectra_derivative`, `filter_spectra` | `.d1`, `.d2`, `.sg5.d1` |
| 2.4 Variable scaling | [Variable scaling][pp1_scaling] | `spectra_scaling` | `.mc`, `.as`, `.ps`, `.poi` |
| 2.5 Band reduction | [Band reduction][pp1_bands] | `select_variance_threshold`, `ward_clustering` | `.vt03`, `.wc40` |
| 2.6 Decomposition | [PCA][pp1_pca] | `spectra_decomposition` | `.pca5` |
| 2.7 Chemometric chain | [Chemometric chain][pp1_chain] | `chemometric_chain` | the suffixes of the steps |
| 3 Delete dataframes | [Delete dataframes][pp1_delete] | `delete_dataframes` | — |

## Per spectrum or per band

Two of the pre-processes rescale the spectra, and they work in different directions of the data table
(rows = samples, columns = bands). Keep the terms apart:

| | Works on | Statistics from | Removes |
|---|---|---|---|
| **Scatter correction** (2.2) | each spectrum (row) on its own | that spectrum only | offset and slope of a spectrum (light scattering, particle size, packing) |
| **Variable scaling** (2.4) | each band (column) across all samples | all samples of that band | the band's mean level, and optionally its spread |

A scatter correction of one sample does not depend on the other samples (except `msc`, which uses the mean
spectrum as reference). A variable scaling changes if you add or remove samples.

## The input dataframe

All spectral processes read their input from the folder `project_root_fp`, here
`./project_neospectra/data/spectra/neospectra/ds2/ai4sh` (the selection of
[Select data][select_data]). The parameter `dataframe` chooses the file:

| `dataframe` | Input |
|---|---|
| `raw` (default) | The selected dataset of the folder (e.g. `1350-2500_10.parquet`), never a pre-processed file |
| `previous` | The dataframe saved last, i.e. the latest record of `.previous_dataframe` |
| a file name | A parquet file in `project_root_fp`, `.parquet` optional, e.g. `1350-2500_10.snv` |

A file counts as pre-processed when its name has a known suffix, or when its companion JSON has a non-empty
`preprocessing_chain`. To build a pipeline step by step, give each process the output of the step before
(or `previous`). The [chemometric chain][pp1_chain] does all steps in one go.

## Three ways to run a pre-process

The notebook has a cell for each way where it applies:

1. **Select**: all methods of one kind are applied and plotted together. You choose one by number at the
   prompt (`0 = none`), and the chosen result is saved without a further y/n question. Used for outlier
   detectors (`"detector": "select"`), filters (`"filter_fp": "select"`), scatter corrections
   (`"scaler": ["select"]`) and variable scalings (`"method": "select"`).
2. **Compare inputs**: one method is applied to several input dataframes (`dataframe_array`, two or more
   entries) and the results are plotted together. You choose which input's result to save. Use it to see
   how earlier pre-processing changes a step (derivative 2.3.3, PCA 2.6.2).
3. **Apply predefined**: one method with the settings in the process file. You get a plot of input and
   output, then the prompt `Accept and save <suffix> result? [y/n]`.

All prompts are answered in the notebook's input field. The plots show at most 100 spectra (the annotation
`n=<samples> shown=<drawn>` gives the number of samples and the number drawn).

## Output names

Every pre-process saves `<input stem>.<suffix>.parquet` and a companion `<input stem>.<suffix>.json` next to
its input. The suffixes are separated by a dot, so the file name spells out the pipeline:

```
1350-2500_10.parquet                         the selected (raw) dataset
1350-2500_10.snv.parquet                     + scatter correction (snv)
1350-2500_10.snv.sg5.d1.parquet              + Savitzky-Golay first derivative, window 5
1350-2500_10.snv.sg5.d1.mc.parquet           + meancentring
1350-2500_10.snv.sg5.d1.mc.wc40.pca5.parquet + Ward clustering (40) + PCA (5 components)
```

Suffixes carry the setting that tells two outputs apart: a number in a suffix is written with `p` instead
of a decimal point (sigma 1.5 → `gf1p5`), and a leading `0.` is dropped (threshold 0.028 → `vt028`).
Chained scatter corrections are joined without a dot (`snv` + `detrend` → `snvdt`).

| Suffix | Method | Category |
|---|---|---|
| `olif`, `olee`, `ollof`, `ol1csvm` | Isolation forest, elliptic envelope, local outlier factor, one-class SVM | Indicator outliers |
| `ma{n}` | Moving average, n = kernel length (`ma5`) | Filter |
| `gf{n}` | Gaussian filter, n = sigma (`gf2`, `gf1p5`) | Filter |
| `sg{n}` | Savitzky-Golay smoothing, n = window length (`sg7`) | Filter |
| `sg{n}.d1`, `sg{n}.d2` | Savitzky-Golay derivative (deriv 1 or 2) | Filter / derivative |
| `lw{n}` | LOWESS, n = number of points (`lw21`) | Filter |
| `mf` | Multi-filter (region-wise) | Filter |
| `l1`, `l2`, `max` | L1, L2 and max normalisation | Scatter correction |
| `snv`, `msc` | Standard Normal Variate, Multiplicative Scatter Correction | Scatter correction |
| `dt` | Detrending (2nd-order polynomial baseline removed) | Scatter correction |
| `snvdt`, `snvmsc` … | Scatter corrections in sequence | Scatter correction |
| `d1`, `d2` … | Finite-difference derivative of order 1, 2 … | Derivative |
| `d1app` … | Derivative appended to the original bands | Derivative |
| `mc`, `as`, `ps`, `poi` | Meancentring, autoscaling, Pareto scaling, Poisson scaling | Variable scaling |
| `vt{n}` | Variance threshold on minmax-scaled bands, n = threshold (`vt03`, `vt028`) | Band reduction |
| `wc{n}` | Ward clustering, n = number of clusters (`wc40`) | Band reduction |
| `pca{n}` | PCA, n = number of components (`pca5`) | Decomposition |

The full table, including the suffixes of ML pre-processing 2, is
`xspatula_ai4sh/ai4sh/default/chemometric/chemometric_abbreviations.md`.

### Name extension

Some settings do not show in the suffix, e.g. the kernel weights of a moving average. Outlier detection,
filter, multi-filter, variance threshold, Ward clustering and the chemometric chain therefore accept
`name_extension`: 1-6 letters, digits or `-`, added after a `#`. A moving average with the kernel
`[1, 2, 3, 2, 1]` and `"name_extension": "12321"` is saved as `1350-2500_10.ma5#12321` (see
[2.1.5][pp1_ma_ext]).

### Overwrite

A pre-process does not recompute an output that already exists. It prints

```
"snv.sg5.d1.mc.wc40.pca5" already applied. Use overwrite=True to rerun.
```

and registers the existing file as previous, so the next step with `"dataframe": "previous"` still finds it.
To recompute, add `"overwrite": true` to the process block, next to `"parameters"`:

```json
{
  "process": [
    {
      "process": "select_variance_threshold",
      "overwrite": true,
      "parameters": { ... }
    }
  ]
}
```

## The companion JSON — the recipe

The JSON next to each parquet holds the metadata of the selected dataset (dataset, provision, preparation,
wavelength range, bandwidth …), `source_parquet` (the raw dataset the chain starts from),
`preprocessing_chain` (one entry per step with its suffix `id`, process and parameters, plus the full
settings where a file was read) and `output_data` (the column format). Excerpt of
`1350-2500_10.snv.sg5.d1.mc.wc40.pca5.json`:

```json
{
  "output_name": "1350-2500_10",
  "provision_tag": "neospectra",
  "preparation_tag": "ds2",
  "begin_wavelength": 1350,
  "end_wavelength": 2500,
  "output_bandwidth": 10,
  "source_parquet": "1350-2500_10.parquet",
  "preprocessing_chain": [
    {"id": "snv", "step": 1, "process": "spectra_scatter_correction",
     "parameters": {"scaler": ["snv"]}},
    {"id": "sg5.d1", "step": 2, "process": "filter_spectra",
     "parameters": {"filter_fp": "./project_neospectra/filter/savitzky-golay_d1.json"},
     "filter": {"savitzky-golay": {"window_length": 5, "polyorder": 2, "deriv": 1, "mode": "nearest"}}},
    {"id": "mc", "step": 3, "process": "spectra_scaling",
     "parameters": {"method": "meancentring"}},
    {"id": "wc40", "step": 4, "process": "ward_clustering",
     "parameters": {"ward_clustering_fp": "inline", "connectivity": true,
                    "n_clusters_tuned_best": null, "n_clusters_selected": 40, "n_clusters_applied": 40},
     "cluster_bands": {"wc1": [1350, 1360], "wc2": [1370, 1380], "wc3": [1390, 1400], "...": "..."}},
    {"id": "pca5", "step": 5, "process": "spectra_decomposition",
     "parameters": {"method": "pca", "n_components": 5}}
  ],
  "output_data": {
    "data_type": "pca",
    "column_format": "pc{n} — n is component number starting at 1",
    "spectral_array": [1, 2, 3, 4, 5],
    "signal_type": "refl"
  }
}
```

The filter entry holds the filter settings themselves, not only the file name. The chain therefore still
works if the filter file is later edited or deleted.

## .previous_dataframe — the run log

The hidden file `.previous_dataframe` in `project_root_fp` is the history of saved dataframes: one record
per run, with the process, time, saved entries and provenance. The provenance holds the process parameters,
every settings file read (with its sha256 and content) and the software versions. Each dataframe is listed
once, at its latest position: an overwritten or re-registered file moves to the end. `configs` lists only
settings *files* (and database parameter sets) that were read. Settings given inline in the process file are
under `parameters`. See also [The previous dataframe][previous_dataframe].

In short, `.previous_dataframe` records what was run, and the companion JSON records how one dataframe was
made.

## Exploration versus modelling

The notebook fits every step on all samples: it is for exploring. When a model is trained on a
pre-processed dataframe ([ML modeling][ml_model]), the model does not use the saved numbers. It reads the
companion JSON, starts from `source_parquet` and **refits the whole `preprocessing_chain` on the training
samples only**, inside the model pipeline. The test samples then never influence the pre-processing (no
leakage).

The model needs the indicators and spectra in one dataframe, i.e. the compiled dataset
([Compile the project dataset][compile]), and it can only refit a chain from an unprocessed source in the
same folder. `compile_dataset` therefore refuses spectra that are pre-processed (any step other than
indicator outlier removal). The workflow is: explore and choose the pipeline in the spectra folder, compile
the unprocessed selections, run the chosen pipeline in the compiled folder with [chain_from][chain_from],
and model the result. Two consequences:

- Intermediate files are not needed for modelling: only the final dataframe and its companion JSON (see
  [Delete dataframes][pp1_delete]).
- Fitted results (msc reference, scaling means, kept bands, clusters, PCA loadings) can differ slightly
  between the exploration run and the model.

Indicator outlier removal is the exception. It only sets values to missing, so an outlier-cleaned indicator
selection can be compiled; its chain is recorded per input in the compiled companion JSON. The model notes
that the outliers were removed using all samples.

## Recommended order

```
raw → filter → scatter correction → derivative → variable scaling → band reduction → PCA
```

- Scatter corrections (snv, msc) model the offset and slope of a whole spectrum. They need complete,
  unscaled spectra: apply them before variable scaling and band reduction.
- Never apply a scatter correction after a derivative. The derivative has already removed the baseline and
  slope that the scatter correction assumes.
- Derivatives need neighbouring bands: apply them before band reduction. After Ward clustering the columns
  are clusters, not wavelengths.
- A Savitzky-Golay derivative replaces both the filter and the derivative step and goes after scatter
  correction.
- Meancentring before PCA changes nothing, because PCA centres the data itself. It does matter before Ward
  clustering and for models without an intercept.

The order follows Rinnan, van den Berg & Engelsen (2009), *Review of the most common pre-processing
techniques for near-infrared spectra*, TrAC Trends in Analytical Chemistry 28(10), 1201–1222.

Not every pipeline needs every step. A common, robust NIR pipeline is
`snv → sg5.d1 → mc`, optionally followed by band reduction or PCA ([Chemometric chain][pp1_chain]).

## Next step

[Indicator outliers][pp1_outliers]: detect and remove outlying indicator values.

[select_data]: {{ site.baseurl }}/machine_learning/select_data/
[previous_dataframe]: {{ site.baseurl }}/machine_learning/parquet_files/#the-previous-dataframe-previous_dataframe
[ml_model]: {{ site.baseurl }}/machine_learning/ml_model/
[compile]: {{ site.baseurl }}/machine_learning/parquet_files/#compile-the-project-dataset
[chain_from]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/#273-load-existing-pre-processing-chain
[pp1_outliers]: {{ site.baseurl }}/machine_learning/ml_preprocess1/outliers/
[pp1_filter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/filter/
[pp1_scatter]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scatter/
[pp1_derivative]: {{ site.baseurl }}/machine_learning/ml_preprocess1/derivative/
[pp1_scaling]: {{ site.baseurl }}/machine_learning/ml_preprocess1/scaling/
[pp1_bands]: {{ site.baseurl }}/machine_learning/ml_preprocess1/bands/
[pp1_pca]: {{ site.baseurl }}/machine_learning/ml_preprocess1/pca/
[pp1_chain]: {{ site.baseurl }}/machine_learning/ml_preprocess1/chain/
[pp1_delete]: {{ site.baseurl }}/machine_learning/ml_preprocess1/delete/
[pp1_ma_ext]: {{ site.baseurl }}/machine_learning/ml_preprocess1/filter/#215-moving-average-with-name-extension
