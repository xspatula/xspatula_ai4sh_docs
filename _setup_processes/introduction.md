---
title: "Setup AI4SH Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "The second stage of seeding the AI4SH database registers all operational processes into the process schema. This enables the framework to handle data insertion, update and deletion for every schema in the database."
permalink: /setup_process/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

After the database schemas and tables are created (see [Setup DB][setup_db]), the second stage registers all operational processes into the `process` schema. These process definitions tell the framework what operations are available, what parameters they accept, and what permissions apply.

Without this stage, the framework has a working database structure but no knowledge of what it can do with it. Process registration is what connects the database tables to callable Python functions.

**Alternative: restore a premade database.** A restored AI4SH dump already holds all registered processes; skip this stage. See [Dump once, restore in minutes][ai4sh_dump].
{: .notice--info}

## The setup processes notebook

The process setup is driven by the Jupyter notebook:

```
./setup/setup_processes.ipynb
```

This notebook uses a separate scheme file from the database setup:

```
./setup/zzz/scheme_local_use.json
```

This scheme file uses ordinary user credentials (not the postgreSQL superuser) to connect to the already-created database.

## The pilot file

The job file for process setup points to:

```
./setup/zzz/ai4sh/setup_processes/setup_processes.txt
```

This pilot file lists the process JSON files to execute, grouped by schema. The execution order is:

