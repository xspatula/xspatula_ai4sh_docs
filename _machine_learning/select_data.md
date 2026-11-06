---
title: "Select Data"
layout: single
sidebar:
  nav: "machine_learning"
excerpt: "Selecting AI4SoilHealth spectra, indicators and covariates from the database for machine learning: scope, sample counts with identify_indicator, class filters on the sample setting, and reusing a selection's samples."
permalink: /machine_learning/select_data/
author_profile: false
date: 2026-10-19 08:00:00 +0200
last_modified_at: 2026-11-05 18:00:00 +0100
---

The first step of the machine learning pipeline: decide which samples and which data you need, check
that there are enough of them, and pull them out of the database onto local disk. This page covers the
selection. [Parquet files][parquet_files] covers what is saved, and [Plot data][plot_data] how to look at
it.

Spectra, indicators (the targets you want to predict) and covariates are selected **separately**, each
into its own tag-named Parquet file. A compile step then joins them on `sample_id` into the dataset that
every later ML step works on. Alternatively, `compile_dataset_from_db` selects them in one call and saves the
compiled dataset directly ([Compile directly from the database][compile_from_db]).

The code is the same as for LUCAS. The [LUCAS Select Data][lucas_select] page is the extended version of
this page. It covers texture, land cover, land use and biogeographic region filtering in depth, because
LUCAS has that data and AI4SH (so far) does not.

The extracted files contain AI4SH data and may only be shared within the AI4SoilHealth partners (see
[AI4SH data: partners only][partners_only]).


## Workflow

One notebook cell per step:

1. **Identify** (process `identify_indicator`): how many samples there are, per class filter entry,
   spectral sensor and indicator. Nothing is saved.
2. **Select** (process `select_indicator_only`): saves a `.parquet` + `.json`. See
   [Parquet files][parquet_files].
3. **Inspect** (process `inspect_pandas_dataset`): shows what was just saved. See
   [Inspect the dataset][inspect].
4. **Plot** (process `plot_indicators`). See [Plot data][plot_data].

Spectra and covariates follow the same pattern with `select_spectra_only` and `select_covariate_only`.
The LUCAS notebook `select_indicator_only.ipynb` is the model for the AI4SH notebook still to be written.

## The select processes

One function does all selections; the process sets the **role** of the output (`SELECT_ROLE_D` in
`src/ai4sh/select.py`):

| Process | Role | Saves |
|---|---|---|
| `identify_indicator` | — | prints sample counts per indicator, spectra sensor and filter class; saves nothing |
| `select_spectra_only` | `spectra` | spectra only |
| `select_indicator_only` | `indicator` | numeric (and nominal) indicators |
| `select_covariate_only` | `covariate` | covariates |
| `compile_dataset_from_db` | `compiled` | indicators, spectra and/or covariates in one call, saved as a compiled dataset, see [Compile directly from the database][compile_from_db] |
| `compile_dataset` | `compiled` | joins saved selections on `sample_id`, see [Compile][compile] |

The process decides the role, not the data: pH can be an indicator in one file and a covariate in
another. Whether a name is a numeric measurement, a nominal classification or a landscape class is
looked up in the database.

## Scope

All select processes take the same scope parameters:

| Parameter | Description |
|---|---|
| `dataset` | Dataset name, tag or alias, e.g. `ai4sh` |
| `campaign` | Optional: one campaign (the site), e.g. `ai4sh_dk_foulum` |
| `sampling_log` | Optional: one sampling log (the survey round), e.g. `ai4sh_dk_foulum_2024` |
| `sample_array` | Optional: a list of sample names; the output sub path then ends with `sample_select` |
| `min_profile` / `max_profile` | Depth profile filter, cm |

Each can be given as name, tag or alias. If a value matches more than one record, the process stops and
asks for the tag, or the parent (dataset/campaign). See [Automatic naming and tags][auto_naming].

Each process skips silently if its output already exists, unless `overwrite: true`.

### Campaigns and sampling logs

The `ai4sh` dataset spans many sites. Each site is a **campaign**, with one or more **sampling logs**
(survey rounds), for example:

