---
title: "Single-Step vs Dual-Step Loading"
layout: single
sidebar:
  nav: "loading_data"
excerpt: "Two ways to get spreadsheet data into the AI4SH database: a single-step insert route (INSERT-only, no manual hand-off) and a 2-step translate-then-manage route (supports UPDATE and hand-inspection)."
permalink: /insert_vs_translate/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-10-16 08:00:00 +0200
---

Every collection under [Loading data][loading_data] that starts from an Excel or CSV
spreadsheet now offers two ways to get that data into the database. Both read the same
spreadsheets and produce the same rows — the difference is what happens between "spreadsheet"
and "database row," and what you can and can't do along the way.

## The two routes

**Single step** — `insert_tabular_data`, one notebook cell:

```
Excel/CSV  --[insert_tabular_data]-->  database
```

Reads the spreadsheet, writes a staging JSON file (a debug trail, not meant to be hand-edited),
and immediately applies it to the database, row by row — no separate cell, no pilot-file
hand-off. Every row is generated `overwrite: false, delete: false`, and because there's no
human in the loop before it's applied, this is **structurally INSERT-only**: a row that already
exists (matched on its natural key) is silently left alone, never updated or duplicated.
Re-running the same cell twice is always safe.

**Dual step** — `translate_tabular_data` then a `manage_*` process, two notebook cells:

```
Excel/CSV  --[translate_tabular_data]-->  JSON  --[manage_*]-->  database
```

The translate cell writes a JSON process file you're meant to be able to open and read (and,
if needed, hand-edit) before the manage cell applies it. Because a human can review or change
that file first, this route also supports `overwrite: true` — i.e. `UPDATE`s — not just
inserts.

## Which to use

| Situation | Route |
|---|---|
| Just want a spreadsheet's rows in the database, nothing else | **Single step** |
| Need to `UPDATE` an existing record | **Dual step** |
| Want to hand-inspect or fix a bad row in the generated JSON before it's applied | **Dual step** |
| Loading from a JSON source that has no spreadsheet at all | **Dual step** (there's nothing for `insert_tabular_data` to translate) |
| Re-running the same load repeatedly (e.g. as a spreadsheet grows) | **Single step** — safe by construction |

Full mechanism details: [Insert process][setup_process_insert] (single step) and
[Translate][setup_process_translate] (dual step).

## Where each route exists today

The single-step route is the one used by `./ai4sh/import_data/insert_ai4sh_data.ipynb`. The
dual-step notebooks are kept in `./ai4sh/import_data/more_notebooks/`.

{% capture notice-1 %}
| Collection | Single step: `insert_ai4sh_data.ipynb` | Dual step: `more_notebooks/` |
|---|---|---|
| [Utility]({{ site.baseurl }}/utility/) | Cells 1.1–1.4 — see [Insert utility data]({{ site.baseurl }}/utility/insert_utility_data/) | `load_ai4sh_utility_data.ipynb` — [Translate general utilities]({{ site.baseurl }}/utility/translate_general_utilities/) etc. |
| [Dataset metadata]({{ site.baseurl }}/dataset_meta/) | Cell 2.1 — see [Insert dataset metadata]({{ site.baseurl }}/dataset_meta/insert_dataset_meta/) | `load_ai4sh_dataset_meta.ipynb` — [Translate dataset]({{ site.baseurl }}/dataset_meta/translate_dataset/) etc. |
| [Sample]({{ site.baseurl }}/sample/) | Cell 3.1 — see [Insert sample data]({{ site.baseurl }}/sample/insert_sample_data/) | `load_ai4sh_sample_data.ipynb` — [Translate geotag sample]({{ site.baseurl }}/sample/translate_geotag_sample/) etc. |
| [Wetlab]({{ site.baseurl }}/wetlab/) | Cell 3.2.1 — see [Insert wetlab data]({{ site.baseurl }}/wetlab/insert_wetlab_data/) | `load_ai4sh_wetlab_data.ipynb` — [Translate Agrolab wetlab data]({{ site.baseurl }}/wetlab/translate_ai4sh_wetlab_data/) etc. |
| [Spectra]({{ site.baseurl }}/spectra/) | Cells 3.3.1–3.3.12 (manage-only) | Manage-only, no translate step at all |
| [eDNA]({{ site.baseurl }}/edna/) | Cells 3.4.1–3.4.6 | `insert_ai4sh_taxon_data.ipynb` has disabled translate/manage cells for the taxa |
| [All data]({{ site.baseurl }}/all_data/) | All blocks | `load_ai4sh_data.ipynb` |
{% endcapture %}

<div class="notice">{{ notice-1 | markdownify }}</div>

**Spectra never had a translate step to begin with** — spectral instrument files are converted
to `manage_*` JSON directly by dedicated converters, not by translating a spreadsheet. There's
nothing for `insert_tabular_data` to collapse there, so the single-step/dual-step distinction
on this page doesn't apply to spectra.

[loading_data]: {{ site.baseurl }}/loading_data/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[setup_process_translate]: {{ site.baseurl }}/setup_process/translate/
[utility]: {{ site.baseurl }}/utility/
[dataset_meta]: {{ site.baseurl }}/dataset_meta/
[sample]: {{ site.baseurl }}/sample/
[wetlab]: {{ site.baseurl }}/wetlab/
[spectra]: {{ site.baseurl }}/spectra/
[all_data]: {{ site.baseurl }}/all_data/
[edna]: {{ site.baseurl }}/edna/
[insert_utility_data]: {{ site.baseurl }}/utility/insert_utility_data/
[insert_dataset_meta]: {{ site.baseurl }}/dataset_meta/insert_dataset_meta/
[insert_sample_data]: {{ site.baseurl }}/sample/insert_sample_data/
[insert_wetlab_data]: {{ site.baseurl }}/wetlab/insert_wetlab_data/
[translate_general_utilities]: {{ site.baseurl }}/utility/translate_general_utilities/
[translate_dataset]: {{ site.baseurl }}/dataset_meta/translate_dataset/
[translate_geotag_sample]: {{ site.baseurl }}/sample/translate_geotag_sample/
[translate_ai4sh_wetlab_data]: {{ site.baseurl }}/wetlab/translate_ai4sh_wetlab_data/
