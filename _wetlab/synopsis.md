---
title: "Load Wetlab Data"
layout: single
sidebar:
  nav: "wetlab"
  nav2: "loading_data"
excerpt: "Overview of Agrolab wet laboratory data loading. Soil chemical properties are linked to samples via an observation log. Two Excel files are translated and managed in three steps."
permalink: /wetlab/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Wet laboratory (Agrolab) data contains chemically determined soil properties measured per sample — pH, organic carbon, nitrogen, texture fractions, ions, and more. Each measurement is anchored to a sample through an observation log that records which laboratory protocol and provision were used.

Two notebooks can load this data — pick one:

```
./ai4sh/import_data/insert_ai4sh_wetlab_data.ipynb   (single-step, recommended default)
./ai4sh/import_data/load_ai4sh_wetlab_data.ipynb     (2-step translate + manage)
```

**Which one should I use?** Use `insert_ai4sh_wetlab_data.ipynb` unless you need to `UPDATE` an
existing record or want to hand-inspect/edit the generated JSON before it's applied — the
single-step route is INSERT-only by construction, so it's always safe to re-run. See
[Insert wetlab data][insert_wetlab_data] for the single-step walkthrough and
[Single-step vs dual-step][insert_vs_translate] for the full comparison.

## Two tables, two source files

Wetlab loading populates two database tables from two Excel files:

| Excel file | Database table | Description |
|---|---|---|
| `AI4SH_observation_log_agrolab.xlsx` | `observation.observation_log` | Links sampling logs to the Agrolab provision, plus lab handling metadata |
| `AI4SH_observation_agrolab.xlsx` | `observation.observation` | One row per sample per measured property |

Source directory: `./ai4sh/import_data/wetlab/agrolab/excel/`

Both files are translated in a single process file call, then managed separately.

## Required loading sequence

**Single step** (default): **[Insert wetlab data]** — 2 cells, observation log then observation, translate and insert combined.

**2-step** (for `UPDATE`s or manual inspection), 3 cells:

1. **[Translate Agrolab wetlab data]** — translate both Excel files to JSON in one call
2. **[Manage Agrolab observation log]** — insert observation log records linking sampling logs to the Agrolab provision
3. **[Manage Agrolab observation]** — insert individual soil property measurements

## Prerequisites

[Load sample data] must be complete. Observation utility records (provision, provision_indicator, preparation, preservation, storage, transportation) must exist from [Load utility data].

[Insert wetlab data]: /wetlab/insert_wetlab_data/
[insert_vs_translate]: /insert_vs_translate/
[Translate Agrolab wetlab data]: /wetlab/translate_ai4sh_wetlab_data/
[Manage Agrolab observation log]: /wetlab/manage_ai4sh_agrolab_obs_log/
[Manage Agrolab observation]: /wetlab/manage_ai4sh_wetlab_data/
[Load sample data]: /sample/
[Load utility data]: /utility/
