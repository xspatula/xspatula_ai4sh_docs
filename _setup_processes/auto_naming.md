---
title: "Automatic Naming and Tags"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Datasets, campaigns, sampling logs, provisions and observation logs carry a free-text name for people and a short tag for machines. Campaign, sampling log and observation log names and tags are composed automatically from their parents."
permalink: /setup_process/auto_naming/
author_profile: false
date: 2026-10-11 08:00:00 +0200
last_modified_at: 2026-10-17 08:00:00 +0200
---

This page summarises automatic naming (`auto_name`) as it applies to AI4SH. The rules are shared with all Xspatula projects and are described in full in the core documentation, [Automatic naming and tags][core_auto_naming]. Each section below links to the matching part of that page.

**Status:** the code, process definitions and AI4SH Excel sources are updated (campaign = place, sampling log = time), but the AI4SH database has not yet been rebuilt with them. The AI4SH examples below are composed from the concat rules and the Excel files; values marked **PLACEHOLDER** are to be confirmed after the rebuild.
{: .notice--warning}

## Name and tag

Records that are used to build other names have two identifiers:

- `name` is free text for people. It may contain blanks and should read naturally.
- `tag` is a short key for machines. It is used to build composed names and, later, the file names produced by the machine learning steps.

Think of the name as the label on a jar and the tag as its barcode. ([Full description][core_name_and_tag])

Do not confuse these tags with the tables `observation.dataset_tag` and `observation.campaign_tag`, which hold free-text keywords and substances (see [Schema conventions][setup_process_schema_conventions]).

## Inputs per level

The tag columns are new Excel columns (and process parameters). ([Full description][core_tag_columns])

{% capture notice-2 %}
| Level | Excel column | Required | Given by | Unique |
|---|---|---|---|---|
| dataset | `tag` | yes | you | globally |
| provision | `tag` | yes | you | globally |
| campaign | `territory_id__territory_name` | no (`no location` / `0` if empty) | you | — |
| campaign | `site` | no (dropped if empty) | you | — |
| campaign | `name`, `tag` | no | auto (or you) | `tag` globally |
| sampling_log | `datetime_tag` | no (`0` = no date, the default) | you | — |
| sampling_log | `name`, `tag`, `display_name` | no | auto (or you) | `tag` globally |
| observation_log | `name`, `tag` | no | auto (or you) | `tag` per sampling log |
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

A `name` or `tag` is composed automatically when it is set to `auto`, or when the column is left out (the default value is `auto`).

The campaign is the **place** and the sampling log the **time**: `datetime_tag` is the time part of the sampling log tag. It must be `0` (no date) or `YYYY`, `YYYYMM`, `YYYYMMDD` or `YYYYMMDDhhmmss`; anything else is rejected. A value given by hand that becomes part of a composed tag (e.g. `site`, `datetime_tag`) may only hold lower case letters, digits, `.` and `-`.

## The naming chain

Each level builds its name and tag from the level above:

{% capture notice-2 %}
| Level | `name` concat | `tag` concat |
|---|---|---|
| campaign | `'%s %s %s' %(dataset_id__dataset_alias, territory_id__territory_name, site?)` | `'%s_%s_%s' %(dataset_id__dataset_tag, territory_id__territory_iso_code_a2_ext, site?)` |
| sampling_log | `'%s %s' %(campaign_id__campaign_name, datetime_tag)` | `'%s_%s' %(campaign_id__campaign_tag, datetime_tag)` |
| observation_log | `'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)` | `'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)` |
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

`site?` is optional: if it is empty, it is dropped together with the separator before it. A missing territory gives `no location` in a name and `0` in a tag; a missing `datetime_tag` gives `no date` / `0`. The territory is a normal foreign key argument: `territory_id__territory_iso_code_a2_ext` returns the `iso_code_a2_ext` of the territory given in `territory_id__territory_name`. The earlier special argument `campaign_territory_id`, which read the territory from `observation.campaign_location`, is gone. ([Full description][core_concat_arguments])

The pattern, as run in the LUCAS project:

```
dataset tag      lucas
campaign         lucas european union       / lucas_eu
sampling_log     lucas european union 2009  / lucas_eu_2009
observation_log  tag lucas_eu_2009@foss-xds-rca
```

LUCAS 2009 and LUCAS 2015 are two sampling logs of the one campaign `lucas european union`.

In AI4SH each pilot site is a campaign (dataset + territory + site) and each sampling year is a sampling log. The AI4SH campaigns give a hyphenated `name` by hand (e.g. `ai4sh-se-loennstorp`) and leave the tag to be composed. For Lönnstorp, sampled in 2024 and measured on the FOSS DS2500:

```
dataset tag      ai4sh
campaign         ai4sh-se-loennstorp                     / ai4sh_se_loennstorp
sampling_log     ai4sh-se-loennstorp 2024                / ai4sh_se_loennstorp_2024
observation_log  tag ai4sh_se_loennstorp_2024@foss-ds2500-l
```

A site sampled in two years, e.g. Neretva in 2024 and 2025, has one campaign `ai4sh_hr_neretva` and the two sampling logs `ai4sh_hr_neretva_2024` and `ai4sh_hr_neretva_2025`. ([Worked example][core_worked_example])

