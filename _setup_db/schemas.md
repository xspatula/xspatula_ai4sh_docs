---
title: "AI4SH Database Schemas"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The AI4SH database is organised into 10 postgreSQL schemas. The schema process file must be run first — before any tables — to create all schema namespaces."
permalink: /setup_db/schemas/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

The AI4SH database is organised into 10 postgreSQL schemas. Schemas group related tables into logical namespaces and allow different access permissions to be applied per schema. The schema process file must be executed first in the pilot file — before any table definitions — because table creation requires the target schema to already exist.

## The schema process file

**File**: `./setup/zzz/ai4sh/setup_db/json/schema/schema_v10_sql.json`

This single file creates all 10 schemas using repeated calls to the `create_schema` process:

```json
{
  "process": [
    { "process_id": "create_schema", "parameters": { "schema": "utility" } },
    { "process_id": "create_schema", "parameters": { "schema": "community" } },
    { "process_id": "create_schema", "parameters": { "schema": "process" } },
    { "process_id": "create_schema", "parameters": { "schema": "audit" } },
    { "process_id": "create_schema", "parameters": { "schema": "landscape_utility" } },
    { "process_id": "create_schema", "parameters": { "schema": "landscape" } },
    { "process_id": "create_schema", "parameters": { "schema": "organism_utility" } },
    { "process_id": "create_schema", "parameters": { "schema": "organism" } },
    { "process_id": "create_schema", "parameters": { "schema": "observation_utility" } },
    { "process_id": "create_schema", "parameters": { "schema": "observation" } }
  ]
}
```

## Schema overview

| Schema | Type | Purpose |
|---|---|---|
| `utility` | Framework default | General support catalogues shared across schemas (territory etc.) — see [Utility][setup_db_utility] |
| `community` | Framework default | Organisations and users; manages all database access — see [Community][setup_db_community] |
| `process` | Framework default | Process definitions and parameter specifications — see [Process][setup_db_process] |
| `audit` | Framework default | Audit when and by whom data in core tables were changed — see [Auditing][auditing] |
| `landscape_utility` | AI4SH | Reference catalogues for landscape classification — see [Landscape][setup_db_landscape] |
| `landscape` | AI4SH | Landscape observations — see [Landscape][setup_db_landscape] |
| `organism_utility` | AI4SH | Biological reference data: the taxon tree, taxon ranks, status, functions and the taxonomy reference databases (e.g. SILVA, UNITE) — see [Organism utility][setup_db_organism_utility] |
| `organism` | AI4SH | Reserved for organism-level observations; no tables in the setup pilot yet — see [Organism][setup_db_organism] |
| `observation_utility` | AI4SH | Reference catalogues for FAIR-compliant soil data (units, methods, instruments, software, eDNA primer pairs, lab protocols, method pipelines, etc.) — see [Observation utility][setup_db_observation_utility] |
| `observation` | AI4SH | Actual soil property data (datasets, campaigns, samples, observations, eDNA ASVs and abundances) — see [Observation][setup_db_observation] |

The four **framework default** schemas (`utility`, `community`, `process`, `audit`) are created for every Xspatula database, not just AI4SH. Their table structure is the same as described in the [core framework documentation][setup_core_db_docs_schemas]. The remaining schemas are specific to the AI4SH project.

## Schema dependencies

Tables across schemas reference each other using foreign keys, for instance:

```
utility ← community ← process
utility ← observation_utility ← observation
          organism_utility    ← observation   (macrofauna, edna_asv → taxon)
          organism_utility    ← observation_utility.method_pipeline_step (→ taxonomy_reference)
landscape_utility ← landscape
```

This is reflected in the execution order of the pilot file — utility and community tables are created before observation_utility, organism_utility before the method pipeline tables (which reference `organism_utility.taxonomy_reference`), and all utility schemas before the observation tables. Note that the method pipeline tables live in `observation_utility` but are created *after* `organism_utility` — see [eDNA metabarcoding][setup_db_edna].


[setup_core_db_docs_schemas]: https://xspatula.github.io/setup_core_db_docs/setup_db/schemas_tables/
[setup_db_observation_utility]: /setup_db/observation_utility/
[setup_db_observation]: /setup_db/observation/
[setup_db_organism_utility]: /setup_db/organism_utility/
[setup_db_organism]: /setup_db/organism/
[setup_db_edna]: /setup_db/edna_metabarcoding/
[setup_db_utility]: /setup_db/utility/
[setup_db_community]: /setup_db/community/
[setup_db_process]: /setup_db/process/
[auditing]: /auditing/
[setup_db_landscape]: /setup_db/landscape/
