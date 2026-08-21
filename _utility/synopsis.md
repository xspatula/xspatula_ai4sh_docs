---
title: "Load Utility Data"
layout: single
sidebar:
  nav: "utility"
  nav2: "loading_data"
excerpt: "Overview of the utility data types and the required loading sequence before any observation data can be entered into the AI4SH database."
permalink: /utility/
author_profile: false
date: 2026-06-10 08:00:00 +0200
last_modified_at: 2026-06-10 08:00:00 +0200
---

Utility data provides the controlled vocabularies and reference catalogues that all observation data depends on. Every observation record must reference — an indicator, a unit and a provision (e.g. instrument, tool or laboratory analysis method). Utility data must therefore be fully loaded before any dataset, sample, or observation data can be entered.

All utility loading is driven by a single notebook:

```
./ai4sh/import_data/load_ai4sh_utility_data.ipynb
```

## Two groups of utility data

Utility data is split into two groups with different purposes and storage locations.

### General utilities

General utilities contain reference data used across the whole framework, not just observations. For the AI4SH project this group contains a single table:

| Excel file | Database table | Description |
|---|---|---|
| `territory.xlsx` | `utility.territory` | Countries and regions (ISO codes) |

Source: `./ai4sh/import_data/utility/general/excel/`

### Observation utilities

Observation utilities are the catalogues that make soil observations FAIR-compliant — every instrument, method, unit, and indicator used in an observation must first be registered here.

There are three sub-groups within observation utilities:

**Independent** (no foreign key dependencies on other observation utilities):

| Excel file | Database table | Description |
|---|---|---|
| `analysis_method.xlsx` | `analysis_method` | Laboratory and field analysis methods |
| `apparatus.xlsx` | `apparatus` | Instruments and tools delivering data |
| `classification_order.xlsx` | `classification_order` | Highest Linnean style taxonomy level |
| `license.xlsx` | `license` | Dataset and campaign licenses |
| `location_method.xlsx` | `location_method` | Geolocation methods |
| `method_tier.xlsx` | `method_tier` | Professionality level of a method |
| `preparation.xlsx` | `preparation` | Pre-analysis sample preparation |
| `preservation.xlsx` | `preservation` | Sample preservation methods |
| `provider.xlsx` | `provider` | Labs, services and instruments providing results |
| `quantity.xlsx` | `quantity` | Unambiguous quantity definitions |
| `setting_system.xlsx` | `setting_system` | Thematic frame for juxtapositions |
| `spatial_reference.xlsx` | `spatial_reference` | EPSG-based coordinate reference systems |
| `storage.xlsx` | `storage` | Sample storage methods |
| `transportation.xlsx` | `transportation` | Sample transport conditions |
| `unit.xlsx` | `unit` | Units of observation values |

**Dependent** (have foreign key (FK) requirements from the independent tables above and use `__` notation for FK lookup):

| Excel file | Database table | Requires |
|---|---|---|
| `classification_family.xlsx` | `classification_family` | `classification_order` |
| `classification_genus.xlsx` | `classification_genus` | `classification_family` |
| `indicator.xlsx` | `indicator` | `quantity` |
| `juxtaposition.xlsx` | `juxtaposition` | `setting_system` |
| `profiling.xlsx` | `profiling` | `unit` |

**With inheritance** (depend on pre-inserted records in a parent table which value is inherited if the user does not explicitly define a custom value):

| Excel file | Database table | Requires |
|---|---|---|
| `provision.xlsx` | `provision` | `apparatus`, `provider`, `method_tier` |
| `provision_indicator.xlsx` | `provision_indicator` | `provision`, `indicator`, `analysis_method`, `unit` |
| `provision_serial_nr.xlsx` | `provision_serial_nr` | `provision` |

Source: `./ai4sh/import_data/utility/observation/excel/`

## Required loading sequence

The six notebook cells must be run in this order:

1. **[Translate general utilities]** — Excel → JSON for `territory`
2. **[Manage general utilities]** — Insert `territory` into the database
3. **[Translate observation utilities]** — Excel → JSON for independent and dependent tables
4. **[Manage observation utilities]** — Insert those tables into the database
5. **[Translate observation utilities with inheritance]** — Excel → JSON for `provision`, `provision_indicator`, `provision_serial_nr`
6. **[Manage observation utilities with inheritance]** — Insert those tables into the database

[Translate general utilities]: /utility/translate_general_utilities/
[Manage general utilities]: /utility/manage_general_utilities/
[Translate observation utilities]: /utility/translate_observation_utilities/
[Manage observation utilities]: /utility/manage_observation_utilities/
[Translate observation utilities with inheritance]: /utility/translate_observation_utilities_inherit/
[Manage observation utilities with inheritance]: /utility/manage_observation_utilities_inherit/
