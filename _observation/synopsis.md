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
last_modified_at: 2026-10-17 08:00:00 +0200
---

This collection covers block **3.2** of the single loading notebook (see [Load all data][all_data]): all AI4SH observations except spectra and eDNA. Spectra need spectrometer records and signal arrays, and eDNA needs taxa and method pipelines, so they have their own collections ([Load spectral data][spectra], [Load eDNA data][edna]). Everything in block 3.2 is a set of indicator values per sample, and every source is loaded the same way.

```
./ai4sh/import_data/insert_ai4sh_data.ipynb   block 3.2, cells 3.2.1–3.2.9
```

## One pattern for all sources

Each cell runs one process file with two (or more) `insert_tabular_data` entries — see [Insert process][setup_process_insert]:

1. **Observation log** — `manage_observation_log` reads an Excel file with one row per sampling log, linking it to the provision (instrument, kit, app or laboratory) and recording the sample handling (preparation, preservation, storage, transportation). The observation log name is composed automatically as `<sampling log>@<provision>`, see [Automatic naming and tags][auto_naming].
2. **Observations** — `manage_observation` reads an Excel file with one row per sample (or subsample). The identifier columns name the observation log and the sample; every column starting with `@` is an indicator value, e.g. `@ph-h2o`. Each `@` column must match an indicator registered for that provision in `observation_utility.provision_indicator`.

The observation log must be loaded first, as the observation rows reference it by name. Both are INSERT-only and safe to re-run. The `@` convention and all columns are described in detail for Agrolab on [Translate Agrolab data][translate_agrolab]; the other sources only differ in folder, provision and indicators.

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
