---
title: "Insert Utility Data (Single Step)"
layout: single
sidebar:
  nav: "utility"
  nav2: "loading_data"
excerpt: "Block 1 of insert_ai4sh_data.ipynb loads all utility groups in 4 cells (1.1–1.4) — translate and insert happen together, with no manual pilot-file hand-off. INSERT-only, safe to re-run."
permalink: /utility/insert_utility_data/
author_profile: false
date: 2026-08-21 08:00:00 +0200
last_modified_at: 2026-10-16 08:00:00 +0200
---

Block **1. Utilities** of `./ai4sh/import_data/insert_ai4sh_data.ipynb` (see [Load all data][all_data])
is the single-step alternative to the 6-cell translate-then-manage walkthrough covered by
[Translate general utilities][translate_general_utilities], [Manage general utilities][manage_general_utilities],
and the observation-utility page pairs. Each of its cells does in one step what previously took a
translate cell and a manage cell — see [Insert process][setup_process_insert] for the mechanism
this relies on.

## Which route to use

Use block 1 when you just want a spreadsheet's rows in the database and don't need to
inspect the generated JSON or `UPDATE` an existing record. Use the existing translate/manage
route (`more_notebooks/load_ai4sh_utility_data.ipynb`) when you need to `UPDATE`, or want to hand-inspect/edit the generated JSON before it's
applied. See [Synopsis][utility_synopsis] for the fuller "which one should I use" guidance,
and [Single-step vs dual-step][insert_vs_translate] for the site-wide comparison.

## Notebook cells

In `insert_ai4sh_data.ipynb`, after the shared imports and scheme-file cells, block 1 has four
cells that each translate and insert one utility group in a single call. Run them in this order:

{% capture notice-1 %}
| Cell | Name | `job_file` | Pilot file |
|---|---|---|---|
| 1.1 | Load general utilities | `import_data/utility/job_insert_general_utility.json` | `general/insert_general_utility.txt` |
| 1.2 | Load observation utilities | `import_data/utility/job_insert_observation_utility.json` | `observation/insert_observation_utility.txt` |
| 1.3 | Load observation utilities with inheritance | `import_data/utility/job_insert_observation_utility_inherit.json` | `observation/insert_observation_utility_inherit.txt` |
| 1.4 | Load landscape utility | `import_data/utility/job_insert_landscape_utility.json` | `landscape/insert_landscape_utility.txt` |
{% endcapture %}

<div class="notice">{{ notice-1 | markdownify }}</div>

Cell 1.1:

```python
job_file = 'import_data/utility/job_insert_general_utility.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, job_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

Cells 1.2–1.4 are identical in shape, pointing at the job files in the table above. Order
matters — general utilities first, then observation utilities, then the inheritance-dependent
ones (1.3 must run after 1.2), then landscape utilities — the same dependency order as the
translate/manage route.

- **Cell 1.1** inserts `territory` and `foreign_key`.
- **Cell 1.2** inserts the independent and dependent observation utilities, ending with the
  [eDNA method catalogues][edna_catalogues].
- **Cell 1.3** inserts `provision`, `provision_indicator` and `provision_serial_nr`, which can
  inherit values from the tables loaded in 1.2.
- **Cell 1.4** inserts the landscape catalogues `biogeo16`, `landuse` and `landcover`. AI4SH has no
  landscape observations; the block is there for completeness and for merging with LUCAS data.
  You can switch it off (see [Running only part of the notebook][partial]).

## Job files

Each `job_insert_*.json` mirrors the existing `job_translate_*.json` shape, but points at a
parallel `insert_process/` folder instead of `process/`:

```json
{
  "process": {
    "job_folder": "import_data/utility/general",
    "process_sub_folder": "insert_process",
    "pilot_file": "insert_general_utility.txt"
  }
}
```

The pilot files (`insert_general_utility.txt`, `insert_observation_utility.txt`,
`insert_observation_utility_inherit.txt`, `insert_landscape_utility.txt`) list the same tables,
in the same dependency order, as the `translate_*.txt` pilot files of the 2-step route — just
re-headed for the insert route. The exception is `foreign_key.json`, which only exists in the
insert route.

## Process files

Each table gets one process file under `insert_process/`, same shape as the existing
`process/<table>.json` translate files, but calling `insert_tabular_data` instead of
`translate_tabular_data`, and writing its staging output to `insert_process/staging` instead
of `manage_process`:

```json
{
  "process": [
    {
      "overwrite": true,
      "process": "insert_tabular_data",
      "parameters": {
        "process": "manage_territory",
        "tabular_data_path": "import_data/utility/general/excel/territory.xlsx",
        "dst_path": "import_data/utility/general/insert_process/staging"
      }
    }
  ]
}
```

The observation-utility group works the same way, e.g. `unit.json`:

```json
{
  "process": [
    {
      "overwrite": true,
      "process": "insert_tabular_data",
      "parameters": {
        "process": "manage_unit",
        "tabular_data_path": "import_data/utility/observation/excel/unit.xlsx",
        "dst_path": "import_data/utility/observation/insert_process/staging"
      }
    }
  ]
}
```

There is deliberately **no** `insert_process/manage_process` step and no separate "manage"
job/pilot files for this route — that whole hand-off is what got automated away. The
`insert_process/staging/*.json` files that appear after running these cells are a debug trail,
not something you're meant to edit.

[translate_general_utilities]: {{ site.baseurl }}/utility/translate_general_utilities/
[manage_general_utilities]: {{ site.baseurl }}/utility/manage_general_utilities/
[utility_synopsis]: {{ site.baseurl }}/utility/
[setup_process_insert]: {{ site.baseurl }}/setup_process/insert/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
[all_data]: {{ site.baseurl }}/all_data/
[partial]: {{ site.baseurl }}/all_data/#running-only-part-of-the-notebook
[edna_catalogues]: {{ site.baseurl }}/edna/edna_catalogues/
