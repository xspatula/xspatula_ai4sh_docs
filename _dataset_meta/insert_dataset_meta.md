---
title: "Insert Dataset Metadata (Single Step)"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "insert_ai4sh_dataset_meta.ipynb loads data source, persons, dataset, campaign, and sampling log in 5 cells instead of 6 — translate and insert happen together per table, with no separate manage step."
permalink: /dataset_meta/insert_dataset_meta/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-08-21 08:00:00 +0200
---

`insert_ai4sh_dataset_meta.ipynb` is the single-step alternative to [Translate dataset][translate_dataset]
plus its five manage pages. Unlike the utility route, each table here gets its own cell and its
own single `process_file` (not a shared `job_file`/pilot list) — see [Insert process][setup_process_insert]
for the mechanism.

## Which route to use

Use this notebook when you just want the five spreadsheets' rows in the database and don't
need to `UPDATE` an existing record or hand-inspect the generated JSON. Use [Translate dataset][translate_dataset]
plus the manage pages when you do. See [Single-step vs dual-step][insert_vs_translate] for the
full comparison.

## Notebook cells

In `insert_ai4sh_dataset_meta.ipynb`, after the shared imports and scheme-file cells, five
cells each translate and insert one table, in the same dependency order as the 2-step route
(data source and person before dataset, dataset before campaign, campaign before sampling
log):

```python
process_file = 'import_data/dataset/insert_process/data_source.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

The remaining four cells are identical in shape, pointing at `person.json`, `dataset.json`,
`campaign.json`, and `sampling_log.json` in turn.

## Process files

Each table gets one process file under `./ai4sh/import_data/dataset/insert_process/`, same
shape as the existing `process/<table>.json` translate files, but calling
`insert_tabular_data` instead of `translate_tabular_data`, and writing its staging output to
`insert_process/staging` instead of `manage_process`. Example, `data_source.json`:

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_data_source",
        "tabular_data_path": "import_data/dataset/excel/data_source.xlsx",
        "dst_path": "import_data/dataset/insert_process/staging"
      }
    }
  ]
}
```

The other four tables follow the same pattern, one process file each. There is deliberately no
`insert_process/manage_process` step and no separate pilot file for this route — the
`insert_process/staging/*.json` files that appear after running these cells are a debug trail,
not something you're meant to edit.

## A note for readers of [Insert sample data][insert_sample_data]

The single-step sample route re-applies `import_data/dataset/insert_process/sampling_log.json`
directly as a safety net (in case that notebook is run on its own, without this one having run
first) — the same file described above, reused as-is. That's safe precisely because
`insert_tabular_data` is INSERT-only: re-running it never duplicates or overwrites the
sampling logs this notebook already inserted.

[translate_dataset]: /dataset_meta/translate_dataset/
[insert_sample_data]: /sample/insert_sample_data/
[setup_process_insert]: /setup_process/insert/
[insert_vs_translate]: /insert_vs_translate/
