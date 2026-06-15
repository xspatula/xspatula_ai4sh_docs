---
title: "Manage Persons"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Inserting person records into the AI4SH database. Persons are linked to a data source and referenced by campaigns and sampling logs."
permalink: /dataset_meta/manage_person/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Person records register the named individuals responsible for data collection and analysis. Each person is linked to a data source (their affiliated organisation) and may be referenced as a contact in campaigns and sampling logs.

## Prerequisites

- [Manage data source] must be complete — persons reference a data source via `data_source_id__data_source_name`.
- `utility.territory` must be populated (from [Load utility data]).

## Notebook cell

In `load_ai4sh_dataset_meta.ipynb`, the **Manage persons** cell runs:

```python
process_file = 'import_data/dataset/manage_process/manage_person.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/dataset/manage_process/manage_person.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_person",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "data_source_id__data_source_name": "esdac-jrc",
        "first_name": "fn",
        "middle_name": "mn",
        "last_name": "ln",
        "email": "ec-esdac@jrc.ec.europa.eu",
        "territory_id__territory_name": "eu"
      }
    }
  ]
}
```

One JSON object is generated per row in `person.xlsx`. Replace the placeholder names (`fn`, `mn`, `ln`) with actual person details in the source Excel file before translating.

## Parameters

| Parameter | Description |
|---|---|
| `data_source_id__data_source_name` | FK lookup: data source by alias (must exist) |
| `first_name` | Given name |
| `middle_name` | Middle name (optional; use empty string if absent) |
| `last_name` | Family name |
| `email` | Contact email |
| `territory_id__territory_name` | FK lookup: territory by name |

## Next step

Proceed to [Manage dataset].

[Manage data source]: /dataset_meta/manage_data_source/
[Load utility data]: /utility/
[Manage dataset]: /dataset_meta/manage_dataset/