{% capture notice-2 %}
1. `root_process/` — define the process families
2. `translate/` — the translate and insert tabular data processes
3. `utility/` — foreign key and territory management processes
4. `community/` — organisation and user management processes
5. `observation_utility/` — processes for managing observation utility catalogues, including the eDNA catalogues (software, primer pairs, lab protocols) and method pipelines
6. `organism_utility/` — `manage_taxonomy_reference` and the bulk taxon loader `manage_taxon`
7. `observation/` — processes for managing observation data, including the bulk eDNA loader `manage_edna_asv`
8. `landscape_utility/` and `landscape/` — landscape processes (see [below](#landscape-processes))
9. `select/` and `pandas/` — processes for selecting and compiling data, and for inspecting the saved files (see [below](#select-compile-and-inspect))
10. `plot/` — processes for plotting indicators and spectra
11. `machine_learning/` — processes for spectral preprocessing, outlier detection, and regression
{% endcapture %}
 
<div class="notice">{{ notice-2 | markdownify }}</div>

## Landscape processes

The landscape processes belong to the `manage_table_data` root (min stratum 4).

| File | Process | Parameters |
|---|---|---|
| `landscape_utility/land_cover_v10_sql.json` | `manage_landcover` | `class_rank_id__class_rank_name`, `parent_landcover_id__landcover_alias`, `name`, `alias`, `tag`, `display_name`, `abstract` |
| `landscape_utility/land_use_v10_sql.json` | `manage_landuse` | `class_rank_id__class_rank_name`, `parent_landuse_id__landuse_alias`, `name`, `alias`, `tag`, `display_name`, `abstract` |
| `landscape_utility/biogeo16_v10_sql.json` | `manage_biogeo16` | `name`, `alias`, `tag`, `display_name`, `abstract` |
| `landscape_utility/*` (other files) | `manage_crop_growth_stage`, `manage_soil_texture_usda`, `manage_soil_texture_isss`, `manage_reference_soil_groups`, `manage_major_landform`, `manage_slope_position`, `manage_sky_condition`, `manage_ground_condition`, `manage_soil_preparation` | `name`, `alias`, `display_name`, `abstract` |
| `landscape/observation_lucc_v10_sql.json` | `manage_land_cover_observation`, `manage_land_use_observation` | `observation_log_id__observation_log_name`, `geolocation_id__geolocation_name`, `landcover_id__landcover_name` / `landuse_id__landuse_name`, `observed_at` |
| `landscape/observation_biogeo16_v10_sql.json` | `manage_biogeo16_observation` | `observation_log_id__observation_log_name`, `geolocation_id__geolocation_name`, `biogeo16_id__biogeo16_name`, `observed_at` |

`manage_landcover` and `manage_landuse` replace the former per-rank processes (`manage_landcover_order`, `_family`, `_genus` and the land use equivalents); one process handles a class at any rank. `manage_biogeo16` replaces `manage_biogeo16_eu`. In the observation processes the class value is the class alias (code), e.g. `b11`, and it may be a class at any rank. The `class_rank` catalogue is seeded by the database setup and has no process. See [Landscape schema][setup_db_landscape].

## Select, compile and inspect

The `select_data` root holds the processes that read from the database and save local Parquet files for machine learning; `pandas` is a separate root for inspecting such files. All except `inspect_pandas_dataset` (stratum 4) run at stratum 1.

| File | Process | Purpose |
|---|---|---|
| `select/select_data_v10_sql.json` | `select_spectra` | Spectra, indicators and optional covariates in one call, saved as one file |
| `select/select_data_v10_sql.json` | `select_indicator` | List the indicators recorded under a dataset or campaign |
| `select/select_spectra_only_v10_sql.json` | `select_spectra_only` | Spectra only, resampled to the output bandwidth |
| `select/select_indicator_only_v10_sql.json` | `select_indicator_only` | Indicators (target features) only, translated to target units |
| `select/select_covariate_only_v10_sql.json` | `select_covariate_only` | Covariates other than spectra (e.g. field pH, land cover, eDNA taxa) |
| `select/compile_dataset_v10_sql.json` | `compile_dataset` | Join saved spectra/indicator/covariate files on `sample_id`, keeping samples present in all inputs |
| `pandas/pandas_inspect_v10_sql.json` | `inspect_pandas_dataset` | Show the structure and contents of a saved Parquet file |

`select_spectra` takes the parameters `dataset`, `campaign`, `sampling_log`, `spectra_provision`, `spectra_preparation_name`, `output_spectra_signal`, `covariate_array` and `covariate_rank` (the land class rank to report a covariate at), among others. `plot_indicators` (in `plot/`) takes `covariate_array`, `max_classes`, `class_label`, `single_canvas_plot` and `individual_canvas_plot`. The pipeline is described with LUCAS examples in [Explore and select data][lucas_explore_select].

## Process file structure

Every process definition JSON file follows the same structure. A process is registered using the `manage_process` action with a `parameters` block describing the operation, and a `nodes` block describing each parameter it accepts:

```json
{
  "process": [
    {
      "process": "add_process",
      "overwrite": false,
      "parameters": {
        "root_process": "manage_table_data",
        "process": "manage_apparatus",
        "min_user_stratum": 4,
        "title": "Manage apparatus type",
        "label": "Insert, update or delete a generic apparatus type used for observing properties"
      },
      "nodes": [
        {
          "parent": "process",
          "element": "parameters",
          "parameter": [
            {
              "parameter": "name",
              "parameter_type": "text",
              "required": true,
              "default_value": "",
              "hint": "Generic name of the apparatus",
              "schema_table": {
                "schema": "observation_utility",
                "table": "apparatus",
                "write": true
              },
              "permission": {
                "update": false,
                "delete": false
              },
              {
                "parameter": "..."
              },
              {
                "parameter": "..."
              }
            }
          ]
        }
      ]
    }
  ]
}
```

Key fields:

| Field | Description |
|---|---|
| `root_process` | The process family this operation belongs to |
| `process` | Unique process identifier |
| `min_user_stratum` | Minimum user access level required to run this process |
| `title` | Short human-readable name |
| `label` | Longer description |
| `parameter` | Parameter name (matches the column name in the target table) |
| `parameter_type` | Data type: `text`, `integer`, `real`, `boolean` |
| `required` | Whether the parameter must be supplied |
| `schema_table` | The target database table this parameter writes to |
| `permission.update` | Whether the value can be changed after initial insertion |
| `permission.delete` | Whether the record can be deleted |


[setup_db]: {{ site.baseurl }}/setup_db/
[ai4sh_dump]: {{ site.baseurl }}/all_data/#dump-once-restore-in-minutes
[setup_process_observation_utility]: {{ site.baseurl }}/setup_process/observation_utility/
[setup_process_organism_utility]: {{ site.baseurl }}/setup_process/organism_utility/
[setup_process_observation]: {{ site.baseurl }}/setup_process/observation/
[setup_db_landscape]: {{ site.baseurl }}/setup_db/landscape/
[lucas_explore_select]: https://xspatula.github.io/xspatula_lucas_docs/lucas_2009/machine_learning/explore_select_data/
