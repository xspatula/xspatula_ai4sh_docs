---
title: "Manage Campaign"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Inserting campaign records into the AI4SH database. A campaign is a bounded sampling and observation effort within a dataset, linking the dataset to the provisions used for analysis."
permalink: /dataset_meta/manage_campaign/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

A campaign is a bounded data collection effort — defined by dates, geographic scope, and the instruments used — that belongs to a dataset. Every sampling log must reference a campaign.

The campaign record is also where the link between a dataset and its analytical provisions (instruments and labs) is established, via the `provision_id__provision_name_array` parameter.

## Prerequisites

- [Manage dataset] must be complete.
- Utility records referenced by campaigns must be populated (from [Load utility data]): `territory`, `spatial_reference`, `location_method`, `unit`, `provision`.

## Notebook cell

In `load_ai4sh_dataset_meta.ipynb`, the **Manage campaigns** cell runs:

```python
process_file = 'import_data/dataset/manage_process/manage_campaign.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/dataset/manage_process/manage_campaign.json`

```json
{
  "process": [
    {
      "root_process_id": "import_tabular_data",
      "process": "manage_campaign",
      "delete": false,
      "overwrite": false,
      "parameters": {
        "dataset_id__dataset_name": "ai4soilhealth",
        "name": "ai4sh_se_loennstorp",
        "display_name": "AI4SH SE Loennstorp",
        "contact_name": "thomas gumbricht",
        "contact_email": "thomas.gumbricht@natgeo.su.se",
        "begun_at": 20240910,
        "ended_at": 20240913,
        "url": "https://ai4soilhealth.eu",
        "abstract": "AI4SoilHealth",
        "field": 1.0,
        "home": 1.0,
        "laboratory": 1,
        "satellite": 1.0,
        "territory_id__territory_name": "se",
        "site": "loennstorp",
        "spatial_reference_id__spatial_reference_name": "geographic",
        "location_method_id__location_method_name": "map",
        "location_error": 10,
        "location_error_unit_id__unit_name": "m",
        "provision_id__provision_name_array": "ai4sh-agrolab,foss ds2500 l,ai4sh-slu,ai4sh-metabarcoding,digit soil sear bob,best infiltration,microbiometer-classic,soil cylinder,slakes"
      }
    }
  ]
}
```

## Parameters

| Parameter | Description |
|---|---|
| `dataset_id__dataset_name` | FK lookup: parent dataset by alias |
| `name` | Campaign identifier (lowercase) |
| `display_name` | Display label |
| `contact_name` | Campaign contact name |
| `contact_email` | Campaign contact email |
| `begun_at` | Start date as integer (`YYYYMMDD`) |
| `ended_at` | End date as integer (`YYYYMMDD`) |
| `url` | Campaign or project landing page |
| `abstract` | Free-text description |
| `field`, `home`, `satellite` | Flags: 1 if that observation type is included |
| `laboratory` | Flag: 1 if laboratory analyses included, 0 otherwise |
| `territory_id__territory_name` | FK lookup: geographic territory |
| `site` | Free-text site description |
| `spatial_reference_id__spatial_reference_name` | FK lookup: coordinate reference system |
| `location_method_id__location_method_name` | FK lookup: geolocation method |
| `location_error` | Positional accuracy value |
| `location_error_unit_id__unit_name` | FK lookup: unit for `location_error` (e.g. `m`) |
| `provision_id__provision_name_array` | Comma-separated list of provision aliases — links this campaign to all the instruments and labs used |

## The `provision_id__provision_name_array` field

This parameter links a campaign to one or more provisions defined in the utility catalogues. Multiple provisions are listed as a comma-separated string:

```
"provision_id__provision_name_array": "ai4sh-agrolab,foss ds2500 l,ai4sh-slu,ai4sh-metabarcoding"
```

The framework resolves each name to its `provision_id` and creates a record in the campaign–provision junction table. All provisions listed here must already exist in `observation_utility.provision`.

## Next step

Proceed to [Manage sampling log].

[Manage dataset]: /dataset_meta/manage_dataset/
[Load utility data]: /utility/
[Manage sampling log]: /dataset_meta/manage_sampling_log/
