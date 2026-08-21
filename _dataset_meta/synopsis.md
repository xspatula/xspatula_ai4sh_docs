---
title: "Load Dataset Metadata"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Overview of the dataset metadata hierarchy and the required loading sequence. Covers data source, persons, dataset, campaign, and sampling log — all prerequisites for loading any sample or observation data."
permalink: /dataset_meta/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Dataset metadata describes the provenance of all soil observations: who collected them, under what organisational umbrella, in which campaign, and at which sampling operation. This chain of records must be in place before any sample position or observation value can be entered.

Two notebooks can load this data — pick one:

```
./ai4sh/import_data/insert_ai4sh_dataset_meta.ipynb   (single-step, recommended default)
./ai4sh/import_data/load_ai4sh_dataset_meta.ipynb     (2-step translate + manage)
```

**Which one should I use?** Use `insert_ai4sh_dataset_meta.ipynb` unless you need to `UPDATE`
an existing record or want to hand-inspect/edit the generated JSON before it's applied — the
single-step route is INSERT-only by construction, so it's always safe to re-run. See
[Insert dataset metadata][insert_dataset_meta] for the single-step walkthrough and
[Single-step vs dual-step][insert_vs_translate] for the full comparison.

## The dataset hierarchy

Data in the AI4SH database follows a strict five-level hierarchy:

```
data_source
    └── dataset         (belongs to one data_source)
          └── campaign  (belongs to one dataset)
                └── sampling_log  (belongs to one campaign)
                      └── sample  (belongs to one sampling_log — loaded separately)
```

Persons are registered independently and referenced by campaigns and sampling logs.

| Table | Description |
|---|---|
| `observation.data_source` | The organisation or individual who owns and provides the data |
| `observation.person` | Named contacts responsible for data collection and analysis |
| `observation.dataset` | Top-level grouping of data from one source, usually thematically coherent |
| `observation.campaign` | A bounded sampling and observation effort within a dataset |
| `observation.sampling_log` | A specific sampling operation (tools, dates, persons) within a campaign |

## Source files

All five Excel files are in `./ai4sh/import_data/dataset/excel/`:

| File | Database table |
|---|---|
| `data_source.xlsx` | `observation.data_source` |
| `person.xlsx` | `observation.person` |
| `dataset.xlsx` | `observation.dataset` |
| `campaign.xlsx` | `observation.campaign` |
| `sampling_log.xlsx` | `observation.sampling_log` |

## Required loading sequence

Same dependency order either way — data source and person before dataset, dataset before
campaign, campaign before sampling log.

**Single step** (default): **[Insert dataset metadata]** — 5 cells, one per table, translate
and insert combined, no shared job file.

**2-step** (for `UPDATE`s or manual inspection), 6 cells — all five tables translated in a
single notebook cell, then each manage step uses a direct `process_file` call, one per cell:

1. **[Translate dataset]** — single cell translates all five tables to JSON
2. **[Manage data source]** — insert data source records
3. **[Manage persons]** — insert person records (requires data source)
4. **[Manage dataset]** — insert dataset records (requires data source, license, spatial reference, profiling, species)
5. **[Manage campaign]** — insert campaign records (requires dataset, territory, spatial reference, location method, unit, provision)
6. **[Manage sampling log]** — insert sampling log records (requires campaign)

After completing either route, proceed to [Load sample data] to register individual sample positions.

[Insert dataset metadata]: /dataset_meta/insert_dataset_meta/
[insert_dataset_meta]: /dataset_meta/insert_dataset_meta/
[insert_vs_translate]: /insert_vs_translate/
[Translate dataset]: /dataset_meta/translate_dataset/
[Manage data source]: /dataset_meta/manage_data_source/
[Manage persons]: /dataset_meta/manage_person/
[Manage dataset]: /dataset_meta/manage_dataset/
[Manage campaign]: /dataset_meta/manage_campaign/
[Manage sampling log]: /dataset_meta/manage_sampling_log/
[Load sample data]: /sample/
