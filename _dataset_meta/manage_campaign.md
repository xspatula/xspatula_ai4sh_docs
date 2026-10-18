---
title: "Dataset Metadata: Manage Campaign"
layout: single
sidebar:
  nav: "dataset_meta"
  nav2: "loading_data"
excerpt: "Inserting campaign records into the AI4SH database. A campaign is the place (territory and site) where a dataset is sampled, and links the dataset to the provisions used for analysis."
permalink: /dataset_meta/manage_campaign/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-10-17 08:00:00 +0200
---

A campaign is a **place** within a dataset: a territory and, optionally, a site. In AI4SH each pilot site is one campaign. The **time** part lives one level down: each sampling of the campaign at a given date is a [sampling log][Manage sampling log], identified by its `datetime_tag`. A pilot site sampled in 2024 and again in 2025 is therefore one campaign with two sampling logs. Every sampling log must reference a campaign.

The campaign record is also where the link between a dataset and its analytical provisions (instruments and labs) is established, via the `provision_id__provision_name_array` parameter.

## Prerequisites

- [Manage dataset] must be complete.
- Utility records referenced by campaigns must be populated (from [Load utility data]): `territory`, `spatial_reference`, `location_method`, `unit`, `provision`.

## Notebook cell

In the 2-step notebook `./ai4sh/import_data/more_notebooks/load_ai4sh_dataset_meta.ipynb`, the **Manage campaigns** cell runs:

```python
process_file = 'import_data/dataset_metadata/manage_process/manage_campaign.json'

structured_process_D, scheme_params_D = Initiate_process(notebook_path, scheme_file, process_file)

if structured_process_D is not None:
    Run_process(structured_process_D, scheme_params_D)
```

## Manage process file

**Path**: `./ai4sh/import_data/dataset_metadata/manage_process/manage_campaign.json`

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
        "name": "ai4sh-se-loennstorp",
        "display_name": "AI4SH SE Loennstorp",
        "contact_name": "thomas gumbricht",
        "contact_email": "thomas.gumbricht@natgeo.su.se",
        "begun_at": 20240910,
        "ended_at": 20240913,
        "url": "https://ai4soilhealth.eu",
        "abstract": "AI4SoilHealth",
        "field": 1,
        "home": 1,
        "laboratory": 1,
        "satellite": 1,
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

This is the first of the twelve AI4SH campaigns (one per pilot site) that `campaign.xlsx` generates. `tag` is left out and is therefore composed automatically (see below).

## Parameters

| Parameter | Description |
|---|---|
| `dataset_id__dataset_name` | **Required.** FK lookup: parent dataset by name, tag or alias |
| `name` | Optional. Left out or `auto`: composed as `<dataset alias> <territory name> <site>`. Unique within the dataset |
| `tag` | Optional. Left out or `auto`: composed as `<dataset tag>_<territory code>_<site>`. Globally unique |
| `display_name` | Optional. Left out or `auto`: composed as `<dataset alias> <territory name>` |
| `territory_id__territory_name` | FK lookup: the territory of the campaign (name or ISO code, e.g. `se`). Part of the auto name and tag |
| `site` | Name of the site within the territory (e.g. `loennstorp`). Part of the auto name and tag |
| `contact_name` | Campaign contact name; left out it is inherited from the dataset |
| `contact_email` | Campaign contact email |
| `begun_at` | Start date (`YYYYMMDD`) |
| `ended_at` | End date (`YYYYMMDD`) |
| `url` | Campaign or project landing page; inherited from the dataset if left out |
| `abstract` | Free-text description; inherited from the dataset if left out |
| `field`, `home`, `laboratory`, `drone`, `satellite`, `document`, `senses`, `auxiliary` | Flags: 1 if that type of observation is included, default 0 |
| `time_series` | 1 if the campaign is part of a time series (the dataset must then also be flagged as time series) |
| `spatial_reference_id__spatial_reference_name` | FK lookup: coordinate reference system; inherited from the dataset if left out |
| `location_method_id__location_method_name` | FK lookup: geolocation method (e.g. `gps`, `map`) |
| `location_error` | Positional accuracy value |
| `location_error_unit_id__unit_name` | FK lookup: unit for `location_error` (default `m`) |
| `provision_id__provision_name_array` | **Required.** Comma-separated list of provision aliases — links this campaign to all the instruments and labs used |

The parameters `species_id__species_name`, `profiling_id__profiling_name`, `setting_system_id__setting_system_name_array`, `license_id__license_name`, `substance_array`, `keyword_array` and `doi` are also accepted; left out they are inherited from the dataset.

A campaign has **no** `datetime_tag`. It moved to the [sampling log][Manage sampling log] when campaigns became places and sampling logs became samplings in time.
{: .notice--info}

## Automatic naming

| Parameter | Auto concat | Empty territory | Empty site |
|---|---|---|---|
| `name` | `'%s %s %s' %(dataset_id__dataset_alias, territory_id__territory_name, site?)` | `no location` | dropped with its separator |
| `tag` | `'%s_%s_%s' %(dataset_id__dataset_tag, territory_id__territory_iso_code_a2_ext, site?)` | `0` | dropped with its separator |

The territory part of the tag is the extended ISO code (`iso_code_a2_ext`), e.g. `se` or `gb-eng`. A value given by hand that becomes part of a tag (here `site`) may only hold lower case letters, digits, `.` and `-`.

A hand-written `name` or `tag` containing `_` or `@` is rejected unless it equals the auto name. The AI4SH campaigns therefore give a hyphenated `name` (e.g. `ai4sh-se-loennstorp`), which is accepted, and leave `tag` to be composed. For the example above that gives the tag `ai4sh_se_loennstorp` (dataset tag `ai4sh`, territory code `se`, site `loennstorp`).

The campaign `name` and `tag` are inherited downstream: the [sampling log][Manage sampling log] adds its `datetime_tag` to them. See [Automatic naming and tags][auto_naming].

## The `provision_id__provision_name_array` field

This parameter links a campaign to one or more provisions defined in the utility catalogues. Multiple provisions are listed as a comma-separated string:

```
"provision_id__provision_name_array": "ai4sh-agrolab,foss ds2500 l,ai4sh-slu,ai4sh-metabarcoding"
```

The framework resolves each name to its `provision_id` and creates a record in the campaign–provision junction table. All provisions listed here must already exist in `observation_utility.provision`.

## Next step

Proceed to [Manage sampling log].

[Manage dataset]: {{ site.baseurl }}/dataset_meta/manage_dataset/
[Load utility data]: {{ site.baseurl }}/utility/
[Manage sampling log]: {{ site.baseurl }}/dataset_meta/manage_sampling_log/
[auto_naming]: {{ site.baseurl }}/setup_process/auto_naming/
