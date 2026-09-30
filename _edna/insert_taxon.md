---
title: "Insert Taxa"
layout: single
sidebar:
  nav: "edna"
  nav2: "loading_data"
excerpt: "insert_ai4sh_taxon_data.ipynb bulk-loads the prokaryote (SILVA) and fungal (UNITE) lineages into organism_utility.taxon, rank by rank. Must run before the ASVs."
permalink: /edna/insert_taxon/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

Every ASV carries a lineage such as `k__Bacteria|p__Bacillota|…|g__Niallia|s__`. Before ASVs can be loaded, every name in those lineages must exist in the taxon tree `organism_utility.taxon`. This step builds the tree from the two lineage files written by the CLI.

It must run **before** [Insert ASVs]: the ASV loader looks up each lineage's deepest taxon and aborts if any is missing.

## Prerequisites

- [Prepare eDNA data] has written `taxa_prokaryotes_singlecolumn.xlsx` and `taxa_fungi_singlecolumn.xlsx` to `./ai4sh/import_data/eDNA/excel/`.
- Taxonomy references `silva unspecified` and `unite unspecified` exist (see [eDNA method catalogues][edna_catalogues]).
- The process `manage_taxon` is registered (see [Organism utility processes][setup_process_organism_utility]).

## Notebook

```
./ai4sh/import_data/insert_ai4sh_taxon_data.ipynb
```

After the imports and scheme-file cells, the notebook has two single-step cells, one per organism group:

```python
process_file = 'import_data/eDNA/insert_process/insert_taxon_prokaryotes.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

and the same cell pattern with `insert_taxon_fungi.json`. The process files:

| File | `tabular_data_path` | `dst_path` | Taxonomy reference |
|---|---|---|---|
| `insert_taxon_prokaryotes.json` | `import_data/eDNA/excel/taxa_prokaryotes_singlecolumn.xlsx` | `import_data/eDNA/insert_process/staging/taxon_prokaryotes` | `silva` / `unspecified` |
| `insert_taxon_fungi.json` | `import_data/eDNA/excel/taxa_fungi_singlecolumn.xlsx` | `import_data/eDNA/insert_process/staging/taxon_fungi` | `unite` / `unspecified` |

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

If you change a reference's version in `taxonomy_reference.xlsx`, change `taxonomy_reference_version` here as well.

## What happens

1. The lineages are read, normalised (lower case, `_` → space) and deduplicated. The result is written to `taxon_lineage.csv` under `dst_path`.
2. The tree is loaded **rank by rank**, kingdom first. At each rank, parents are resolved from the ranks already loaded, taxa already in the database are skipped, and the new ones are copied in with PostgreSQL `COPY`. The CSV behind each copy is kept as `taxon_copy_<rank id>_<rank>.csv`.
3. Empty ranks (`o__` with no name) are skipped. The child attaches to the nearest named ancestor.
4. Species are stored with the epithet in `name` and the binomial in `scientific_name`.

The whole load is one transaction: it completes, or it leaves the table untouched. Rerunning inserts nothing.

## Dual-step alternative

The notebook also has two cells, disabled with `%%script false`, that split the work so you can inspect the result before loading:

1. **Translate** — `import_data/eDNA/process/translate_taxon_prokaryotes.json` and `…_fungi.json` (`translate_tabular_data`) write `taxon_lineage.csv` and a `manage_taxon.json` to `import_data/eDNA/manage_process/taxon_<group>/`.
2. **Manage** — run `import_data/eDNA/manage_process/taxon_<group>/manage_taxon.json` to load the CSV.

Remove the `%%script false` line to enable them. Use either the single-step or the dual-step route, not both.

## Check

```sql
SELECT r.name AS rank, tr.alias AS reference, count(*)
FROM organism_utility.taxon t
JOIN organism_utility.taxon_rank r ON r.id = t.taxon_rank_id
JOIN organism_utility.taxonomy_reference tr ON tr.id = t.taxonomy_reference_id
GROUP BY r.id, r.name, tr.alias ORDER BY tr.alias, r.id;
```

A species epithet can occur in several genera:

```sql
SELECT name, scientific_name FROM organism_utility.taxon WHERE name = 'japonicum';
```

This should return several rows with different genera.

## Next step

Proceed to [Insert ASVs].

[Insert ASVs]: /edna/insert_edna_asv/
[Prepare eDNA data]: /edna/prepare_edna_data/
[edna_catalogues]: /edna/edna_catalogues/
[setup_process_organism_utility]: /setup_process/organism_utility/
