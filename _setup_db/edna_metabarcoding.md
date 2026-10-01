---
title: "eDNA Metabarcoding"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "How eDNA metabarcoding works and how xspatula_ai4sh stores it: method catalogues and pipelines in observation_utility, the taxon tree in organism_utility, and ASVs, abundances and summary indicators in observation."
permalink: /setup_db/edna_metabarcoding/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

eDNA metabarcoding tells you *which* microorganisms live in a soil sample and *how abundant* they are, without growing or seeing any of them. This page explains the method, then how `xspatula_ai4sh` stores both the results and the full provenance of how they were produced. How to load the data is covered in the [eDNA metabarcoding loading collection][edna].

## What is eDNA metabarcoding?

Soil is full of DNA — from living cells, dead cells and free fragments. Metabarcoding extracts all of it at once, then copies (amplifies) one short, standardised stretch of a single gene — the *barcode* — from every organism in the mix. Sequencing those copies and comparing them to a reference database tells you who was there.

An analogy: think of a crowded stadium where everyone wears a shirt with a printed barcode. You don't count people; you photocopy every barcode you can find, feed the copies through a scanner, and look each code up in a register. The number of copies of each code is a proxy for how many people carried it.

AI4SH uses two barcodes, and therefore two pipelines:

| Organism group | Marker gene (region) | Primer pair | Reference database | Functional prediction | Pipeline |
|---|---|---|---|---|---|
| Prokaryotes (bacteria and archaea) | 16S rRNA (V4) | 515F modified / 806R modified | SILVA | FAPROTAX | `ai4sh-16s` |
| Fungi | ITS (ITS2) | gITS7 / ITS4 | UNITE | FUNGuild | `ai4sh-its` |

### ASVs rather than OTUs

After sequencing, reads are cleaned and grouped. Older pipelines cluster near-identical reads into **OTUs** (operational taxonomic units, typically at 97 % similarity). AI4SH uses **ASVs** — amplicon sequence variants — inferred by DADA2. An ASV is an exact sequence, so the same ASV found in two studies is the same biological variant. Each ASV is then annotated with a taxonomic lineage, e.g.

```
k__Fungi|p__Ascomycota|c__Sordariomycetes|o__Hypocreales|f__Nectriaceae|g__Fusarium|s__
```

Ranks (taxonomic hierarchical levels) the reference database cannot resolve are left empty (`s__` above), so the deepest resolved rank varies between ASVs.

### Rarefaction

Samples yield different numbers of reads. To compare them, each sample is *rarefied* — randomly subsampled — to the same depth, here **20,000 reads**. The lab delivers relative abundances after rarefaction; the database stores both the relative abundance and the exact rarefied count (`rel_abundance × 20000`). The raw, unrarefied counts are preferable for modern statistics and have a column waiting for them (`raw_read_count`).

### The 12 pipeline steps

Both pipelines have the same 12 steps; only the primers, reference database and functional tool differ.

| Step | Stage | Name | Protocol / primer / software / reference |
|---|---|---|---|
| 1 | laboratory | extraction | `zymobiomics dna miniprep` (ZymoBIOMICS DNA Miniprep Kit) |
| 2 | laboratory | amplification | `pcr amplification` + primer pair (`515f-806r modified` or `gits7-its4`) |
| 3 | laboratory | purification | `magnetic bead purification` |
| 4 | laboratory | sequencing | `illumina pe250` (Illumina, 2×250 bp paired-end, ≥ 30k tags) |
| 5 | bioinformatics | demultiplexing and primer trimming | — |
| 6 | bioinformatics | paired-end merging | FLASH |
| 7 | bioinformatics | quality filtering | fastp |
| 8 | bioinformatics | chimera removal | vsearch |
| 9 | bioinformatics | ASV inference | DADA2 |
| 10 | bioinformatics | taxonomic annotation | QIIME2 2024.10 + SILVA or UNITE |
| 11 | bioinformatics | functional prediction | FAPROTAX or FUNGuild |
| 12 | bioinformatics | diversity indices | QIIME2 2024.10, rarefaction depth 20,000 |

Two design choices are worth knowing:

- **Forward and reverse primers are one step.** PCR uses both primers in the same reaction, so amplification is a single step that references a primer *pair*.
- **Demultiplexing is bioinformatics, not lab work.** The sequencer outputs one pooled file; splitting it into samples by their barcodes (and trimming primers) is the first computational step.

## How xspatula_ai4sh stores eDNA

The central idea is to keep the **method** apart from the **results**. The method is the same for every sample and is stored once, in the utility schemas. The results are per sample and stored in `observation`. Every result row points back to its pipeline, so the full methods section of a paper can be reproduced by a query.

