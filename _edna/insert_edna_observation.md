---
title: "Insert eDNA Observations"
layout: single
sidebar:
  nav: "edna"
  nav2: "loading_data"
excerpt: "Cells 3.4.3–3.4.4 of insert_ai4sh_data.ipynb insert one eDNA observation log per sampling log, then one observation per sample with its 19 summary indicators."
permalink: /edna/insert_edna_observation/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-10-16 08:00:00 +0200
---

Cells 3.4.3 and 3.4.4 of `insert_ai4sh_data.ipynb` load the per-sample summary of the eDNA analysis. This works exactly like the [wetlab data][insert_wetlab_data]: first an observation log that links each sampling log to the metabarcoding provision, then one observation per sample carrying the indicator values.

## Prerequisites

- [Prepare eDNA data] has been run, and `AI4SH_observation_log_eDNA.xlsx` and `AI4SH_observation_eDNA.xlsx` exist in `./ai4sh/import_data/eDNA/excel/`.
- [Load sample data] is complete.
- Provision `ai4sh-metabarcoding` and its 19 `provision_indicator` records exist (see [eDNA method catalogues][edna_catalogues]).

## Notebook cells

```
./ai4sh/import_data/insert_ai4sh_data.ipynb   block 3.4, cells 3.4.3 and 3.4.4
```

### Cell 3.4.3 Load eDNA observation log

```python
process_file = 'import_data/eDNA/insert_process/insert_eDNA_observation_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

The process file:

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_observation_log",
        "tabular_data_path": "import_data/eDNA/excel/AI4SH_observation_log_eDNA.xlsx",
        "dst_path": "import_data/eDNA/insert_process/staging"
      }
    }
  ]
}
```

This inserts one observation log per sampling log. Its name is generated automatically as `<sampling log>@ai4sh-metabarcoding`, for example `ai4sh_dk_foulum_2024@ai4sh-metabarcoding`.

### Cell 3.4.4 Load eDNA summary observation

The same cell pattern with `process_file = 'import_data/eDNA/insert_process/insert_eDNA_observation.json'`. The target process is `manage_observation`, reading `AI4SH_observation_eDNA.xlsx`.

This inserts one `observation` per sample, and in `observation_measurement` one value per `@indicator` column:

| Group | Indicators |
|---|---|
| Prokaryotes, alpha diversity | richness, Chao1 richness, dominance, Pielou evenness, Shannon, Simpson |
| Prokaryotes, functional (FAPROTAX) | chemoheterotrophy, human pathogens all, nitrogen fixation |
| Fungi, alpha diversity | richness, Chao1 richness, dominance, Pielou evenness, Shannon, Simpson |
| Fungi, functional (FUNGuild) | ectomycorrhizal, arbuscular mycorrhizal, saprotrophic, plant pathogenic |

Diversity indices follow the QIIME2 definitions: Shannon uses log2, Simpson is 1 − dominance, dominance is Σp², and Pielou evenness is Shannon / log2(richness).

The staging JSON written by both cells (`staging/manage_observation_log.json`, `staging/manage_observation.json`) can be inspected afterwards, but is not needed again.

## Check

```sql
SELECT ol.name, count(DISTINCT o.id) AS observations, count(m.id) AS n_values
FROM observation.observation_log ol
JOIN observation_utility.provision p ON p.id = ol.provision_id
JOIN observation.observation o ON o.observation_log_id = ol.id
LEFT JOIN observation.observation_measurement m ON m.observation_id = o.id
WHERE p.alias = 'ai4sh-metabarcoding'
GROUP BY ol.name ORDER BY ol.name;
```

You should see one row per sampling log, with at most 19 values per observation.

## Next step

Proceed to [Insert ASVs] (cells 3.4.5–3.4.6). The taxa (cells 3.4.1–3.4.2, [Insert taxa]) must be loaded before the ASVs.

[insert_wetlab_data]: {{ site.baseurl }}/wetlab/insert_wetlab_data/
[Prepare eDNA data]: {{ site.baseurl }}/edna/prepare_edna_data/
[Load sample data]: {{ site.baseurl }}/sample/
[edna_catalogues]: {{ site.baseurl }}/edna/edna_catalogues/
[Insert taxa]: {{ site.baseurl }}/edna/insert_taxon/
[Insert ASVs]: {{ site.baseurl }}/edna/insert_edna_asv/
