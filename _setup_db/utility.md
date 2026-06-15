---
title: "Utility Schema"
layout: single
sidebar:
  nav: "setup_db"
excerpt: "The utility schema holds general support tables used across the entire AI4SH database. It is a default Xspatula framework schema and is required by the community schema."
permalink: /setup_db/utility/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The `utility` schema holds general support tables shared across the entire database. It is one of the three default Xspatula framework schemas and must be populated before the `community` schema, because the `community.user` table references `utility.territory`.

## Process files

Two process files create the utility schema content:

| File | Purpose |
|---|---|
| `utility/utility_v10_sql.json` | Creates the utility tables |
| `utility/utility_territory_v10_sql.json` | Inserts territory reference records |

## Tables

### territory

The `territory` table holds country/territory codes used as reference data for users and observations. It provides a standardised way to associate any record with a geographic territory without duplicating geographic metadata in every table.

Key columns:

| Column | Type | Description |
|---|---|---|
| `id` | SERIAL | Primary key |
| `name` | TEXT | territory name in small letters |
| `display_name` | TEXT | Territory name to display |
| `iso_code_a2` | TEXT | Territory iso code (or equivalent) |
| `iso_code_a2_ext` | TEXT | Territory iso code or extended iso-code (or equivalent) |

The `utility_territory_v10_sql.json` file pre-loads standard territory records so that the community user table can immediately reference valid territory IDs.

### foreign_key

When a process requires a foreign key (fk), the default parameter for identifying a foreign key includes a double underscore:

```
source_table_name__fk-table_fk-column
```

For example, to find the foreign key for the column _dataset_id_, the parameter:

```
"dataset_id__dataset_name": "AI4SH"
```

will search the table _dataset_ column _name_ for the value _AI4SH_ and return the _id_ of that records, and the _id_ will be used as the foreign key. This syntax is not possible to use in all cases, for instance when a process requires more than one fk from the same destination table.

The `foreign_key` table holds information on which schema, table and column to use for searching foreign keys that can not be resolved with the default syntax. Most foreign keys are constructed such that the name of the parameter reveals the foreign key to look for, but there are exceptions. For these exceptions, the python scripts looks for the correct foreign key in the utility.foreign_key table.

Key columns:

| Column | Type | Description |
|---|---|---|
| `foreign_key` | TEXT | Primary key |
| `dst_schema` | TEXT | The schema where the foreign key is found |
| `dst_table` | TEXT | The table where the foreign key is found |
| `dst_search_column` | TEXT | The column where the foreign key is found |
| `dst_alt_search_column` | TEXT | Alternative columns where the foreign key is found |

## Role in the database

The utility schema is referenced by:

- `community.user` — links users to a territory
- `observation.campaign_location` — geographic extent of a campaign
- Potentially any other table that needs geographic context
- Any process where the foreign_key definition does not adhere to standard syntax

For the full table specification of the default utility schema, see the [core framework documentation][setup_core_db_docs_schemas].


[setup_core_db_docs_schemas]: https://xspatula.github.io/setup_core_db_docs/setup_db/schemas_tables/