| Schema | Table | Holds | Page |
|---|---|---|---|
| `observation_utility` | `software` | Software name and version (e.g. `qiime2 2024.10`) | [Observation utility][setup_db_observation_utility] |
| `observation_utility` | `edna_primer_pair` | Forward + reverse primers, marker gene, target group | [Observation utility][setup_db_observation_utility] |
| `observation_utility` | `lab_protocol` | Extraction, amplification, purification, sequencing protocols | [Observation utility][setup_db_observation_utility] |
| `observation_utility` | `method_pipeline` | Named pipeline (`ai4sh-16s`, `ai4sh-its`), linked to an `analysis_method` | [Observation utility][setup_db_observation_utility] |
| `observation_utility` | `method_pipeline_step` | The 12 ordered steps with protocol, primer pair, software, taxonomy reference and JSONB parameters | [Observation utility][setup_db_observation_utility] |
| `organism_utility` | `taxonomy_reference` | SILVA, UNITE (name + version) | [Organism utility][setup_db_organism_utility] |
| `organism_utility` | `taxon`, `taxon_rank`, … | The taxon tree built from the delivered lineages | [Organism utility][setup_db_organism_utility] |
| `observation` | `observation_measurement` | The 19 summary indicators per sample (richness, Shannon, Simpson, Pielou, Chao1, functional groups) | [Observation][setup_db_observation] |
| `observation` | `edna_asv` | One row per ASV and pipeline, linked to its deepest resolved taxon | [Observation][setup_db_observation] |
| `observation` | `edna_asv_abundance` | Relative abundance and rarefied read count per ASV and observation (non-zero only) | [Observation][setup_db_observation] |
| `observation` | `edna_run_step` | Reads in/out per bioinformatics step per observation (empty until delivered) | [Observation][setup_db_observation] |

### How the tables connect

[![How the eDNA tables connect]({{ "/assets/media/edna/edna_overview.png" | relative_url }})]({{ "/assets/media/edna/edna_overview.png" | relative_url }})

The link to the ordinary indicator machinery runs through `analysis_method`: provision `ai4sh-metabarcoding` delivers 19 indicators via `provision_indicator`; 9 prokaryote indicators use analysis method `ai4sh 16s metabarcoding`, 10 fungal indicators use `ai4sh its metabarcoding`, and each analysis method has exactly one `method_pipeline`.

### Setup order

In the pilot file `db_setup.txt` the eDNA tables are split over three sections, dictated by foreign keys:

1. **OBSERVATION UTILITIES** (end): `software`, `edna_primer_pair`, `lab_protocol`
2. **ORGANISM UTILITIES**: `taxonomy_reference`, `taxon_rank`, `taxon_status`, `taxon`, `taxon_parity`, `taxon_function`
3. **METHOD PIPELINES**: `observation_utility/method_pipeline_v10_sql.json` — after organism utilities, because steps reference `taxonomy_reference`
4. **OBSERVATION** (after `measurement`): `edna_asv` (with `edna_asv_abundance`), then `edna_run_step`

The setup files create **structure only**. All content — pipelines, steps, software, primers, protocols, taxonomy references — is inserted from Excel through the utility insert chain; see [eDNA method catalogues][edna_catalogues].

### Example: the methods section as a query

```sql
SELECT s.step, s.stage, s.name, sw.alias AS software, tr.alias AS reference, s.parameters
FROM observation_utility.method_pipeline_step s
JOIN observation_utility.method_pipeline p ON p.id = s.method_pipeline_id
LEFT JOIN observation_utility.software sw ON sw.id = s.software_id
LEFT JOIN organism_utility.taxonomy_reference tr ON tr.id = s.taxonomy_reference_id
WHERE p.name = 'ai4sh-16s'
ORDER BY s.step;
```

## Known limitations

- **No sequences yet.** The lab delivered lineages and abundances but not the ASV sequences (`rep-seqs.fasta`). `edna_asv.sequence` and `sequence_md5` are empty.
- **ASV keys are row numbers.** Without sequences, each ASV is keyed `<group>-<row>` (e.g. `fungi-000001`) from its row in the delivered sheet. The key is only stable as long as the lab keeps the row order; the ASV loader aborts, writing nothing, if an existing key arrives with a different lineage.
- **Rarefied counts only.** `raw_read_count` stays empty until the lab delivers the unrarefied feature table.
- **`edna_run_step` is empty** until the lab delivers per-step read counts (e.g. the DADA2 denoising statistics).
- **Step parameters are placeholders.** Most bioinformatics steps list their parameter names with `null` values (e.g. `min_phred_score`, `trunc_len_forward`) until the lab reports them.
- **Functional assignments per taxon** are not stored; functional groups are available only as the summary indicators per sample.

[edna]: /edna/
[edna_catalogues]: /edna/edna_catalogues/
[setup_db_observation_utility]: /setup_db/observation_utility/
[setup_db_organism_utility]: /setup_db/organism_utility/
[setup_db_observation]: /setup_db/observation/
