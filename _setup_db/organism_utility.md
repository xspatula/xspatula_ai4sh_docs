---
title: "Organism Utility Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The organism_utility schema holds the biological reference data — the taxon tree, taxon ranks, status, synonyms, ecosystem functions and the taxonomy reference databases (e.g. SILVA, UNITE) that names are resolved against."
permalink: /setup_db/organism_utility/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

The `organism_utility` schema holds the biological reference data of the database — *what* an organism is, as opposed to *how* it was observed. Think of it as a shared family tree: macrofauna counted by hand and fungi found by eDNA sequencing both point to the same `taxon` rows, so a genus is stored once regardless of the method that detected it.

The schema is deliberately limited to biology. Method catalogues — software, primers, lab protocols, pipelines — live in [observation_utility][setup_db_observation_utility], even when they are only used for organisms.

## Process files

All six files are in the `### ORGANISM UTILITIES ###` section of the pilot file `db_setup.txt`, placed after the observation utilities and before the method pipelines and the observation schema (both of which reference `taxon` or `taxonomy_reference`).

| File | Table created | Dependencies | Seeded content |
|---|---|---|---|
| `organism_utility/taxonomy_reference_v10_sql.json` | `taxonomy_reference` | none | blank `id = 0` row only |
| `organism_utility/taxon_rank_v10_sql.json` | `taxon_rank` | none | 9 Linnean ranks (see below) |
| `organism_utility/taxon_status_v10_sql.json` | `taxon_status` | none | `accepted`, `synonym` |
| `organism_utility/taxon_v10_sql.json` | `taxon` | `taxon_rank`, `taxon_status`, `taxonomy_reference`, itself | none — loaded in bulk |
| `organism_utility/taxon_parity_v10_sql.json` | `taxon_parity` | `taxon` | none |
| `organism_utility/taxon_function_v10_sql.json` | `taxon_function` | none | 7 ecosystem functions |

## taxonomy_reference

A named and versioned reference database that taxon names were resolved against, e.g. SILVA for prokaryotes (16S) and UNITE for fungi (ITS).

| Column | Notes |
|---|---|
| `name` | e.g. `silva`, `unite` |
| `version` | `NOT NULL DEFAULT 'unspecified'` — see the note below |
| `alias` | UNIQUE, by convention `'<name> <version>'`, e.g. `silva unspecified`; used by other tables to look up the reference |
| `marker_gene` | e.g. `16S rRNA`, `ITS` |
| `url`, `doi`, `abstract` | Provenance |

Constraint: `UNIQUE (name, version)`.

**Why `'unspecified'` and not NULL?** PostgreSQL treats two NULLs as different, so `UNIQUE (name, version)` would let `silva`/NULL be inserted again on every rerun. `'unspecified'` makes the pair unique and re-runs idempotent. Once the lab confirms the release, update it with SQL (the insert route never updates existing rows).

The content rows are inserted from Excel in the utility insert chain — see [eDNA method catalogues][edna_catalogues].

## taxon_rank

| id | name | alias | prefix |
|---|---|---|---|
| 1 | kingdom | regnum | `k__` |
| 2 | phylum | divisio | `p__` |
| 3 | class | classis | `c__` |
| 4 | order | ordo | `o__` |
| 5 | family | familia | `f__` |
| 6 | genus | genus | `g__` |
| 7 | species | species | `s__` |
| 8 | subspecies | subspecies | — |
| 9 | variety | varietas | — |

The `prefix` is the rank marker used in lineage strings delivered by bioinformatics pipelines, e.g. `k__Fungi|p__Ascomycota|c__Sordariomycetes|…`. The taxon importer uses it to assign each name to its rank.

## taxon_status

`accepted` (currently accepted name in the taxonomy reference) and `synonym` (outdated or alternative name, linked to the accepted taxon via `taxon_parity`).

## taxon

One row per node in the taxonomic tree.

| Column | Notes |
|---|---|
| `parent_taxon_id` | Self-reference to the parent node; NULL for kingdoms |
| `taxon_rank_id` | → `taxon_rank` |
| `taxon_status_id` | → `taxon_status` |
| `taxonomy_reference_id` | → `taxonomy_reference` — which database the name came from |
| `name` | Name at its rank. **For species, the epithet only** (e.g. `elkanii`) |
| `scientific_name` | Full scientific name (e.g. `bradyrhizobium elkanii`); equals `name` above species rank |

Constraints and indexes:

- `UNIQUE NULLS NOT DISTINCT (parent_taxon_id, name)` — a name is unique *under its parent*. The same epithet (e.g. `japonicum`) can exist in many genera, and `NULLS NOT DISTINCT` makes two kingdoms with the same name (parent NULL) collide as they should.
- Indexes on `name` and `scientific_name`, with column comments that repeat the epithet rule.
- Audited on `UPDATE` and `DELETE` only — the bulk insert of thousands of taxa is not written to the audit log.

When a lineage skips a rank (e.g. an empty `o__`), the child is attached to the nearest named ancestor, so the tree has no placeholder nodes.

The table is filled in bulk by the process `manage_taxon` — see [Organism utility processes][setup_process_organism_utility] and [Insert taxa][edna_insert_taxon].

## taxon_parity

Links synonymous taxa: `src_taxa_id` and `dst_taxa_id`, both → `taxon`, `UNIQUE (src_taxa_id, dst_taxa_id)`. (The column names keep the older `taxa` spelling.) No content is loaded yet.

## taxon_function

Ecosystem functions that taxa can be assigned to, matching the FAPROTAX and FUNGuild categories used in the AI4SH eDNA indicators:

| name | abstract |
|---|---|
| chemoheterotrophy | gets both energy and carbon by consuming pre-formed organic compounds |
| human pathogens all | recognised human pathogens, usually from eDNA analysis |
| nitrogen fixation | converts atmospheric nitrogen gas into plant-accessible nitrogen compounds |
| ectomycorrhizal fungi | symbiotic fungi forming a sheath around root tips |
| arbuscular mycorrhizal fungi | fungi in close mutual partnership with plant roots |
| saprotrophs | decompose dead and decaying organic matter |
| plant pathogens | infect plants and cause disease |

Taxon functions are not yet linked to individual taxa or ASVs; the functional values currently arrive as summary indicators per sample.

## Taxonomy used by other schemas

| Table | Column | References |
|---|---|---|
| `observation.macrofauna` | `taxon_id` | `organism_utility.taxon` |
| `observation.edna_asv` | `taxon_id` | `organism_utility.taxon` (deepest resolved rank) |
| `observation_utility.method_pipeline_step` | `taxonomy_reference_id` | `organism_utility.taxonomy_reference` |

Earlier versions of the database had duplicate taxonomy tables in `observation_utility` (`taxa`, `taxa_level`, `taxa_status`, `taxa_function`). These have been removed; `organism_utility` is the single source of taxonomy.

[setup_db_observation_utility]: /setup_db/observation_utility/
[setup_process_organism_utility]: /setup_process/organism_utility/
[edna_catalogues]: /edna/edna_catalogues/
[edna_insert_taxon]: /edna/insert_taxon/
