---
title: "Utility Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Utility schema processes register operations for managing foreign key relations and territory records in the utility schema. Both processes require a high user stratum as they affect core framework reference data."
permalink: /setup_process/utility/
author_profile: false
date: 2026-04-07 08:00:00 +0200
last_modified_at: 2026-10-01 08:00:00 +0200
---

The utility process files register operations for managing reference data in the `utility` schema. The two processes registered here give authorised users the ability to manage foreign key definitions and territory records — both of which underpin referential integrity and geographic attribution across the whole database.

## Process files

| File | Process registered | Target table | Min stratum |
|---|---|---|---|
| `utility/foreign_key_v10_sql.json` | `manage_foreign_key` | `utility.foreign_key` | 5 |
| `utility/territory_v10_sql.json` | `manage_territory` | `utility.territory` | 5 |

## How Excel columns reach the database

Every tabular import — single-step `insert_tabular_data` or two-step translate + manage — works the same way. The Excel sheet does not say where its data goes; the **registered process** does. Each column header is a parameter name of the target process, and `process.process_parameter_schema_table` tells the framework which `schema.table` each parameter is written to.

### A single table without foreign keys

`territory.xlsx` loaded with `manage_territory`: every column maps to `utility.territory`, the row is checked against the table's unique columns, and a new record gets its `id` from the database.

[![Excel columns to utility.territory]({{ "/assets/media/process_mapping/territory.png" | relative_url }})]({{ "/assets/media/process_mapping/territory.png" | relative_url }})

### A single table with a foreign key

`organisation.xlsx` loaded with `manage_organisation` (see [Community processes][setup_process_community]). The territory is given by **name** in a column called `territory_id__territory_name`. The part before `__` is the target column; the framework confirms that the name exists in the referenced table and writes its `id` instead. This is also where `manage_foreign_key` comes in: when a column name does not match a table (e.g. `src_unit_id` and `dst_unit_id` in `unit_translate` → `unit`), the lookup is defined in `utility.foreign_key`.

[![Excel columns to community.organisation with a foreign key lookup]({{ "/assets/media/process_mapping/organisation.png" | relative_url }})]({{ "/assets/media/process_mapping/organisation.png" | relative_url }})

An example of a process writing to two tables, with the child table receiving the main table's new `id`, is shown under [manage_observation][setup_process_observation_manage].

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
| `display_name` | text | yes | Territory name to display |
| `iso_code_a2` | text | yes | Two-letter ISO code (e.g. `SE`, `EU`) |
| `iso_code_a2_ext` | text | yes | Extended two-letter code for sub-national or custom territories |

The `name` is immutable after insertion. The `display_name` and ISO codes can be updated but the territory cannot be deleted once referenced by other records.

{% capture notice-2 %}
## Access level

Both processes require a minimum user stratum of 5. This is the highest operational stratum, reflecting that changes to foreign key definitions and territory records affect the referential integrity of the entire database.
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

[setup_process_community]: /setup_process/community/
[setup_process_observation_manage]: /setup_process/observation/#manage_observation_log-and-manage_observation
