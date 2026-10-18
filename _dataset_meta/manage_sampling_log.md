---
title: "Dataset Metadata: Manage Sampling Log"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Inserting sampling log records into the AI4SH database. A sampling log is one sampling of a campaign (place) at a given time, identified by its datetime_tag."
permalink: /dataset_meta/manage_sampling_log/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-10-17 08:00:00 +0200
---

A sampling log is one sampling of a campaign at a given time. The [campaign][Manage campaign] is the place (territory and site); the sampling log adds the time, as a `datetime_tag`, together with the tools used, the date range and the responsible contacts. A pilot site sampled in 2024 and again in 2025 has two sampling logs under the same campaign. Individual samples (with geolocations) are later linked to a sampling log.

## Prerequisites

- [Manage campaign] must be complete.

## Notebook cell

In the 2-step notebook `./ai4sh/import_data/more_notebooks/load_ai4sh_dataset_meta.ipynb`, the **Manage sampling logs** cell runs:

```python
process_file = 'import_data/dataset_metadata/manage_process/manage_sampling_log.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/dataset_metadata/manage_process/manage_sampling_log.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_sampling_log",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "campaign_id__campaign_name": "ai4sh-se-loennstorp",
        "contact_name": "thomasg",
        "contact_email": "thomas.gumbricht@natgeo.us.se",
        "datetime_tag": 2024,
        "abstract": "AI4SH in-situ sampling",
        "sampling_tool_array": "spade,auger",
        "begun_at": 20240910,
        "ended_at": 20240913
      }
    }
  ]
}
```

One JSON object is generated per row in `sampling_log.xlsx`, i.e. one per sampling of a campaign. AI4SH has twelve campaigns and fifteen sampling logs, as some pilot sites were sampled in more than one year. `name`, `tag` and `display_name` are left out and composed automatically (see below).

## Parameters

| Parameter | Description |
|---|---|
| `campaign_id__campaign_name` | **Required.** FK lookup: parent campaign by name, tag or alias |
| `datetime_tag` | Time stamp of the sampling: `YYYY`, `YYYYMM`, `YYYYMMDD` or `YYYYMMDDhhmmss`; `0` (default) if no date. Part of the auto name and tag |
| `name` | Optional. Left out or `auto`: composed as `<campaign name> <datetime_tag>`. Unique within the campaign |
| `tag` | Optional. Left out or `auto`: composed as `<campaign tag>_<datetime_tag>`. Globally unique |
| `display_name` | Optional. Left out or `auto`: composed as `<campaign display name> <datetime_tag>` |
| `contact_name` | Name of the person responsible for this sampling; inherited from the campaign if left out |
| `contact_email` | Email of the responsible person; inherited from the campaign if left out |
| `url` | URL describing this sampling (e.g. the data release of the sampling year) |
| `abstract` | Free-text description of the sampling |
| `sampling_tool_array` | Comma-separated list of sampling tools used (e.g. `spade,auger`) |
| `setting_system_id__setting_system_name_array` | FK lookup: juxtaposition settings (e.g. inside/outside); inherited from the campaign if left out |
| `time_series` | 1 if the sampling log is part of a time series (dataset and campaign must then also be flagged) |
| `begun_at` | Start date (`YYYYMMDD`); left out it is set to the date of registration |
| `ended_at` | End date (`YYYYMMDD`); left out it is set to the date of the last entry |

## Automatic naming

| Parameter | Auto concat | No date (`datetime_tag` = `0` or empty) |
|---|---|---|
| `name` | `'%s %s' %(campaign_id__campaign_name, datetime_tag)` | `no date` |
| `tag` | `'%s_%s' %(campaign_id__campaign_tag, datetime_tag)` | `0` |
| `display_name` | `'%s %s' %(campaign_id__campaign_display_name, datetime_tag)` | `no date` |

For the example above (campaign `ai4sh-se-loennstorp`, campaign tag `ai4sh_se_loennstorp`, `datetime_tag` 2024) this gives:

| Parameter | Composed value |
|---|---|
| `name` | `ai4sh-se-loennstorp 2024` |
| `tag` | `ai4sh_se_loennstorp_2024` |
| `display_name` | `AI4SH SE Loennstorp 2024` |

A `datetime_tag` that is not `0` or one of the four date formats is rejected. A hand-written `name` or `tag` containing `_` or `@` is rejected unless it equals the auto name. If a campaign name exists in more than one dataset, give the campaign tag in `campaign_id__campaign_name` instead. The sampling log tag is in turn the first part of the observation log tag (`<sampling log tag>@<provision tag>`). See [Automatic naming and tags][auto_naming].

## The `sampling_tool_array` field

Unlike the `__` FK-lookup arrays, `sampling_tool_array` is a plain comma-separated string stored directly — no foreign key resolution occurs. List the tools used during the sampling operation in the Excel source file.

## Next step

All dataset metadata is now loaded. Proceed to [Load sample data] to register the individual sample geolocations and link them to sampling logs.

**Or, do it in one step instead**: [Insert dataset metadata][insert_dataset_meta] covers all five dataset metadata tables, including this one, via the single-step `insert_tabular_data` route.

[Manage campaign]: {{ site.baseurl }}/dataset_meta/manage_campaign/
[Load sample data]: {{ site.baseurl }}/sample/
[insert_dataset_meta]: {{ site.baseurl }}/dataset_meta/insert_dataset_meta/
[auto_naming]: {{ site.baseurl }}/setup_process/auto_naming/