**PLACEHOLDER:** confirm the AI4SH names and tags against the database after the rebuild.
{: .notice--warning}

## Example: the sampling_log auto_name block

The `auto_name` block sits on the parameter in the process definition (`sampling_log_v10_sql.json`):

```json
{
  "parameter": "tag",
  "parameter_type": "text",
  "required": false,
  "default_value": "auto",
  "auto_name": {
    "concat": "'%s_%s' %(campaign_id__campaign_tag, datetime_tag)"
  }
}
```

The format string must be in single quotes. Arguments named `<table>_id__<table>_<column>` return that column of the record given by `<table>_id__<table>_name`; any other argument (here `datetime_tag`) is taken from the process parameters as given. An argument ending in `?` is optional. ([Full description][core_concat_arguments])

## Slakes, aggregate app and Beerkan observations

The app-based observation processes do not compose a new name; they use auto naming to **find** the observation log for each row from the sampling log tag and a fixed provision tag:

{% capture notice-2 %}
| Process | File | Concat on `observation_log_id__observation_log_name` |
|---|---|---|
| `manage_app_aggregate_stability_observation` | `slakes_v10_sql.json` | `'%s@slakes' %(sampling_log_id__sampling_log_tag)` |
| `manage_aggregate_app_observation` | `aggregate_app_observation_v10_sql.json` | `'%s@slakes' %(sampling_log_id__sampling_log_tag)` |
| `manage_beerkan_observation` | `beerkan_infiltration_v10_sql.json` | `'%s@beerkan' %(sampling_log_id__sampling_log_tag)` |
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

The composed value is looked up as name, tag or alias of an existing observation log. It therefore only resolves if the observation log was auto named with a provision whose tag is exactly `slakes` or `beerkan`. In `provision.xlsx` the slakes app has the tag `slakes`, but the Beerkan method (`beerkan estimation of soil hydraulic properties`) has the tag `best-infiltration`. **PLACEHOLDER:** either the provision tag or the Beerkan concat must change before `manage_beerkan_observation` can find its observation log.

## Reserved separators and the geolocation exception

`_` and `@` join the parts of composed names, so that a composed name can be split back into its parts. A hand-written `name`, `alias` or `tag` that contains `_` or `@` is **rejected**, unless it exactly equals the auto name. Either leave the value as `auto`, or give your own value without separators; use a hyphen for word breaks (e.g. `ai4sh-agrolab`). ([Full description][core_reserved_separators])

The one exception is the geolocation name in `manage_geolocation`, e.g. `<country>_<source>@<point id>`. This matters most for AI4SH:

- In AI4SH the same physical sample often arrives under different names from different sources — the field team, the wet lab, the spectral lab, the eDNA lab.
- The geolocation is registered **first**, with one unique entry per sample, and acts as the key that ties those names together.
- Without that single key, the same sample could be registered twice under two names, and its observations would be split between them.

The geolocation name is set before any composed names exist and is never split again, so it may contain separators. For the same reason, never use `geolocation_id__geolocation_name` as an argument in a concat. ([Exemptions][core_exemptions])

Further exceptions, if AI4SH needs them, go in `separator_exempt_D` in `src/lib/structure.py`:

```python
separator_exempt_D = {'manage_geolocation': ['name']}
```

## Rules for users

- **A parent can be given by its name, tag or alias.** `dataset_id__dataset_name` accepts any of the three; the concat still returns the column it asks for. ([Record lookup][core_record_lookup])
- **Ambiguous names.** If a name matches more than one record (e.g. the same campaign name in two datasets), give the tag instead, or add the parent parameter to the call. A tag always resolves to one record. ([Record lookup][core_record_lookup])
- **Load order.** Parents must exist, with tags, before children are auto named: dataset → campaign → sampling_log → observation_log → slakes / aggregate app / Beerkan observations. Provisions must be registered before the observation logs that use them. ([Requirements][core_requirements])
- **Errors stop the process.** Every failed lookup logs an `❌ ERROR auto_name: ...` message and nothing is written. The core page lists each message with its fix. ([Errors][core_errors])

## Full description

- [Name and tag][core_name_and_tag]
- [Tag columns][core_tag_columns]
- [Reserved separators][core_reserved_separators]
- [Concat arguments][core_concat_arguments]
- [Record lookup][core_record_lookup]
- [Worked example][core_worked_example]
- [Errors][core_errors]
- [Requirements][core_requirements]

[core_auto_naming]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/
[core_name_and_tag]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#name-and-tag
[core_tag_columns]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#tag-columns
[core_reserved_separators]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#reserved-separators
[core_exemptions]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#exemptions
[core_concat_arguments]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#concat-arguments
[core_record_lookup]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#record-lookup
[core_worked_example]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#worked-example
[core_errors]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#errors
[core_requirements]: https://xspatula.github.io/xspatula_core_docs/setup_processes/auto_naming/#requirements
[setup_process_schema_conventions]: {{ site.baseurl }}/setup_process/schema_conventions/
