---
title: "Insert ASVs"
layout: single
sidebar:
  nav: "edna"
  nav2: "loading_data"
excerpt: "The last two cells of insert_ai4sh_edna_data.ipynb bulk-load the amplicon sequence variants and their abundance per observation for the 16S and ITS pipelines, then verify the load."
permalink: /edna/insert_edna_asv/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

The last step loads the community composition: which ASVs were found in which sample, and how abundant each was. The data goes into `observation.edna_asv` (one row per ASV and pipeline) and `observation.edna_asv_abundance` (one row per ASV and observation, non-zero values only). With several hundred thousand abundance rows, this uses the bulk process `manage_edna_asv`, which loads with PostgreSQL `COPY`.

## Prerequisites

- [Insert eDNA observations] is complete: every observation referenced in the ASV files must exist.
- [Insert taxa] is complete: every lineage must resolve to a taxon.
- Method pipelines `ai4sh-16s` and `ai4sh-its` exist (see [eDNA method catalogues][edna_catalogues]).

## Notebook cells

In `./ai4sh/import_data/insert_ai4sh_edna_data.ipynb`, after the two observation cells:

```python
process_file = 'import_data/eDNA/insert_process/insert_asv_prokaryotes.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

and the same cell pattern with `insert_asv_fungi.json`.

| File | `tabular_data_path` | `dst_path` | `method_pipeline` |
|---|---|---|---|
| `insert_asv_prokaryotes.json` | `import_data/eDNA/excel/AI4SH_asv_prokaryotes_eDNA.xlsx` | `import_data/eDNA/insert_process/staging/asv_prokaryotes` | `ai4sh-16s` |
| `insert_asv_fungi.json` | `import_data/eDNA/excel/AI4SH_asv_fungi_eDNA.xlsx` | `import_data/eDNA/insert_process/staging/asv_fungi` | `ai4sh-its` |

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_edna_asv",
        "tabular_data_path": "import_data/eDNA/excel/AI4SH_asv_fungi_eDNA.xlsx",
        "dst_path": "import_data/eDNA/insert_process/staging/asv_fungi",
        "method_pipeline": "ai4sh-its"
      }
    }
  ]
}
```

The `method_pipeline` parameter ties every ASV to its pipeline, and through it to the full method description. It is the only place where the pipeline is named during loading.

Dual-step translate files (`import_data/eDNA/process/translate_asv_prokaryotes.json` and `…_fungi.json`, writing to `import_data/eDNA/manage_process/asv_<group>/`) exist but have no notebook cells.

## What happens

1. The Excel file is converted into a canonical CSV `edna_asv_abundance.csv` under `dst_path`.
2. Each row's observation is resolved from its observation log and sample.
3. Each lineage is resolved to its deepest named taxon.
4. New ASVs are inserted into `edna_asv`, keyed by `(method_pipeline, asv_key)`. Then the abundances go into `edna_asv_abundance` (`rel_abundance`, `read_count`; `raw_read_count` stays empty).

**Safety checks.** The load aborts **without writing anything** if:

- an observation is not found;
- a lineage has no matching taxon;
- an existing `asv_key` now carries a different lineage.

The last check matters because the ASV key is the ASV's row number in the laboratory's sheet. If a new delivery reorders the rows, the keys no longer mean the same ASV, and the load is refused rather than silently mixing two orders.

**Insert only.** ASVs and abundances already in the database are kept. Rerunning inserts nothing.

## Verify

Counts per pipeline. Each sample should sum to 20,000 rarefied reads, so `reads` should be about 20,000 × `observations`:

```sql
SELECT mp.name, count(DISTINCT a.id) AS asvs, count(DISTINCT ab.observation_id) AS observations,
       sum(ab.read_count) AS reads
FROM observation.edna_asv a
JOIN observation_utility.method_pipeline mp ON mp.id = a.method_pipeline_id
JOIN observation.edna_asv_abundance ab ON ab.edna_asv_id = a.id
GROUP BY mp.name;
```

The relative abundances of every observation should sum to 1:

```sql
SELECT observation_id, round(sum(rel_abundance)::numeric, 6) AS total
FROM observation.edna_asv_abundance
GROUP BY observation_id
HAVING abs(sum(rel_abundance) - 1) > 1e-6;
```

No rows means all samples are complete.

Each summary indicator links to exactly one pipeline through its analysis method:

```sql
SELECT mp.name, count(DISTINCT pi.indicator_id) AS indicators
FROM observation_utility.provision_indicator pi
JOIN observation_utility.method_pipeline mp ON mp.analysis_method_id = pi.analysis_method_id
GROUP BY mp.name;
```

This should return 9 indicators for `ai4sh-16s` and 10 for `ai4sh-its`.

The most abundant genera across all fungal samples:

```sql
SELECT t.scientific_name, sum(ab.read_count) AS reads
FROM observation.edna_asv_abundance ab
JOIN observation.edna_asv a ON a.id = ab.edna_asv_id
JOIN observation_utility.method_pipeline mp ON mp.id = a.method_pipeline_id
JOIN organism_utility.taxon t ON t.id = a.taxon_id
JOIN organism_utility.taxon_rank r ON r.id = t.taxon_rank_id
WHERE mp.name = 'ai4sh-its' AND r.name = 'genus'
GROUP BY t.scientific_name ORDER BY reads DESC LIMIT 20;
```

This query counts only ASVs resolved to genus level. ASVs resolved to species level have their species as `taxon_id`; to include them, walk up via `parent_taxon_id`.

## Known gaps

- ASV sequences are not delivered yet, so `edna_asv.sequence` and `sequence_md5` are empty.
- `raw_read_count` is empty until the laboratory delivers the unrarefied feature table.
- `observation.edna_run_step` is empty until the laboratory delivers per-step read counts.

See [eDNA metabarcoding][setup_db_edna] for the full list.

[Insert eDNA observations]: /edna/insert_edna_observation/
[Insert taxa]: /edna/insert_taxon/
[edna_catalogues]: /edna/edna_catalogues/
[setup_db_edna]: /setup_db/edna_metabarcoding/
