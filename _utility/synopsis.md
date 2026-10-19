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
last_modified_at: 2026-10-19 08:00:00 +0200
---

Utility data provides the controlled vocabularies and reference catalogues that all observation data depends on. Every observation record must reference — an indicator, a unit and a provision (e.g. instrument, tool or laboratory analysis method). Utility data must therefore be fully loaded before any dataset, sample, or observation data can be entered.

Utility data is loaded by block **1. Utilities** of the single loading notebook (see [Load all data][all_data]):

```
./ai4sh/import_data/insert_ai4sh_data.ipynb   block 1, cells 1.1–1.4
```

The block uses the single-step `insert_tabular_data` route, which is INSERT-only by construction
and therefore always safe to re-run. If you need to `UPDATE` an existing record or want to
hand-inspect the generated JSON before it's applied, use the 2-step notebook
`./ai4sh/import_data/more_notebooks/load_ai4sh_utility_data.ipynb` instead. See
[Insert utility data][insert_utility_data] for the block 1 walkthrough and
[Single-step vs dual-step][insert_vs_translate] for the full comparison.

## Two groups of utility data

Utility data is split into two groups with different purposes and storage locations.

### General utilities

General utilities contain reference data used across the whole framework, not just observations. For the AI4SH project this group contains two tables:

| Excel file | Database table | Description |
|---|---|---|
| `territory.xlsx` | `utility.territory` | Countries and regions (ISO codes) |
| `foreign_key.xlsx` | `utility.foreign_key` | Non-standard foreign key lookups (single-step route only) |

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
| `indicator_parity.xlsx` | `indicator_parity` | `indicator` |
| `juxtaposition.xlsx` | `juxtaposition` | `setting_system` |
| `profiling.xlsx` | `profiling` | `unit` |
| `nominal_classes.xlsx` | `nominal_classification` | `indicator`, `analysis_method`, `method_tier` |

**With inheritance** (depend on pre-inserted records in a parent table which value is inherited if the user does not explicitly define a custom value):

| Excel file | Database table | Requires |
|---|---|---|
| `provision.xlsx` | `provision` | `apparatus`, `provider`, `method_tier` |
| `provision_indicator.xlsx` | `provision_indicator` | `provision`, `indicator`, `analysis_method`, `unit` |
| `provision_serial_nr.xlsx` | `provision_serial_nr` | `provision` |

**Derived texture classes**: `nominal_classes.xlsx` holds the 32 soil texture classes (12 USDA, 12 IUSS, 4 major groups for each), all with analysis method `derived`. Their four indicators (`soil texture usda`, `soil texture usda major group`, `soil texture iuss`, `soil texture iuss major group`) have quantity `class` (`quantity.xlsx`), and unit `class` is in `unit.xlsx`. `provision_indicator.xlsx` registers all four, with analysis method `derived` and unit `class`, for `ai4sh-agrolab`, `ai4sh-veltia` and `ai4sh-ibec`; removing a row switches that class off for the provision. The classes are computed while the observations load — see [Derived soil texture classes][obs_texture].

**eDNA method catalogues** (inserted last in the observation utility chain, section `EDNA UTILITIES` of `insert_observation_utility.txt`):

| Excel file | Database table | Requires |
|---|---|---|
| `software.xlsx` | `observation_utility.software` | — |
| `taxonomy_reference.xlsx` | `organism_utility.taxonomy_reference` | — |
| `edna_primer_pair.xlsx` | `observation_utility.edna_primer_pair` | — |
| `lab_protocol.xlsx` | `observation_utility.lab_protocol` | — |
| `method_pipeline.xlsx` | `observation_utility.method_pipeline` | `analysis_method` |
| `method_pipeline_step.xlsx` | `observation_utility.method_pipeline_step` | all of the above |

How to fill these in is described in [eDNA method catalogues][edna_catalogues].

Source: `./ai4sh/import_data/utility/observation/excel/`

### Landscape utilities

Landscape utilities are catalogues for land use, land cover, land form, biogeography etc. AI4SoilHealth delivers no landscape data; the catalogues are loaded for completeness and for merging with [LUCAS data]({{ site.baseurl }}/#including-lucas-data). As shipped, cell 1.4 inserts `biogeo16`, `landuse` and `landcover`; the other landscape catalogues are listed in the pilot file but switched off.

Land cover and land use are ranked classifications (order → family → genus), stored like taxa in one table per scheme. `landcover.xlsx` and `landuse.xlsx` hold one row per class, with the columns `class_rank_id__class_rank_name`, `parent_<scheme>_id__<scheme>_alias`, `name`, `alias`, `tag`, `display_name` and `abstract`. A parent row must come before its children; an order has no parent. Example rows from `landcover.xlsx`:

| `class_rank_id__class_rank_name` | `parent_landcover_id__landcover_alias` | `name` | `alias` | `tag` |
|---|---|---|---|---|
| `order` | | `cropland` | `b` | `cropland` |
| `family` | `b` | `cultivated cropland` | `b10` | `cultivated` |
| `genus` | `b10` | `common wheat` | `b11` | `wheat` |

See [Landscape schema]({{ site.baseurl }}/setup_db/landscape/).

Source: `./ai4sh/import_data/utility/landscape/excel/`

## Required loading sequence

Same dependency order either way — general utilities, then independent/dependent observation
utilities, then the inheritance-dependent ones, then landscape utilities.

**Single step** (default): **[Insert utility data]** — block 1 of `insert_ai4sh_data.ipynb`, translate and insert combined:

1. **Cell 1.1** Load general utilities
2. **Cell 1.2** Load observation utilities
3. **Cell 1.3** Load observation utilities with inheritance
4. **Cell 1.4** Load landscape utility

**2-step** (for `UPDATE`s or manual inspection), 6 cells in `more_notebooks/load_ai4sh_utility_data.ipynb`:

1. **[Translate general utilities]** — Excel → JSON for `territory`
2. **[Manage general utilities]** — Insert `territory` into the database
3. **[Translate observation utilities]** — Excel → JSON for independent and dependent tables
4. **[Manage observation utilities]** — Insert those tables into the database
5. **[Translate observation utilities with inheritance]** — Excel → JSON for `provision`, `provision_indicator`, `provision_serial_nr`
6. **[Manage observation utilities with inheritance]** — Insert those tables into the database

[Insert utility data]: {{ site.baseurl }}/utility/insert_utility_data/
[insert_utility_data]: {{ site.baseurl }}/utility/insert_utility_data/
[insert_vs_translate]: {{ site.baseurl }}/insert_vs_translate/
[Translate general utilities]: {{ site.baseurl }}/utility/translate_general_utilities/
[Manage general utilities]: {{ site.baseurl }}/utility/manage_general_utilities/
[Translate observation utilities]: {{ site.baseurl }}/utility/translate_observation_utilities/
[Manage observation utilities]: {{ site.baseurl }}/utility/manage_observation_utilities/
[Translate observation utilities with inheritance]: {{ site.baseurl }}/utility/translate_observation_utilities_inherit/
[Manage observation utilities with inheritance]: {{ site.baseurl }}/utility/manage_observation_utilities_inherit/
[edna_catalogues]: {{ site.baseurl }}/edna/edna_catalogues/
[all_data]: {{ site.baseurl }}/all_data/
[obs_texture]: {{ site.baseurl }}/observation/#derived-soil-texture-classes
