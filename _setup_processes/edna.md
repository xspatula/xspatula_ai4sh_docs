---
title: "eDNA Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "eDNA processes register operations for managing environmental DNA observations, including the method catalogues for extraction, amplification, purification, sequencing and metabarcoding."
permalink: /setup_process/edna/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

eDNA processes register the operations for managing data in the `edna_utility` and `edna` schemas. Environmental DNA (eDNA) metabarcoding is used in the AI4SH project to characterise soil biodiversity by sequencing DNA extracted directly from soil samples.

## Process files

The eDNA process files are located at:

```
./setup/zzz/ai4sh/setup_processes/json_ai4sh/edna/
```

Process files in this directory register operations for:

- Managing eDNA method reference catalogues (`edna_utility` tables)
- Managing eDNA observations (`edna` tables)

## eDNA utility catalogue processes

These processes manage the reference tables that describe the eDNA analytical pipeline. Each must be populated before eDNA observations that reference them can be entered.

| Catalogue | Description |
|---|---|
| Extraction method | Protocol for extracting DNA from a soil sample (e.g. PowerSoil kit, CTAB method) |
| Amplification (PCR) | PCR protocol including primers and cycling conditions |
| Purification | Clean-up procedure for extracted or amplified DNA |
| Sequencing platform | Sequencing technology (e.g. Illumina MiSeq, Oxford Nanopore) |
| Metabarcoding protocol | Amplicon-based protocol for profiling community composition |

All catalogue management processes for eDNA utility require a minimum user stratum of 4.

## eDNA observation processes

eDNA observations use the same sample infrastructure as physicochemical observations. The soil sample is registered through the standard `observation` schema workflow (dataset → campaign → sampling log → sample), and the eDNA observation then references that sample along with the specific eDNA workflow records.

A complete eDNA observation registers:

1. The source sample (linked to the standard observation schema)
2. The extraction method applied to the sample
3. The amplification protocol used
4. The purification method
5. The sequencing platform and run details
6. The metabarcoding protocol and taxonomic assignment method
7. The detected taxa with their abundances (linked to `observation_utility.taxa`)

eDNA observation management processes require a minimum user stratum of 3.

## Integration with observation_utility.taxa

Detected taxa in eDNA results are stored using the Linnean classification hierarchy in `observation_utility` (`order`, `family`, `genus`, `species`, `specimen`). Before eDNA observations can be inserted, the relevant taxa must be registered via the observation utility `manage_classification` process. This links eDNA biodiversity results to the same taxonomic framework used for macrofauna and other biological observations, enabling cross-method biodiversity analyses from the same database.

## Access level

- eDNA utility catalogue management: minimum stratum 4
- eDNA observation management: minimum stratum 3
