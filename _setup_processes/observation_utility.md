---
title: "Observation Utility Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Observation utility processes register operations for managing all reference catalogues in the observation_utility schema — apparatus, indicators, methods, units, provisions, software, primer pairs, lab protocols, method pipelines and more. These must be registered before any observation data can be entered."
permalink: /setup_process/observation_utility/
author_profile: false
date: 2026-04-07 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

Observation utility processes register the operations for managing every reference catalogue in the `observation_utility` schema. Each process allows authorised users to insert, update, or delete records in a specific catalogue table. These processes must be registered before any observation data management processes, because observation processes depend on the utility catalogues being populated and manageable.

## Process files

Process files are located at:

```
./setup/zzz/ai4sh/setup_processes/json/observation_utility/
```

| File | Process registered | Target table | Min stratum |
|---|---|---|---|
| `analysis_method_v10_sql.json` | `manage_analysis_method` | `observation_utility.analysis_method` | 4 |
| `analysis_method_translate_v10_sql.json` | `manage_analysis_method_translate` | `observation_utility.analysis_method_translate` | 4 |
| `apparatus_v10_sql.json` | `manage_apparatus` | `observation_utility.apparatus` | 4 |
| `classification_v10_sql.json` | `manage_order`, `manage_family`, `manage_genus`, `manage_species`, `manage_specimen` | `observation_utility.order` / `family` / `genus` / `species` / `specimen` | 4 |
| `indicator_v10_sql.json` | `manage_indicator` | `observation_utility.indicator` | 4 |
| `indicator_parity_v10_sql.json` | `manage_indicator_parity` | `observation_utility.indicator_parity` | 4 |
| `indicator_default_unit_v10_sql.json` | `manage_indicator_default_unit` | `observation_utility.indicator_default_unit` | 4 |
| `juxtaposition_v10_sql.json` | `manage_juxtaposition` | `observation_utility.juxtaposition` | 4 |
| `license_v10_sql.json` | `manage_license` | `observation_utility.license` | 4 |
| `location_method_v10_sql.json` | `manage_location_method` | `observation_utility.location_method` | 4 |
| `method_tier_v10_sql.json` | `manage_method_tier` | `observation_utility.method_tier` | 4 |
| `preparation_v10_sql.json` | `manage_preparation` | `observation_utility.preparation` | 4 |
| `preservation_v10_sql.json` | `manage_preservation` | `observation_utility.preservation` | 4 |
| `profiling_v10_sql.json` | `manage_profiling` | `observation_utility.profiling` | 4 |
| `provider_v10_sql.json` | `manage_provider` | `observation_utility.provider` | 4 |
| `provision_v10_sql.json` | `manage_provision` | `observation_utility.provision`, `provision_method_tier` | 4 |
| `provision_indicator_v10_sql.json` | `manage_provision_indicator` | `observation_utility.provision_indicator` | 4 |
| `provision_serial_nr_v10_sql.json` | `manage_provision_serial_nr` | `observation_utility.provision_serial_nr` | 4 |
| `quantity_v10_sql.json` | `manage_quantity` | `observation_utility.quantity` | 4 |
| `quantity_default_unit_v10_sql.json` | `manage_quantity_default_unit` | `observation_utility.quantity_default_unit` | 4 |
| `setting_system_v10_sql.json` | `manage_setting_system` | `observation_utility.setting_system` | 4 |
| `spatial_reference_v10_sql.json` | `manage_spatial_reference` | `observation_utility.spatial_reference` | 5 |
| `spectrometer_v10_sql.json` | `manage_spectrometer` | `observation_utility.spectrometer` | 4 |
| `spectroscopy_method_v10_sql.json` | `manage_spectroscopy_method` | `observation_utility.spectroscopy_method` | 4 |
| `storage_v10_sql.json` | `manage_storage` | `observation_utility.storage` | 4 |
| `transportation_v10_sql.json` | `manage_transportation` | `observation_utility.transportation` | 4 |
| `unit_v10_sql.json` | `manage_unit` | `observation_utility.unit` | 4 |
| `unit_translate_v10_sql.json` | `manage_unit_translate` | `observation_utility.unit_translate` | 4 |
| `software_v10_sql.json` | `manage_software` | `observation_utility.software` | 3 |
| `edna_primer_pair_v10_sql.json` | `manage_edna_primer_pair` | `observation_utility.edna_primer_pair` | 4 |
| `lab_protocol_v10_sql.json` | `manage_lab_protocol` | `observation_utility.lab_protocol` | 4 |
| `method_pipeline_v10_sql.json` | `manage_method_pipeline` | `observation_utility.method_pipeline` | 3 |
| `method_pipeline_step_v10_sql.json` | `manage_method_pipeline_step` | `observation_utility.method_pipeline_step` | 3 |

All files above are listed in the pilot file `setup_processes.txt`. The last five register the eDNA method catalogues and pipelines (see [eDNA metabarcoding][setup_db_edna]); in the pilot they follow `manage_setting_system`, and `manage_method_pipeline_step` comes after `manage_method_pipeline`, `manage_software`, `manage_edna_primer_pair` and `manage_lab_protocol` it references. `manage_taxonomy_reference`, also referenced by pipeline steps, is registered in [organism_utility][setup_process_organism_utility].

## Access level

Most observation utility management processes require a minimum user stratum of 4. This reflects that catalogue management (adding new instrument types, methods, units, etc.) is an administrative-level operation, not something an ordinary data contributor should be able to do. Incorrect or inconsistent catalogue entries would break the referential integrity of all observations that reference them.

Two exceptions:

- `manage_spatial_reference` requires stratum **5** — a wrong spatial reference silently misplaces every geolocation that uses it.
- `manage_software`, `manage_method_pipeline` and `manage_method_pipeline_step` require only stratum **3**.