| Campaign | Sampling log |
|---|---|
| `ai4sh_dk_foulum` | `ai4sh_dk_foulum_2024` |
| `ai4sh_fi_jokioinen` | `ai4sh_fi_jokioinen_2024` |
| `ai4sh_gr_ktima-gerovassiliou` | `ai4sh_gr_ktima-gerovassiliou_2023` |
| `ai4sh_gr_zazari` | `ai4sh_gr_zazari_2025` |
| `ai4sh_hr_neretva` | `ai4sh_hr_neretva_2024`, `ai4sh_hr_neretva_2025` |

`identify_indicator` (below) gives the number of samples in a campaign or sampling log.

Set `campaign` or `sampling_log` to model one site or one round. Leave both empty to pool all sites.
Every saved parquet carries `campaign_tag` and `sampling_log_tag`, so a pooled model can still be
evaluated and plotted per site later. LUCAS has one campaign; for AI4SH this is the main split.

## Observation logs, subsamples and replicates

`select_spectra_only`, `select_indicator_only`, `select_covariate_only` and `compile_dataset_from_db` also take:

| Parameter | Default | Description |
|---|---|---|
| `aggregate` | `average` | How several observations of the same sample *within one observation log* (subsamples and/or replicates) are combined into one value or spectrum: `average`, `median`, `min` or `max` (per band for spectra) |
| `observation_log_array` | empty | The observation logs (names or tags) to take the data from, per kind of data; empty = all |

Aggregation is reported in the log, e.g.:

```
    Aggregating (average) <n> subsample/replicate spectra of <m> samples
    Aggregating (average) subsample/replicate values of <n> sample/indicator pairs
```

**One observation log per sample.** If a sample's spectra, or a sample's value of one indicator, come from
more than one observation log, the selection stops and lists the logs. Typical causes are rescans in
different years, ds2 and ilmx scans when `spectra_preparation_name` is not set, or two laboratories
analysing the same indicator:

```
    ❌ ERROR: <n> samples have data from more than one observation log:
        <log name> (tag <tag>): <n> samples
        <log name> (tag <tag>): <n> samples
    Set observation_log_array to the names or tags of the observation logs to use (and spectra_preparation_name to
    separate sample preparations, e.g. ds2 and ilmx).
```

**Per kind of data, and per indicator.** A listed log restricts only the data it holds: spectral logs restrict
the spectra, laboratory and field logs the indicators, covariates and classes (e.g. soil texture). An indicator or
covariate without a record in any listed log keeps all its logs, with a note naming them:

```
    Note: observation_log_array names no observation log with indicator/covariate data for clay — all its logs used (<logs>)
```

When no listed log holds that kind of data at all, there is one note without `for <name>`. So listing only the
spectral logs, as in the [compile from the database][compile_from_db] example, selects those spectra and every
laboratory value; listing the laboratory log of the indicators keeps a covariate that is held only in a field
log. Classes are checked like numeric values: a sample with a class from two logs stops the selection
(`… sample/class pairs have data from more than one observation log`).

Values of different indicators may come from different logs, e.g. c-org from one laboratory and pH from
another. Observation log tags may repeat across sampling logs, so one tag selects the matching log in every
sampling log. The companion JSON records `observation_log_array` and `aggregate`.

