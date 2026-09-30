---
title: "eDNA Method Catalogues"
layout: single
sidebar:
  nav: "edna"
  nav2: "loading_data"
excerpt: "How to fill in the Excel catalogues that describe the eDNA method — software, taxonomy references, primer pairs, lab protocols, method pipelines and their steps — loaded through the utility insert chain."
permalink: /edna/edna_catalogues/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

The eDNA *method* is described once, in six Excel catalogues, before any results are loaded. Together they reproduce the methods section of a paper as data. Every ASV loaded later points to a pipeline, and the pipeline's steps point to the kit, the primers, the software version and the reference database that were used.

These catalogues are the part of the eDNA workflow you are most likely to edit by hand, for example when the laboratory reports a software version or a parameter value. This page explains each sheet column by column.

## Where they are and how they are loaded

All files are in:

```
./ai4sh/import_data/utility/observation/excel/
```

They are inserted by the utility insert chain, **not** by the eDNA notebooks. `insert_ai4sh_utility_data.ipynb` runs the job `utility/job_insert_observation_utility.json`, whose pilot file `insert_observation_utility.txt` ends with an `EDNA UTILITIES` section:

```
software.json
taxonomy_reference.json
edna_primer_pair.json
lab_protocol.json
method_pipeline.json
method_pipeline_step.json
```

Keep this order: `method_pipeline_step` references all the others. Each entry is an `insert_tabular_data` job in `utility/observation/insert_process/`. A dual-step version (`translate_tabular_data`) of each is in `utility/observation/process/`. All sheets are read from `Sheet1`.

