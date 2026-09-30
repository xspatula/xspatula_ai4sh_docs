---
title: "Organism Utility Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Organism utility processes manage the taxonomy reference databases and bulk-load the taxon tree into organism_utility.taxon."
permalink: /setup_process/organism_utility/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

Organism utility processes manage the biological reference data in the `organism_utility` schema (see [Organism utility schema][setup_db_organism_utility]). Only two tables are fed through processes: `taxonomy_reference`, one row at a time, and `taxon`, in bulk. The ranks, statuses and functions are seeded directly by the setup files.

## Process files

Process files are located at:

```
./setup/zzz/ai4sh/setup_processes/json/organism_utility/
```

| File | Process registered | Target table | Min stratum |
|---|---|---|---|
| `taxonomy_reference_v10_sql.json` | `manage_taxonomy_reference` | `organism_utility.taxonomy_reference` | 3 |
| `taxon_v10_sql.json` | `manage_taxon` | `organism_utility.taxon` (bulk COPY) | 3 |

Both are listed in `setup_processes.txt` in the section `### ORGANISM UTILITIES ###`, after the observation utilities and before the observation processes.

## manage_taxonomy_reference

Registers a named and versioned reference database that taxon names are resolved against.

| Parameter | Type | Required | Update | Description |
|---|---|---|---|---|
| `name` | text | yes | — | e.g. `silva`, `unite` |
| `version` | text | no (default `unspecified`) | — | Release, e.g. `138.2` |
| `alias` | text | yes | no | Unique `'<name> <version>'`, e.g. `silva unspecified` — what pipeline steps reference |
| `marker_gene` | text | yes | — | e.g. `16S rRNA`, `ITS` |
| `url` | text | yes | — | Landing page |
| `doi` | text | no | — | Citation DOI |
| `abstract` | text | no | — | Description |

The AI4SH rows are inserted from `taxonomy_reference.xlsx` in the utility insert chain — see [eDNA method catalogues][edna_catalogues].

## manage_taxon

Bulk-loads taxonomic lineages into `organism_utility.taxon` with PostgreSQL `COPY`. Like `manage_edna_asv`, it is a *bulk* process: it is called once with a path to a prepared CSV, not once per taxon.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `taxon_data_path` | text | yes | Path to the canonical lineage CSV written by `translate_tabular_data` / `insert_tabular_data` with target process `manage_taxon` |
| `dst_path` | text | no | Folder where the CSV source of each per-rank `COPY` is written, for inspection |

You normally don't call `manage_taxon` directly. You call `insert_tabular_data` (single step) or `translate_tabular_data` (dual step) with `"process": "manage_taxon"`. These accept two extra parameters, `taxonomy_reference` and `taxonomy_reference_version` (see [Insert][setup_process_insert]):

```json
{
  "process": [
    {
      "process": "insert_tabular_data",
      "overwrite": true,
      "parameters": {
        "process": "manage_taxon",
        "tabular_data_path": "import_data/eDNA/excel/taxa_fungi_singlecolumn.xlsx",
        "dst_path": "import_data/eDNA/insert_process/staging/taxon_fungi",
        "taxonomy_reference": "unite",
        "taxonomy_reference_version": "unspecified"
      }
    }
  ]
}
```

### Input layouts

Two layouts are accepted, as Excel or CSV:

- **Single column**: one lineage string per row, ranks separated by `|` and tagged with the `taxon_rank.prefix`, e.g. `k__Bacteria|p__Bacillota|…|g__Niallia|s__`.
- **Multi column**: one column per rank, headed by the rank name: `kingdom`, `phylum`, `class`, `order`, `family`, `genus`, `species`, `subspecies`, `variety`.

Either layout may carry the optional columns `taxonomy_reference` and `taxonomy_reference_version`, which override the process parameters for that row. Use this to mix SILVA and UNITE lineages in one file.

Names are normalised: lowercased, underscores converted to spaces and whitespace collapsed. Species rows store the epithet in `name` and the binomial in `scientific_name`.

### How the load works

- **Rank by rank, kingdom first.** Each rank's parents are resolved from the ranks already loaded. Where a rank is empty in the lineage, the child attaches to the nearest named ancestor.
- **Insert only.** Taxa already in the database are skipped; only new taxa are copied. Rerunning the same file inserts nothing.
- **One transaction.** The load either completes or leaves the table untouched.

The process is routed to its bulk importer (`src/ai4sh/import_data/import_taxon.py`) through `BULK_PROCESS_D` in `src/ai4sh/import_data/import_data.py`, rather than through the row-by-row `manage_table_data` handler.

For how to run it on the AI4SH eDNA data, see [Insert taxa][edna_insert_taxon].

## Troubleshooting

If `setup_processes.ipynb` reports that parameter `taxonomy_reference` (or `method_pipeline`) is not defined, the translate/insert processes were registered before these parameters were added. Set `"overwrite": true` once in `translate/insert_tabular_data_v10_sql.json` and `translate/translate_tabular_data_v10_sql.json`, rerun `setup_processes.ipynb`, and set it back to `false`.

[setup_db_organism_utility]: /setup_db/organism_utility/
[setup_process_insert]: /setup_process/insert/
[edna_catalogues]: /edna/edna_catalogues/
[edna_insert_taxon]: /edna/insert_taxon/