**Paired datasets.** To get, e.g., ds2 and ilmx spectra of the same samples (to test ilmx → ds2
translations), run `select_spectra_only` twice with different `spectra_preparation_name` (or
`observation_log_array`), the second with [`sample_source`](#reuse-a-selections-samples-sample_source)
set to the first output. The two outputs get different sub paths (the preparation alias), the same samples
and identical columns, provided the wavelength grid parameters are the same.

## Identify indicators and sample counts

Before saving anything, check how many samples you would get. `identify_indicator` takes the scope
parameters, every class filter (below), `sample_source` and `min_samples`:

```json
{
  "process": [
    {
      "process": "identify_indicator",
      "overwrite": false,
      "parameters": {
        "dataset": "ai4sh",
        "campaign": "",
        "sampling_log": "",
        "min_profile": 0,
        "max_profile": 100,
        "texture_class_array": [],
        "texture_classification": "soil texture usda",
        "landuse_array": [],
        "landcover_array": [],
        "biogeo16_array": [],
        "juxtaposition_array": [],
        "composition_array": [],
        "proximity_array": [],
        "sample_source": "",
        "min_samples": 30
      }
    }
  ]
}
```

`min_samples` (default 30) flags every count below it with ⚠️. The output lists the samples in scope, the
spectral sensors and the indicators (list shortened):

```
    Querying available data for dataset "ai4soilhealth"

    Samples in scope: <n>

    Spectral sensors (provisions):
        - bruker invenio s ftir: <n> samples
        - foss micral: <n> samples
        - foss nirs ds2500 l: <n> samples
        - proxiscout: <n> samples

    Indicators (samples with a value):
        - aggregate stability (aggstab) [index]: <n>  ⚠️  < 30
        - calcium ion (ca2+) [cmol*kg^-1]: <n>
        - cation exchange capacity (cec) [cmol*kg^-1]: <n>
        - clay (<0.002 mm) (clay) [weight percent]: <n>
        ...
```

- Counts are samples with a value. Missing-value codes (-9999, -999, -888) are not counted. The range
  screening ([Invalid values][invalid_values]) can lower the saved count further.
- Nominal indicators get the unit `class`.
- The depth filter (`min_profile`/`max_profile`) applies here too.

## Class filters

A model can be general (all samples) or specific (one model per class, e.g. per field position).
Splitting at selection means each class gets its own preprocessing (indicator transformation, spectral
enhancement); splitting only at the modelling stage loses that. The parquet keeps the class
([Columns][columns]), so a general model can still be coloured or split by class later.

| Parameter | Filters on | Column in parquet | Name part |
|---|---|---|---|
| `juxtaposition_array` | where in the field the sample sits | `class_juxtaposition` | `jx-` |
| `composition_array` | individual or composite sample | `class_composition` | `cp-` |
| `proximity_array` | centre or periphery of the sampled object | `class_proximity` | `px-` |
| `texture_class_array` (+ `texture_classification`) | soil texture class of the sample | `class_texture` | `tx-` |
| `landcover_array` | land cover at the sample position | `class_landcover` | `lc-` |
| `landuse_array` | land use at the sample position | `class_landuse` | `lu-` |
| `biogeo16_array` | biogeographic region at the sample position | `class_biogeo16` | `bg-` |

Common rules:

- Each filter is a list. A sample passes if its class is any entry in the list (OR within a filter).
  Several filters combine with AND.
- An entry is a class name, tag, alias or display name, case-insensitive.
- An unknown entry stops the process and lists the valid classes. Two entries naming the same class stop
  it too.
- Samples without a recorded class are excluded by that filter.
- Filters restrict **every** query (keys, spectra, indicators, covariates). A filtered spectra selection
  and a filtered indicator selection therefore start from the same samples.

What the AI4SH database holds for each filter:

| Filter | AI4SH data |
|---|---|
| `juxtaposition_array` | setting system "agricultural field": uniform, furrow, plant row |
| `composition_array` | composite, individual |
| `proximity_array` | general, central, peripheral |
| `texture_class_array` | derived on load from clay, silt and sand (observation cell, e.g. 3.2.1) |
| `landcover_array`, `landuse_array`, `biogeo16_array` | no landscape observations imported |

So for AI4SH the sample setting filters and texture are the ones that work.

### Sample setting

Every sample is registered with three setting classes (see [Manage samples][manage_sample]). The
classes are defined in `observation_utility`:

- **Juxtaposition**: where the sample sits relative to a structure in the field. Defined per setting
  system (agricultural field, agroforestry field, forest patch): `uniform` (a uniform area with constant
  surface conditions), `furrow` (path between planted rows), `plant row` (in the plant row), `edge`,
  `center`. A name that exists under several setting systems (e.g. `furrow`) matches all of them, so you
  don't name the setting system.
- **Composition**: `individual` (one core) or `composite` (collected from several subsamples).
- **Proximity**: `central` (the geometric centre of the sampled object), `peripheral` (its periphery) or
  `general` (both).

Furrow and plant row samples:

```json
"juxtaposition_array": ["furrow", "plant row"]
```

```
    Filter juxtaposition: ['furrow', 'plant row']

    Samples in scope: <n>

    Samples per class_juxtaposition:
        - furrow: <n>
        - plant row: <n>  ⚠️  < 30

    Spectral sensors (provisions):
        - bruker invenio s ftir: <n> samples
        ...
```

Individual samples, centre and periphery:

```json
"composition_array": ["individual"],
"proximity_array": ["central", "peripheral"]
```

```
    Filter composition: ['individual']
    Filter proximity: ['central', 'peripheral']

    Samples in scope: <n>

    Samples per class_composition:
        - individual: <n>

    Samples per class_proximity:
        - central: <n>  ⚠️  < 30
        - peripheral: <n>
```

In the current data every individual sample is central or peripheral, and every composite sample is
general. `composition` and `proximity` thus split the data the same way; filtering on both adds nothing.

### When the selection is too narrow

The counts exist to catch this. A narrow filter, such as furrow and plant row, can leave some indicators
(e.g. microbial and eDNA-derived ones) and some classes below `min_samples`, flagged ⚠️. A class that thin
is too small for a model of its own. Broaden the selection: pool the classes into one model, drop the
filter, or drop the indicators that fall below `min_samples`.

### Texture

`texture_class_array` keeps samples of the listed soil texture classes, under the classification in
`texture_classification` (default `soil texture usda`, or `soil texture iuss` and the two major-group
indicators). The classes are derived from the measured clay, silt and sand while the observations load,
see [Derived soil texture classes][obs_texture]. See [Soil texture][lucas_texture] on the LUCAS page for the full explanation of the filter.

### Land cover, land use and biogeographic region

`landcover_array`, `landuse_array` and `biogeo16_array` keep samples whose position has the listed
class. Land cover and land use are hierarchical: an entry at any rank includes everything below it. No
landscape observations have been imported for AI4SH, so these filters select nothing. See
[Land cover and land use][lucas_landcover] on the LUCAS page.

## Reuse a selection's samples (`sample_source`)

To get the spectra for exactly the samples of a filtered indicator selection:

- Set `sample_source` to a saved parquet (relative to the project root, `.parquet` optional).
- Its `sample_id`s restrict the scope. Its `class_*` columns and `sample_filter` are inherited, so the
  spectra file gets the same filter part in its name.
- Don't repeat the filter instead. The indicator selection may already have dropped samples (no indicator
  value, also after `data_range`). `sample_source` gives exactly the same samples.

