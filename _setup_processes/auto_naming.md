---
title: "Automatic Naming and Tags"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Datasets, campaigns, sampling logs, provisions and observation logs carry a free-text name for people and a short tag for machines. Campaign, sampling log and observation log names and tags are composed automatically from their parents."
permalink: /setup_process/auto_naming/
author_profile: false
date: 2026-10-11 08:00:00 +0200
last_modified_at: 2026-10-11 08:00:00 +0200
---

This page summarises automatic naming (`auto_name`) as it applies to AI4SH. The rules are shared with all Xspatula projects and are described in full in the core documentation, [Automatic naming and tags][core_auto_naming]. Each section below links to the matching part of that page.

**Status:** the code and process definitions are ported to `xspatula_ai4sh` (branch `feature/tag-auto-naming`), but the AI4SH Excel sources are not yet updated and the chain has not been tested live. Values marked **PLACEHOLDER** are to be filled in after the live test.
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
| campaign | `datetime_tag` | yes | you | — |
| campaign | `name`, `tag` | no | auto (or you) | `tag` globally |
| sampling_log | `name`, `tag` | no | auto (or you) | `tag` globally |
| observation_log | `name`, `tag` | no | auto (or you) | `tag` per sampling log |
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

A `name` or `tag` is composed automatically when it is set to `auto`, or when the column is left out (the default value is `auto`).

The `datetime_tag` is the time part of the campaign tag (e.g. a year). The process definitions carry a `format` block for the tags (no underscore, no `@`, no capitals, no blanks), but **no code enforces it yet**. Keep to it by hand.

## The naming chain

Each level builds its name and tag from the level above:

{% capture notice-2 %}
| Level | `name` concat | `tag` concat |
|---|---|---|
| campaign | `'%s %s' %(dataset_id__dataset_alias, datetime_tag)` | `'%s_%s' %(dataset_id__dataset_tag, datetime_tag)` |
| sampling_log | `'%s %s' %(campaign_id__campaign_name, campaign_territory_id)` | `'%s_%s' %(campaign_id__campaign_tag, campaign_territory_id)` |
| observation_log | `'%s@%s' %(sampling_log_id__sampling_log_name, provision_id__provision_name)` | `'%s@%s' %(sampling_log_id__sampling_log_tag, provision_id__provision_tag)` |
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

`campaign_territory_id` is the territory code of the campaign, read from `observation.campaign_location` (written when the campaign is registered with `territory_id__territory_name`). ([Full description][core_concat_arguments])

The pattern, as tested in the LUCAS project:

```
dataset tag      lucas
campaign         lucas 2009     / lucas_2009
sampling_log     lucas 2009 eu  / lucas_2009_eu
observation_log  tag lucas_2009_eu@xds
```

The AI4SH equivalent, once the Excel sources carry tags:

```
dataset tag      ai4sh                                (PLACEHOLDER: dataset tag)
campaign         ai4sh <datetime_tag> / ai4sh_<datetime_tag>   (PLACEHOLDER: datetime_tag)
sampling_log     ai4sh <datetime_tag> se / ai4sh_<datetime_tag>_se
observation_log  tag ai4sh_<datetime_tag>_se@<provision tag>   (PLACEHOLDER: provision tag)
```

The AI4SH dataset alias is `ai4sh` and the Lönnstorp campaign territory is `se`; the tags themselves are not yet set. ([Worked example][core_worked_example])

## Example: the sampling_log auto_name block

The `auto_name` block sits on the parameter in the process definition (`sampling_log_v10_sql.json`):

```json
{
  "parameter": "tag",
  "parameter_type": "text",
  "required": false,
  "default_value": "auto",
  "auto_name": {
    "concat": "'%s_%s' %(campaign_id__campaign_tag, campaign_territory_id)"
  }
}
```

The format string must be in single quotes. Arguments named `<table>_id__<table>_<column>` return that column of the record given by `<table>_id__<table>_name`; any other argument is taken from the process parameters as given. ([Full description][core_concat_arguments])

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

The composed value is looked up as name, tag or alias of an existing observation log. It therefore only resolves if the observation log was auto named with a provision whose tag is exactly `slakes` or `beerkan`. **PLACEHOLDER:** the provision tags for the slakes app and the Beerkan infiltration method are not yet set; they must be `slakes` and `beerkan` for these lookups to work.

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
