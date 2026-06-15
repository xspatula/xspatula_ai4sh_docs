---
title: "Manage Data Source"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Inserting data source records into the AI4SH database. A data source is the organisation or individual who owns and provides the data."
permalink: /dataset_meta/manage_data_source/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

A data source is the organisation or individual that owns and provides the data — a research institute, a government agency, a private laboratory, or a private person. Every dataset must reference a data source, so this is the first manage step.

## Prerequisite

Run [Translate dataset] first. The `utility.territory` table must be populated (from [Load utility data]).

## Notebook cell

In `load_ai4sh_dataset_meta.ipynb`, the **Manage data source** cell runs:

```python
process_file = 'import_data/dataset/manage_process/manage_data_source.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

The cell calls `process_file` directly rather than a `job_file` — there is one manage file per dataset metadata table.

## Manage process file

**Path**: `./ai4sh/import_data/dataset/manage_process/manage_data_source.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_data_source",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "name": "european soil data centre (esdac), european commission, joint research centre jrc)",
        "alias": "esdac-jrc",
        "display_name": "ESDAC-JRC",
        "url": "https://esdac.jrc.ec.europa.eu",
        "territory_id__territory_name": "eu",
        "contact_name": "esdac - european commissiony",
        "contact_email": "ec-esdac@jrc.ec.europa.eu"
      }
    }
  ]
}
```

One JSON object is generated per row in `data_source.xlsx`. The `alias` field becomes the lookup name referenced by the `__` notation in downstream manage files (e.g. `"data_source_id__data_source_name": "esdac-jrc"`).

## Parameters

| Parameter | Description |
|---|---|
| `name` | Full name of the data source (lowercase) |
| `alias` | Short identifier used for FK lookup in other manage files |
| `display_name` | Display label |
| `url` | Organisation URL |
| `territory_id__territory_name` | FK lookup: territory by name (must exist in `utility.territory`) |
| `contact_name` | Primary contact name |
| `contact_email` | Primary contact email |

## Next step

Proceed to [Manage persons].

[Translate dataset]: /dataset_meta/translate_dataset/
[Load utility data]: /utility/
[Manage persons]: /dataset_meta/manage_person/
