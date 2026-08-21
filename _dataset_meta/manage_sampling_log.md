---
title: "Manage Sampling Log"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Inserting sampling log records into the AI4SH database. A sampling log defines a specific field collection event — tools, dates, and contacts — within a campaign."
permalink: /dataset_meta/manage_sampling_log/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

A sampling log records a specific sampling operation within a campaign — the tools used, the date range, and the responsible contacts. Individual samples (with geolocations) are later linked to a sampling log.

## Prerequisites

- [Manage campaign] must be complete.

## Notebook cell

In `load_ai4sh_dataset_meta.ipynb`, the **Manage sampling logs** cell runs:

```python
process_file = 'import_data/dataset/manage_process/manage_sampling_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/dataset/manage_process/manage_sampling_log.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_sampling_log",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "campaign_id__campaign_name": "ai4sh_se_loennstorp",
        "contact_name": "thomasg",
        "contact_email": "thomas.gumbricht@natgeo.us.se",
        "name": "ai4sh_se_loennstorp_2024",
        "abstract": "AI4SH in-situ sampling",
        "sampling_tool_array": "spade,auger",
        "begin_date": 20240910,
        "end_date": 20240913
      }
    }
  ]
}
```

One JSON object is generated per row in `sampling_log.xlsx`. The file typically contains one entry per field campaign event; a large project may have many.

## Parameters

| Parameter | Description |
|---|---|
| `campaign_id__campaign_name` | FK lookup: parent campaign by name |
| `contact_name` | Name of the person responsible for this sampling operation |
| `contact_email` | Email of the responsible person |
| `name` | Sampling log identifier (lowercase, must be unique within the campaign) |
| `abstract` | Free-text description of the sampling operation |
| `sampling_tool_array` | Comma-separated list of sampling tools used (e.g. `spade,auger`) |
| `begin_date` | Start date as integer (`YYYYMMDD`) |
| `end_date` | End date as integer (`YYYYMMDD`) |

## The `sampling_tool_array` field

Unlike the `__` FK-lookup arrays, `sampling_tool_array` is a plain comma-separated string stored directly — no foreign key resolution occurs. List the tools used during the sampling operation in the Excel source file.

## Next step

All dataset metadata is now loaded. Proceed to [Load sample data] to register the individual sample geolocations and link them to sampling logs.

**Or, do it in one step instead**: [Insert dataset metadata][insert_dataset_meta] covers all five dataset metadata tables, including this one, via the single-step `insert_tabular_data` route.

[Manage campaign]: /dataset_meta/manage_campaign/
[Load sample data]: /sample/
[insert_dataset_meta]: /dataset_meta/insert_dataset_meta/