## Select spectra

```json
{
  "process": [
    {
      "process": "select_spectra_only",
      "overwrite": false,
      "parameters": {
        "dataset": "ai4sh",
        "campaign": "",
        "sampling_log": "",
        "min_profile": 0,
        "max_profile": 100,
        "spectra_provision": "foss nirs ds2500 l",
        "spectra_preparation_name": "ds2",
        "begin_wavelength": 400,
        "end_wavelength": 2500,
        "output_bandwidth": 10,
        "output_spectra_signal": "absorb",
        "sample_source": "",
        "output_root_fp": "./project_neospectra/data"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `spectra_provision` | Spectrometer provision (name, tag or alias), one of those listed by `identify_indicator` |
| `spectra_preparation_name` | Sample preparation (e.g. `ds2`, dried and sieved 2 mm) |
| `begin_wavelength` / `end_wavelength` / `output_bandwidth` | Target wavelength grid, resampled from the spectrometer's native wavelengths |
| `output_spectra_signal` | `absorb` converts the stored reflectance to absorbance on the way out |
| `output_root_fp` | Root directory of all selections |

## Select indicators

```json
{
  "process": [
    {
      "process": "select_indicator_only",
      "overwrite": false,
      "parameters": {
        "dataset": "ai4sh",
        "campaign": "",
        "sampling_log": "",
        "min_profile": 0,
        "max_profile": 100,
        "indicator_array": ["cec", "clay"],
        "juxtaposition_array": ["furrow", "plant row"],
        "data_range": "",
        "output_root_fp": "./project_neospectra/data"
      }
    }
  ]
}
```

| Parameter | Description |
|---|---|
| `indicator_array` | The indicators to select, by name or alias |
| `data_range` | Optional JSON string `{"<indicator>": {"min": …, "max": …}}`; a value outside its range is set to missing; empty = no filter |
| `targetfeaturesymbols` | Which unit to save each indicator in, see [Indicator units][indicator_units]: a `plot_symbol` set name (`"name"` or `"name@version"`, see [ML settings][ml_settings]) or a JSON file path. Default `"default"` = the set `targetfeaturesymbols` |

**`data_range`** sets a value outside its range to missing and keeps the other values of the sample, as the
[indicator range][pp1_range] does. A sample without any indicator value is removed, whether it had none from the
start or `data_range` set them all to missing. At `verbose` 1 or more the run prints, e.g.:

```
    Set to missing by data_range: c-org <n>, n-tot <n>
    Excluded, no indicator value: <n> (<n> emptied by data_range)