## Key processes in detail

### manage_apparatus

Registers a new type of apparatus — any instrument, tool, laboratory process, or service that can deliver observation data. Parameters:

- `name` — generic apparatus name (e.g. "VNIR spectroscopy", "wet chemistry laboratory", "metabarcoding")
- `alias` — common shorthand (e.g. "wetlab")
- `display_name` — display label
- `abstract` — plain language description

The `display_name` and `abstract` can be updated; `name` and `alias` cannot (they are permanent identifiers).

### manage_provision

A provision combines an apparatus, a provider, and a method tier into a single identifiable combination. This is the key linking record that connects "what instrument" + "who operates it" + "at what professionality level" into a single entity that observations can reference. Parameters:

- `provider_id__provider_name` — look up provider by name
- `apparatus_id__apparatus_name` — look up apparatus by name
- `method_tier_id__method_tier_name` — look up method tier by name
- `name`, `alias`, `display_name` — identification; `alias` is what observation logs and campaigns reference (e.g. `ai4sh-metabarcoding`)
- `url`, `abstract`, `contact_name`, `contact_email` — contact fields default to `inherit` (taken from the provider)
- `field`, `home`, `laboratory`, `drone`, `satellite`, `document`, `senses`, `auxiliary` — boolean method-tier capability flags written to `provision_method_tier`

### manage_provision_indicator

Links a provision to the specific indicators (measurable values) it delivers, together with the analysis method and unit. This is the record an observation references when storing an actual measurement value. Parameters:

- `provision_id__provision_name` — the provision delivering this indicator
- `indicator_id__indicator_name` — the indicator being measured
- `analysis_method_id__analysis_method_name` — the analysis method used
- `unit_id__unit_name` — the unit of the reported value

All four are required and none can be updated or deleted — to change a link, delete the record and insert a new one. For eDNA, provision `ai4sh-metabarcoding` links to 19 indicators with analysis methods `ai4sh 16s metabarcoding` or `ai4sh its metabarcoding` and unit `unitless`.

### manage_indicator

Registers a measurable soil property. An indicator belongs to a `quantity` (the physical property type) and has a name, alias, and abstract. The combination of indicator + provision_indicator defines exactly what value an observation stores.

### manage_software

Registers a software tool **at a specific version**.

| Parameter | Type | Required | Update | Description |
|---|---|---|---|---|
| `name` | text | yes | — | Software name, e.g. `qiime2` |
| `version` | text | no (default `unspecified`) | — | Version, e.g. `2024.10` |
| `alias` | text | yes | no | Unique `'<name> <version>'`, e.g. `qiime2 2024.10` — what pipeline steps reference |
| `url` | text | yes | — | Project page |
| `doi` | text | no | — | Citation DOI |
| `abstract` | text | no | — | What the software does |

### manage_edna_primer_pair

Registers a forward + reverse primer pair used together in one PCR. Required: `name`, `marker_gene` (e.g. `16S rRNA`), `target_region` (e.g. `V4`), `target_group` (e.g. `prokaryotes`), `forward_name`, `forward_sequence`, `reverse_name`, `reverse_sequence`. Optional: `url`, `doi`, `abstract`. No parameter can be updated or deleted. Primer sequences are stored with their original upper case (IUPAC codes such as `Y`, `M`, `N` are significant).

### manage_lab_protocol

Registers a laboratory protocol. Required: `name`, `protocol_type` — one of `extraction`, `amplification`, `purification`, `library_preparation`, `sequencing`, `other`. Optional: `kit`, `manufacturer`, `platform`, `strategy` (e.g. `2x250 bp paired-end`), `data_requirement` (e.g. `30k tags`), `url`, `doi`, `abstract`. Only `url` and `doi` can be updated.

### manage_method_pipeline

Registers a named multi-step analysis method.

| Parameter | Type | Required | Update | Description |
|---|---|---|---|---|
| `name` | text | yes | no | Unique pipeline name, e.g. `ai4sh-16s` |
| `version` | text | no (default `unspecified`) | yes | Pipeline version |
| `analysis_method_id__analysis_method_name` | text | yes | no | The `analysis_method` the pipeline implements, e.g. `ai4sh 16s metabarcoding` |
| `url`, `doi`, `abstract` | text | no | yes | Provenance |

### manage_method_pipeline_step

Registers one numbered step of a pipeline. Blank reference cells mean "not applicable".

| Parameter | Type | Required | Update | Description |
|---|---|---|---|---|
| `method_pipeline_id__method_pipeline_name` | text | yes | no | Pipeline the step belongs to |
| `step` | integer | yes | no | Step number, 1 = first |
| `stage` | text | yes | yes | `laboratory` or `bioinformatics` |
| `name` | text | yes | yes | e.g. `paired-end merging` |
| `lab_protocol_id__lab_protocol_name` | text | no | yes | Lab protocol of a laboratory step |
| `edna_primer_pair_id__edna_primer_pair_name` | text | no | yes | Primer pair of the amplification step |
| `software_id__software_alias` | text | no | yes | Software as `'<name> <version>'` |
| `taxonomy_reference_id__taxonomy_reference_alias` | text | no | yes | Taxonomy reference as `'<name> <version>'` |
| `parameters` | text (JSON) | no (default `{}`) | yes | Step parameters as a JSON object, stored as JSONB |
| `abstract` | text | no | yes | Description of the step |

Software and taxonomy references are looked up by **alias including the version**. If a version is changed in the software catalogue but not in the step (or vice versa), the lookup fails and the step is reported as an error.

[setup_db_edna]: /setup_db/edna_metabarcoding/
[setup_process_organism_utility]: /setup_process/organism_utility/
