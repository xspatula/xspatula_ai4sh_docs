---
title: "AI4SH Database Schemas"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The AI4SH database is organised into 9 postgreSQL schemas. The schema process file must be run first — before any tables — to create all schema namespaces."
permalink: /setup_db/schemas/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The AI4SH database is organised into 9 postgreSQL schemas. Schemas group related tables into logical namespaces and allow different access permissions to be applied per schema. The schema process file must be executed first in the pilot file — before any table definitions — because table creation requires the target schema to already exist.

## The schema process file

**File**: `./setup/zzz/ai4sh/setup_db/json_ai4sh/schema/schema_v10_sql.json`

This single file creates all 9 schemas using repeated calls to the `create_schema` process:

```json
{
  "process": [
    { "process_id": "create_schema", "parameters": { "schema": "utility" } },
    { "process_id": "create_schema", "parameters": { "schema": "community" } },
    { "process_id": "create_schema", "parameters": { "schema": "process" } },
    { "process_id": "create_schema", "parameters": { "schema": "landscape_utility" } },
    { "process_id": "create_schema", "parameters": { "schema": "landscape" } },
    { "process_id": "create_schema", "parameters": { "schema": "observation_utility" } },
    { "process_id": "create_schema", "parameters": { "schema": "observation" } },
    { "process_id": "create_schema", "parameters": { "schema": "edna_utility" } },
    { "process_id": "create_schema", "parameters": { "schema": "edna" } }
  ]
}
```

## Schema overview

| Schema | Type | Purpose |
|---|---|---|
| `utility` | Framework default | General support tables shared across schemas (territory etc.) |
| `community` | Framework default | Organisations and users; manages all database access |
| `process` | Framework default | Process definitions and parameter specifications |
| `landscape_utility` | AI4SH | Reference tables for landscape classification |
| `landscape` | AI4SH | Landscape observations |
| `observation_utility` | AI4SH | Reference catalogues for FAIR-compliant soil data (units, methods, instruments, taxa, etc.) |
| `observation` | AI4SH | Actual soil property data (datasets, campaigns, samples, observations) |
| `edna_utility` | AI4SH | Reference tables for environmental DNA workflows |
| `edna` | AI4SH | eDNA observations |

The three **framework default** schemas (`utility`, `community`, `process`) are created for every Xspatula database, not just AI4SH. Their table structure is the same as described in the [core framework documentation][setup_core_db_docs_schemas]. The remaining six schemas are specific to the AI4SH project.

## Schema dependencies

Tables across schemas reference each other using foreign keys, for instance:

```
utility ← community ← process
utility ← observation_utility ← observation
landscape_utility ← landscape
edna_utility ← edna
```

This is reflected in the execution order of the pilot file — utility and community tables are created before observation_utility, and observation_utility tables are created before observation tables.


[setup_core_db_docs_schemas]: https://xspatula.github.io/setup_core_db_docs/setup_db/schemas_tables/