```

The part in brackets is printed only when `data_range` emptied a sample.

The range is recorded as `data_range` in the companion JSON, not as a pre-processing step. `data_range` works the
same in `select_covariate_only` and `compile_dataset_from_db`.

If a requested indicator has no data, the process lists what is available and asks whether to continue
without it. The output name is explained under [Output names][output_names].

## Select covariates

`select_covariate_only` takes `covariate_array` (and `covariate_rank` for hierarchical landscape
classes). AI4SH has no landscape covariates yet. Any indicator can still be selected as a covariate,
e.g. a field measurement used as model input. See [Covariates and ranks][lucas_ranks] on the LUCAS page.

Soil texture is the main nominal covariate. When observations are imported, texture is derived into four
classifications ([Derived soil texture classes][obs_texture]): `soil texture usda` (12 classes),
`soil texture usda major group` (4 groups), `soil texture iuss` and `soil texture iuss major group`. They are
saved as text with unit `class`.

**Process file**: `project_neospectra/process/select/select_ai4sh_texture.json`

```json
{
  "process": [
    {
      "process": "select_covariate_only",
      "overwrite": false,
      "parameters": {
        "dataset": "ai4sh",
        "campaign": "",
        "sampling_log": "",
        "min_profile": 0,
        "max_profile": 20,
        "covariate_array": ["soil texture usda", "soil texture usda major group"],
        "output_root_fp": "./project_neospectra/data",
        "output_name": "texture-usda_0-20"
      }
    }
  ]
}
```

The selection is then [compiled with the indicators and spectra][compile_covariates], and used in a model as
described under [Covariates][ml_model_covariates].

## Select and compile in one step

`compile_dataset_from_db` takes the spectra parameters plus `indicator_array`, `covariate_array` and `data_range`,
and saves a compiled dataset without separate selections. See
[Compile directly from the database][compile_from_db], also for when to use which route.

## Next step

[Parquet files][parquet_files]: what the selection saved, how it is named, and how to inspect it.

[parquet_files]: {{ site.baseurl }}/machine_learning/parquet_files/
[plot_data]: {{ site.baseurl }}/machine_learning/plot_data/
[inspect]: {{ site.baseurl }}/machine_learning/parquet_files/#inspect-the-dataset
[compile]: {{ site.baseurl }}/machine_learning/compile/
[compile_from_db]: {{ site.baseurl }}/machine_learning/compile/#compile-directly-from-the-database
[pp1_range]: {{ site.baseurl }}/machine_learning/ml_preprocess1/range/
[columns]: {{ site.baseurl }}/machine_learning/parquet_files/#columns
[output_names]: {{ site.baseurl }}/machine_learning/parquet_files/#output-names
[indicator_units]: {{ site.baseurl }}/machine_learning/parquet_files/#indicator-units
[invalid_values]: {{ site.baseurl }}/machine_learning/parquet_files/#invalid-values
[partners_only]: {{ site.baseurl }}/all_data/#ai4sh-data-partners-only
[auto_naming]: {{ site.baseurl }}/setup_process/auto_naming/
[manage_sample]: {{ site.baseurl }}/sample/manage_sample/
[lucas_select]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/select_data/
[lucas_texture]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/select_data/#soil-texture
[lucas_landcover]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/select_data/#land-cover-and-land-use
[lucas_ranks]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/select_data/#covariates-and-ranks
[obs_texture]: {{ site.baseurl }}/observation/#derived-soil-texture-classes
[ml_settings]: {{ site.baseurl }}/machine_learning/ml_settings/
[compile_covariates]: {{ site.baseurl }}/machine_learning/compile/#compile-with-covariates
[ml_model_covariates]: {{ site.baseurl }}/machine_learning/ml_model/#covariates
