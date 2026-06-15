---
title: "eDNA Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The edna and edna_utility schemas support environmental DNA (eDNA) observations. Two schemas are used: edna_utility for method and workflow reference tables, and edna for actual eDNA observations."
permalink: /setup_db/edna/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

Environmental DNA (eDNA) analysis is a key method in the AI4SH project for characterising soil biodiversity. Two schemas handle this data: `edna_utility` holds the method and workflow reference catalogues, and `edna` holds the observations.

## Process files

| File | Schema | Purpose |
|---|---|---|
| `edna/edna_utility_v10_sql.json` | `edna_utility` | Creates eDNA method and workflow reference tables |
| `edna/edna_observation_v10_sql.json` | `edna` | Creates eDNA observation tables |

## Schema: edna_utility

The `edna_utility` schema holds reference tables for the specific methods and reagents used in eDNA workflows. eDNA processing involves several distinct steps, each with its own catalogue:

| Reference table | Description |
|---|---|
| Extraction method | Method for extracting DNA from a soil sample |
| Amplification (PCR) | PCR amplification protocol |
| Purification method | Clean-up of extracted or amplified DNA |
| Sequencing platform | Sequencing technology used (e.g. Illumina, Nanopore) |
| Metabarcoding protocol | Protocol for amplicon-based community profiling |

Each reference table must be populated before eDNA observations can be entered.

## Schema: edna

The `edna` schema stores actual eDNA observations. An eDNA observation in the AI4SH context typically represents a metabarcoding result: the detected presence, abundance, or diversity of taxa in a soil sample identified via DNA sequencing.

eDNA observations link back to:

- The soil `sample` from which DNA was extracted (via `observation` schema infrastructure)
- `edna_utility` reference records describing the extraction, amplification, and sequencing workflow
- `observation_utility.taxa` for the detected organisms (using the Linnean classification hierarchy)

## Relationship to other schemas

eDNA observations use the same sample infrastructure as physicochemical soil observations — the same `sampling_log`, `sample`, and campaign hierarchy applies. What differs is the analytical pipeline: instead of a chemical or spectral measurement, the result is a sequence-based taxon detection. The `edna_utility` schema captures the specific reagents and protocols of that pipeline in a structured, queryable form.

This design allows combining eDNA biodiversity results with physicochemical soil properties from the same sample, which is a core analytical goal of the AI4SH project.
