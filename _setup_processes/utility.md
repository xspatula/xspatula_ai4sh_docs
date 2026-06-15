---
title: "Utility Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Utility schema processes register operations for managing foreign key relations and territory records in the utility schema. Both processes require a high user stratum as they affect core framework reference data."
permalink: /setup_process/utility/
author_profile: false
date: 2026-04-07 08:00:00 +0200
last_modified_at: 2026-04-07 08:00:00 +0200
---

The utility process files register operations for managing reference data in the `utility` schema. The two processes registered here give authorised users the ability to manage foreign key definitions and territory records — both of which underpin referential integrity and geographic attribution across the whole database.

## Process files

| File | Process registered | Target table | Min stratum |
|---|---|---|---|
| `utility/foreign_key_v10_sql.json` | `manage_foreign_key` | `utility.foreign_key` | 5 |
| `utility/territory_v10_sql.json` | `manage_territory` | `utility.territory` | 5 |

## manage_foreign_key

Registers, updates, or deletes a foreign key relation in `utility.foreign_key`. This table tracks inter-schema foreign key relationships used by the framework for dynamic lookups. Parameters:

| Parameter | Type | Required | Description |
|---|---|---|---|
| `foreign_key` | text | yes | Name of the foreign key |
| `dst_schema` | text | yes | Schema of the referenced (destination) table |
| `dst_table` | text | yes | Table name of the referenced (destination) table |
| `dst_search_column` | text | yes | Primary search column in the destination table |
| `dst_alt_search_column` | text | yes | Alternative search column in the destination table |

The `foreign_key` name and all destination references are immutable after initial insertion — neither can be updated or deleted once registered.

## manage_territory

Registers, updates, or deletes a territory record in `utility.territory`. Territories follow the ISO 3166 naming convention by default but the framework accepts non-standard codes where needed. Parameters:

| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | text | yes | Full territory name |
| `iso_code_a2` | text | yes | Two-letter ISO code (e.g. `SE`, `EU`) |
| `iso_code_a2_ext` | text | yes | Extended two-letter code for sub-national or custom territories |

The `name` is immutable after insertion. The ISO codes can be updated but the territory cannot be deleted once referenced by other records.

## Access level

Both processes require a minimum user stratum of 5. This is the highest operational stratum, reflecting that changes to foreign key definitions and territory records affect the referential integrity of the entire database.
