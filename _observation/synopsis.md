---
title: "Load Observation Data"
layout: single
sidebar:
  nav: "observation"
  nav2: "loading_data"
excerpt: "Block 3.2 of the loading notebook: all AI4SH observations except spectra and eDNA — wet laboratory, enzymatic activity, infiltration, microbiometer and aggregate stability. Every source is loaded the same way, with an observation log and a table of indicator values."
permalink: /observation/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-10-19 08:00:00 +0200
---

This collection covers block **3.2** of the single loading notebook (see [Load all data][all_data]): all AI4SH observations except spectra and eDNA. Spectra need spectrometer records and signal arrays, and eDNA needs taxa and method pipelines, so they have their own collections ([Load spectral data][spectra], [Load eDNA data][edna]). Everything in block 3.2 is a set of indicator values per sample, and every source is loaded the same way.

```
./ai4sh/import_data/insert_ai4sh_data.ipynb   block 3.2, cells 3.2.1–3.2.9
```

## One pattern for all sources

Each cell runs one process file with two (or more) `insert_tabular_data` entries — see [Insert process][setup_process_insert]:

1. **Observation log** — `manage_observation_log` reads an Excel file with one row per sampling log, linking it to the provision (instrument, kit, app or laboratory) and recording the sample handling (preparation, preservation, storage, transportation). The observation log name is composed automatically as `<sampling log>@<provision>`, see [Automatic naming and tags][auto_naming].
2. **Observations** — `manage_observation` reads an Excel file with one row per sample (or subsample). The identifier columns name the observation log and the sample; every column starting with `@` is an indicator value, e.g. `@ph-h2o`. Each `@` column must match an indicator registered for that provision in `observation_utility.provision_indicator`, see [the `@` prefix](#observation-values-the--prefix).

The observation log must be loaded first, as the observation rows reference it by name. Both are INSERT-only and safe to re-run. The `@` convention and all columns are described in detail for Agrolab on [Translate Agrolab data][translate_agrolab]; the other sources only differ in folder, provision and indicators.

Soil texture classes are derived during any observation load whose provision has texture fractions and registered texture indicators — see [Derived soil texture classes](#derived-soil-texture-classes).

## Cells in block 3.2

{% capture notice-2 %}
| Cell | Source | Provision | Indicators | Page |
|---|---|---|---|---|
| 3.2.1 | Agrolab wet laboratory | `ai4sh-agrolab` | 15 (pH, EC, organic C, total N, cations, CEC, P, texture fractions) | [Insert Agrolab data]({{ site.baseurl }}/observation/insert_agrolab/) |
| 3.2.2–3.2.5 | Veltia, iBEC, SLU, Neiker wet laboratories | — | — | pending, see below |
| 3.2.6 | Digit Soil enzymatic activity | `digit soil sear bob` | 5 enzyme activity rates | [Insert Digit Soil data]({{ site.baseurl }}/observation/insert_digitsoil/) |
| 3.2.7 | BEST infiltration | `best infiltration` | 10 soil hydraulic properties | [Insert BEST infiltration data]({{ site.baseurl }}/observation/insert_best_infiltration/) |
| 3.2.8 | Microbiometer | `microbiometer-classic`, `microbiometer-pro` | microbial carbon (and nitrogen), fungi and bacteria share | [Insert Microbiometer data]({{ site.baseurl }}/observation/insert_microbiometer/) |
| 3.2.9 | Aggregate stability | `slakes` | aggregate stability index | [Insert aggregate stability data]({{ site.baseurl }}/observation/insert_aggregate_stability/) |
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

**Pending — 3.2.2 to 3.2.5:** the Veltia, iBEC, SLU and Neiker wet laboratory data are not yet ready for loading (the metadata is missing). They will follow the Agrolab pattern and get one page each when they are added to the notebook.
{: .notice--warning}

## Single step or two steps

The cells above use the single-step `insert_tabular_data` route. If you need to `UPDATE` an existing record or want to hand-inspect the generated JSON before it is applied, use the two-step route: translate the Excel files to manage process files, then run them. The two-step route is documented once, for Agrolab, with the notebook `./ai4sh/import_data/more_notebooks/load_ai4sh_wetlab_data.ipynb`:

1. **[Translate Agrolab data][translate_agrolab]** — translate both Excel files to JSON in one call
2. **[Manage Agrolab observation log][manage_agrolab_obs_log]** — insert the observation log records
3. **[Manage Agrolab observation][manage_agrolab_obs]** — insert the indicator values

For any other source in block 3.2, use the same three steps with that source's folder and Excel files. See [Single-step vs dual-step][insert_vs_translate] for the full comparison.

## Prerequisites

- [Load sample data][Load sample data] (cell 3.1) must be complete; the observations reference samples by name.
- The sampling logs must exist from [Load dataset metadata][dataset_meta] (cell 2.1).
- The provisions, their indicators (`provision_indicator`) and the handling catalogues (preparation, preservation, storage, transportation) must exist from [Load utility data][Load utility data] (block 1).

## Observation values: the `@` prefix

Every column (or process-file parameter) whose name starts with `@` is an observation value, not a column of `observation.observation`. The full description is on [Translate Agrolab data → The @ prefix][at_prefix]. In short:

- The text after `@` must be the **name or alias** of an indicator registered for the observation's provision in `observation_utility.provision_indicator` (`provision_indicator.xlsx`). An unregistered `@` key **stops the import** with `No indicator found for name/alias: <key>`.
- A number is stored as is. A below-detection string such as `"<0.10"` is stored as **half the limit**, `0.05` (a number). Any other non-numeric string gives the warning `⚠️ Value for indicator … is not a number` and is skipped. A missing or empty value stores nothing.
- A single value is stored in `observation.observation_measurement` (one row per observation × indicator), a list (spectra) in `observation.observation_measurement_array`.
- The unit is not in the observation record. It comes from the provision's `provision_indicator` row.

## Derived soil texture classes

After an observation's `@` values are written to `observation_measurement`, the loader derives soil texture classes from the same `@` values and stores them. There is no extra cell or process file: it runs inside the normal observation load (e.g. **3.2.1 Load Agrolab**), on both the single-step `insert_tabular_data` route and the two-step translate → manage route.

**Code** (in `xspatula_ai4sh`): `_Insert_derived_classifications` in `src/ai4sh/import_data/import_data.py`; the classifiers are in `src/ai4sh/import_data/nominal_classification.py`.

### Four derived indicators

| Indicator name | Alias | Classes | Meaning |
|---|---|---|---|
| `soil texture class usda` | `soil texture usda` | 12 | USDA texture triangle |
| `soil texture class usda major group` | `soil texture usda major group` | 4 | aggregation of the 12 USDA classes |
| `soil texture class iuss` | `soil texture iuss` | 12 | IUSS texture triangle |
| `soil texture class iuss major group` | `soil texture iuss major group` | 4 | aggregation of the 12 IUSS classes |

The classes, and how they aggregate to the four major groups (as in `ai4sh/import_data/utility/observation/excel/nominal_classes.xlsx` and the classifier):

| Major group | USDA classes | IUSS classes |
|---|---|---|
| heavy clayey soils | clay, silty clay | clay, silty clay, light clay |
| light clayey soils | sandy clay, silty clay loam, clay loam, sandy clay loam | sandy clay, silty clay loam, clay loam, sandy clay loam |
| loamy soils | silt loam, silt, loam, sandy loam | silt loam, loam, sandy loam |
| sandy soils | loamy sand, sand | loamy sand, sand |

### Inputs and formulas

Inputs are the `@` values of the same observation:

| Source | Texture `@` values |
|---|---|
| Agrolab | `@clay`, `@fine silt`, `@coarse silt`, `@fine sand`, `@coarse sand` |
| Veltia, iBEC (Bouyoucos hydrometer) | `@clay`, `@silt`, `@sand` |

- **USDA**: sand = `@sand` (else `@fine sand` + `@coarse sand`), silt = `@silt` (else `@fine silt` + `@coarse silt`), clay = `@clay`.
- **IUSS**: the IUSS silt/sand boundary is 0.02 mm, so coarse silt (0.02–0.06 mm) counts as sand.
  - All four split fractions present (**Agrolab**): sand = coarse silt + fine sand + coarse sand, silt = fine silt. Exact.
  - Otherwise (**Veltia, iBEC**): approximation sand = SAND + SILT/2, silt = SILT/2. Half the measured silt is assumed to be coarse silt.
- **Sanity check**: if any of sand, silt or clay is missing, or sand + silt + clay (before the IUSS transform) is outside **98–102 %**, no class is derived for that observation. This is not an error; the observation simply gets no texture class.
- A below-detection value (`"<x"`) has already been halved when the classifier sees it (see [the `@` prefix](#observation-values-the--prefix)), so the class is derived from the halved value.

Example, the Agrolab sample `366-2-c5@0-20` (clay 11, fine silt 7, coarse silt 7, fine sand 51, coarse sand 24):

| System | sand / silt / clay | Class | Major group |
|---|---|---|---|
| USDA | 75 / 14 / 11 | loamy sand | sandy soils |
| IUSS | 82 / 7 / 11 | sandy loam | loamy soils |

### Accuracy

- **USDA, sand ≥ 70 % corner**: the decision tree uses box limits there (sand > 85 and clay < 8 → sand; clay < 12 → loamy sand; else sandy loam) instead of the true diagonal lines of the triangle. A small number of samples near the sand / loamy sand / sandy loam boundaries can get the neighbouring class. Elsewhere the USDA tree matches the reference table.
- **IUSS** is exact against the IUSS polygon vertices (tested on all 5151 integer sand + silt + clay = 100 points). For sources with only sand, silt and clay the input is approximate (the SILT/2 split above), which is a larger uncertainty than the classifier.

### Opt-in per provision

A derived class is stored only if its indicator is registered for the observation's provision in `provision_indicator.xlsx`, with analysis method `derived` and unit `class`. All four indicators are registered for `ai4sh-agrolab`, `ai4sh-veltia` and `ai4sh-ibec`. Removing a row (e.g. a major group) switches that indicator off for that provision.

Database prerequisites, all from [Load utility data][Load utility data] (block 1):

| Prerequisite | Excel file | Cell |
|---|---|---|
| quantity `class`, unit `class` | `quantity.xlsx`, `unit.xlsx` | 1.2 |
| the four indicators, quantity `class` | `indicator.xlsx` | 1.2 |
| analysis method `derived` | `analysis_method.xlsx` | 1.2 |
| the 32 classes in `observation_utility.nominal_classification` | `nominal_classes.xlsx` (via `nominal_classes.json`) | 1.2 |
| the `provision_indicator` rows | `provision_indicator.xlsx` | 1.3 |

### Where it is stored

`observation.observation_classification` — **not** `observation_measurement`:

| Column | Content |
|---|---|
| `id` | BIGSERIAL |
| `observation_id` | → `observation.observation(id)` |
| `indicator_id` | → `observation_utility.indicator(id)`, e.g. `soil texture usda` |
| `value_id` | → `observation_utility.nominal_classification(id)`, the class, e.g. `sandy loam` |

`UNIQUE (observation_id, indicator_id)`: one class per observation and texture system, so an observation with valid texture gets up to four rows. The table is audited for UPDATE and DELETE.

List the classes per observation:

```sql
SELECT oc.observation_id,
       i.alias AS indicator,
       nc.name AS class
FROM observation.observation_classification oc
JOIN observation_utility.indicator i ON i.id = oc.indicator_id
JOIN observation_utility.nominal_classification nc ON nc.id = oc.value_id
ORDER BY oc.observation_id, i.alias;
```

### Re-runs and errors

- Re-running an observation cell on observations that already exist derives and stores missing classes. Existing rows are left alone unless `overwrite` is set. No delete is needed after fixing the utility data.
- If texture can be derived but none of the four indicators is registered for the provision, the loader prints an ERROR at every verbosity level, **once per provision per run**, lists what is missing, and continues. Example, utility insert run without quantity `class`:

```
.  ❌ ERROR: soil texture can be derived for provision ai4sh-agrolab but no derived class indicator is registered - no observation_classification will be stored for this provision (reported once per run, import continues):
.      - quantity "class" missing in observation_utility.quantity (quantity.xlsx) - every derived class indicator depends on it
.      - indicator "soil texture iuss" missing in observation_utility.indicator (indicator.xlsx)
...
```

- The same once-per-run ERROR is printed for a `provision_indicator` row without analysis method, and for a derived class missing in `nominal_classes.xlsx`.

### Selection

The derived classes are what class-filtered selection uses: `texture_class_array` in the select process, with the classification given by `texture_classification` (default `soil texture usda`). See [Select data → Texture][select_texture].

[all_data]: {{ site.baseurl }}/all_data/
[spectra]: {{ site.baseurl }}/spectra/
[edna]: {{ site.baseurl }}/edna/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[auto_naming]: {{ site.baseurl }}/setup_process/auto_naming/
[insert_agrolab]: {{ site.baseurl }}/observation/insert_agrolab/
[translate_agrolab]: {{ site.baseurl }}/observation/translate_agrolab/
[manage_agrolab_obs_log]: {{ site.baseurl }}/observation/manage_agrolab_obs_log/
[manage_agrolab_obs]: {{ site.baseurl }}/observation/manage_agrolab_obs/
[insert_digitsoil]: {{ site.baseurl }}/observation/insert_digitsoil/
[insert_best_infiltration]: {{ site.baseurl }}/observation/insert_best_infiltration/
[insert_microbiometer]: {{ site.baseurl }}/observation/insert_microbiometer/
[insert_aggregate_stability]: {{ site.baseurl }}/observation/insert_aggregate_stability/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
[Load sample data]: {{ site.baseurl }}/sample/
[dataset_meta]: {{ site.baseurl }}/dataset_meta/
[Load utility data]: {{ site.baseurl }}/utility/
[at_prefix]: {{ site.baseurl }}/observation/translate_agrolab/#the--prefix-for-observation-values
[select_texture]: {{ site.baseurl }}/machine_learning/select_data/#texture
