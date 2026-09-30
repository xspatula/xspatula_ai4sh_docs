---
title: "Organism Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "The organism schema is reserved; no processes are registered for it yet."
permalink: /setup_process/organism/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

No processes are registered for the `organism` schema. There is no `organism/` folder under `./setup/zzz/ai4sh/setup_processes/json/`, and `setup_processes.txt` has no organism section.

This matches the database side: the schema is created, but it holds no tables — see [Organism schema][setup_db_organism].

Organism-related data is currently handled by processes in other schemas:

| Data | Process | Page |
|---|---|---|
| Taxonomy reference databases | `manage_taxonomy_reference` | [Organism utility processes][setup_process_organism_utility] |
| Taxon tree | `manage_taxon` (bulk) | [Organism utility processes][setup_process_organism_utility] |
| eDNA ASVs and abundances | `manage_edna_asv` (bulk) | [Observation processes][setup_process_observation] |

[setup_db_organism]: /setup_db/organism/
[setup_process_organism_utility]: /setup_process/organism_utility/
[setup_process_observation]: /setup_process/observation/
