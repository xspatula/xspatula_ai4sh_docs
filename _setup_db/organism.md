---
title: "Organism Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The organism schema is reserved for organism-level observations. It is created by the schema file but holds no tables in the current setup pilot."
permalink: /setup_db/organism/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

The `organism` schema is created by `schema/schema_v10_sql.json` together with the other nine schemas, but **no tables are created in it** by the current pilot file `db_setup.txt`. It is reserved for observations made on individual organisms (as opposed to soil samples), mirroring how `landscape` pairs with `landscape_utility`.

## Files in the organism folder

The folder `./setup/zzz/ai4sh/setup_db/json/organism/` holds one file:

| File | Table created | In pilot? |
|---|---|---|
| `observation_measurement_taxon_v10_sql.json` | `observation.taxon_observation_measurement` | No |

Note that this file creates its table in the **`observation`** schema, not in `organism`. The table is prepared for indicator values resolved per taxon — e.g. the abundance of a single genus in a sample reported as an indicator:

| Column | Notes |
|---|---|
| `observation_id` | → `observation.observation` |
| `indicator_id` | → `observation_utility.indicator` |
| `taxon_id` | → `organism_utility.taxon` |
| `value` | Double precision |

Constraint: `UNIQUE (observation_id, indicator_id, taxon_id)`.

Nothing in the current AI4SH loading chain writes to this table — eDNA community composition is stored per ASV in `observation.edna_asv_abundance` (see [Observation schema][setup_db_observation]). To create it anyway, add the file to `db_setup.txt` after the `observation/measurement_v10_sql.json` line and rerun `setup_db.ipynb`; existing tables are not touched.

For the biological reference data (taxon tree, ranks, functions), see [Organism utility][setup_db_organism_utility].

[setup_db_observation]: /setup_db/observation/
[setup_db_organism_utility]: /setup_db/organism_utility/