**Insert-only.** Rerunning the chain adds new rows and never changes existing ones. To change a value already in the database (e.g. a version), use SQL `UPDATE`. See [Changing values later](#changing-values-later).

## The alias rule: `<name> <version>`

`software` and `taxonomy_reference` identify a row by **name and version together**. Their `alias` column must be the name and version joined by one space:

| name | version | alias |
|---|---|---|
| `qiime2` | `2024.10` | `qiime2 2024.10` |
| `flash` | `unspecified` | `flash unspecified` |
| `silva` | `unspecified` | `silva unspecified` |

Pipeline steps reference software and taxonomy references **by alias**. If a version is changed in one sheet but not in the other, the step lookup fails.

Use `unspecified` when the version is not known — never leave it blank. A blank version is stored as NULL. PostgreSQL treats every NULL as distinct, so each rerun would insert a duplicate.

## software.xlsx

| Column | Required | Example | Notes |
|---|---|---|---|
| `name` | yes | `qiime2` | Lower case |
| `version` | no (default `unspecified`) | `2024.10` | |
| `alias` | yes | `qiime2 2024.10` | Must follow the alias rule; unique |
| `url` | yes | project page | Case is preserved |
| `doi` | no | | Citation |
| `abstract` | no | | What the tool does |

AI4SH rows: `qiime2`, `flash`, `fastp`, `vsearch`, `dada2`, `faprotax`, `funguild`.

## taxonomy_reference.xlsx

| Column | Required | Example | Notes |
|---|---|---|---|
| `name` | yes | `silva` | |
| `version` | no (default `unspecified`) | `138.2` | Release of the reference database |
| `alias` | yes | `silva unspecified` | Alias rule; unique |
| `marker_gene` | yes | `16S rRNA` / `ITS` | |
| `url` | yes | | |
| `doi`, `abstract` | no | | |

AI4SH rows: `silva` (16S rRNA) and `unite` (ITS). The taxon load jobs name the reference by `name` + `version` — see [Insert taxa][insert_taxon].

## edna_primer_pair.xlsx

A forward and a reverse primer are used together in one PCR, so they are one record.

| Column | Required | Example |
|---|---|---|
| `name` | yes | `515f-806r modified` |
| `marker_gene` | yes | `16S rRNA` |
| `target_region` | yes | `V4` |
| `target_group` | yes | `prokaryotes` |
| `forward_name` | yes | `515F modified` |
| `forward_sequence` | yes | `GTGYCAGCMGCCGCGGTAA` |
| `reverse_name` | yes | `806R modified` |
| `reverse_sequence` | yes | `GGACTACNVGGGTWTCTAAT` |
| `doi`, `url`, `abstract` | no | |

Write sequences in **upper case** with IUPAC ambiguity codes (`Y`, `M`, `N`, `V`, `W`, `R` …). The import keeps them as written.

AI4SH rows: `515f-806r modified` (16S V4, prokaryotes) and `gits7-its4` (ITS2, fungi).

## lab_protocol.xlsx

| Column | Required | Example | Notes |
|---|---|---|---|
| `name` | yes | `zymobiomics dna miniprep` | Unique; referenced by pipeline steps |
| `protocol_type` | yes | `extraction` | One of `extraction`, `amplification`, `purification`, `library_preparation`, `sequencing`, `other` — anything else is rejected by the database |
| `kit` | no | `ZymoBIOMICS DNA Miniprep Kit` | |
| `manufacturer` | no | `Zymo Research` | |
| `platform` | no | `Illumina` | Sequencing platform |
| `strategy` | no | `2x250 bp paired-end` | |
| `data_requirement` | no | `30k tags` | Minimum sequencing depth |
| `abstract` | no | | |

AI4SH rows: `zymobiomics dna miniprep`, `pcr amplification`, `magnetic bead purification`, `illumina pe250`.

## method_pipeline.xlsx

| Column | Required | Example | Notes |
|---|---|---|---|
| `name` | yes | `ai4sh-16s` | Unique; used by the ASV load jobs |
| `version` | no (default `unspecified`) | | |
| `analysis_method_id__analysis_method_name` | yes | `ai4sh 16s metabarcoding` | Must exist in `analysis_method.xlsx` |
| `url`, `doi`, `abstract` | no | | |

AI4SH rows: `ai4sh-16s` → `ai4sh 16s metabarcoding`, and `ai4sh-its` → `ai4sh its metabarcoding`.

## method_pipeline_step.xlsx

One row per step per pipeline. AI4SH has 24 rows: 12 steps × 2 pipelines.

| Column | Required | Example | Notes |
|---|---|---|---|
| `method_pipeline_id__method_pipeline_name` | yes | `ai4sh-16s` | |
| `step` | yes | `6` | Integer, 1 = first; unique per pipeline |
| `stage` | yes | `bioinformatics` | `laboratory` or `bioinformatics` |
| `name` | yes | `paired-end merging` | |
| `lab_protocol_id__lab_protocol_name` | no | `illumina pe250` | Laboratory steps |
| `edna_primer_pair_id__edna_primer_pair_name` | no | `515f-806r modified` | The amplification step |
| `software_id__software_alias` | no | `flash unspecified` | **Alias with version** |
| `taxonomy_reference_id__taxonomy_reference_alias` | no | `silva unspecified` | The taxonomic annotation step; **alias with version** |
| `parameters` | no (default `{}`) | `{"min_overlap": null, "max_overlap": null}` | A JSON object as text |
| `abstract` | no | | |

A blank reference cell, or one reading `none`, means "not applicable". A laboratory step normally has only a lab protocol (plus a primer pair for amplification). A bioinformatics step normally has only software (plus a taxonomy reference for annotation).

### The parameters cell

`parameters` holds a JSON object written as text in one cell. It is stored as JSONB, so it can be queried later.

- Use `{}` when a step has no parameters.
- List each known parameter name with `null` when the value is not yet known, e.g. `{"min_phred_score": null, "min_length": null}`. The empty slots then show what to ask the laboratory for.
- Use double quotes, not single quotes. Write `true`/`false`/`null` in lower case. Lists are allowed: `{"indices": ["shannon", "simpson"], "rarefaction_depth": 20000}`.
- Excel may turn straight quotes into curly quotes (“ ”), which makes the JSON invalid. Turn off autocorrect for quotes, or check the cell.

Keep `"rarefaction_depth"` on the diversity step (12): it documents the depth that the rarefied `read_count` in `edna_asv_abundance` refers to.

## Related rows in other utility sheets

The eDNA results also depend on rows in the ordinary catalogues, loaded earlier in the same chain:

| Sheet | Row(s) |
|---|---|
| `analysis_method.xlsx` | `ai4sh 16s metabarcoding`, `ai4sh its metabarcoding` |
| `apparatus.xlsx`, `provider.xlsx` | `metabarcoding`, `ai4sh-metabarcoding` |
| `provision.xlsx` | `ai4sh-metabarcoding` (provider `ai4sh-metabarcoding`, apparatus `metabarcoding`, method tier `laboratory`) |
| `indicator.xlsx` | The 19 indicators; their `alias` values are the `@…` column names in the observation file (e.g. `prokaryotes alpha richness`) |
| `provision_indicator.xlsx` | 19 rows: provision `ai4sh-metabarcoding` → indicator → analysis method (9 × `ai4sh 16s metabarcoding`, 10 × `ai4sh its metabarcoding`) → unit `unitless` |
| `preservation.xlsx` | `dna/rna shield` (alias `dnash`) |

## Changing values later

Because the chain is insert-only, edit existing rows with SQL. Example: the laboratory confirms the SILVA release and the QIIME2 classifier settings.

```sql
UPDATE organism_utility.taxonomy_reference
   SET version = '138.2', alias = 'silva 138.2'
 WHERE alias = 'silva unspecified';

UPDATE observation_utility.method_pipeline_step s
   SET parameters = s.parameters || '{"confidence_threshold": 0.7}'
  FROM observation_utility.method_pipeline p
 WHERE p.id = s.method_pipeline_id AND p.name = 'ai4sh-16s' AND s.step = 10;
```

Make the same change in the Excel sheets, so that a database rebuilt from scratch gets the same values.

## Note on version control

`.gitignore` excludes `ai4sh/import_data/utility/`. The Excel files in this folder are tracked only because they were force-added (`git add -f`). New sheets you create there must be force-added too, or they will be missing from the repository.

[insert_taxon]: /edna/insert_taxon/
