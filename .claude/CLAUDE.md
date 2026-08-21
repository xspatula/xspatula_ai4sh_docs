# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

`xspatula_ai4sh_docs` is a documentation site written in Markdown using the Jekyll theme Minimal Mistakes (https://mmistakes.github.io/minimal-mistakes/) for the database-integrated Xspatula framework written in python in the sibling repo `load_ai4sh_db`. This site (`xspatula_ai4sh_docs`) is intended as instructions for how to seed and load soil data for the AI4SH (AI4SoilHealth) project using the Xspatula framework introduced in the sibling directory `setup_core_db_docs`. The seeding documentation for the AI4SH framework project (sibling: `seed_ai4sh_db_doc`) was done for an earlier version of the framework (`seed_ai4sh_db`). The content of `seed_ai4sh_db_docs` should be transferred to this new documentation repo while being updated. I left the repo `seed_ai4sh_db` intact so you can compare the differences and what needs updating compared to the new `load_ai4sh_db` repo that is being documented in this repo.



The Xspatula framework is written in Python and uses JSON files to define all executions and processes. These JSON files are called from Jupyter notebooks. The AI4SH framework is available:
- As a sibling directory at relative path `../load_ai4sh_db`
- On GitHub at https://github.com/xspatula/load_ai4sh_db

## Site Architecture

**Theme**: Minimal Mistakes Jekyll theme, version 4.27.3 (local install, not remote)
**URL (production)**: `https://xspatula.github.io/xspatula_ai4sh_docs/`
**Search**: Lunr (client-side, full content)

### Collections

| Collection | Directory | Output | Purpose |
|---|---|---|---|
| `setup_db` | `_setup_db/` | true | Step-by-step database setup guide |
| `setup_processes` | `_setup_processes/` | true | Step-by-step database processes setup guide |
| `utility` | `_utility/` | true | Utility data translation and insertion guide |
| `dataset_meta` | `_dataset_meta/` | true | Dataset metadata translation and insertion guide |
| `sample` | `_sample/` | true | Sample data translation and insertion guide |
| `wetlab` | `_wetlab/` | true | Wet laboratory data translation and insertion guide |
| `spectra` | `_spectra/` | true | Spectral data insertion guide |
| `all_data` | `_all_data/` | true | Single-notebook guide for loading all data |

### Navigation Structure in repo `seed_ai4sh_db_docs`

```
Top navbar:
├─ Setup AI4SoilHealth DB   →  _db/
├─ Setup AI4SoilHealth processes   →  _processes/
└─ Import AI4SoilHealth data   →  _data/

Setup DB subsection (9 pages):
  introduction, schemas, process, community, utility, observation_utility, observation, landscape, edna

Setup DB processes subsection (8 pages):
    introduction, root_process, translate, community, utility, observation_utility, observation, edna

data subsection (8 pages):
        introduction, tabular_source_data, JSON_data_structure, import_utility, manage_utility, import_observation, manage_observation, foreign_key_handling
```

The repo `seed_ai4sh_db_docs` only covers the dataset meta data loading. The structure for some of the commands has since been updated from the repo `seed_ai4sh_db` in the repo `load_ai4sh_db`. The `_import_data` collection in `seed_ai4sh_db_docs` was an embryo for the additional loading of data covered in this repo. Use the layout and general content of the `_import_data` collection in `seed_ai4sh_db_docs` when creating the collections for `utility`, `dataset_meta`, and other data-loading collections in this repo.

### Navigation Structure this repo

For this documentation repo I would like to create the following navigation structure:

```
Top navbar:
├─ Setup DB   →  _setup_db/
├─ Setup processes   →  _setup_processes/
├─ Utility   →  _utility/
├─ Dataset meta   →  _dataset_meta/
├─ Sample   →  _sample/
├─ Wetlab   →  _wetlab/
├─ Spectra   →  _spectra/
└─ All →  _all_data/

Setup DB subsection (8 pages):
  introduction, schemas, process, community, utility, observation_utility, observation, landscape

Setup DB processes  subsection(7 pages):
    introduction, root_process, translate, community, utility, observation_utility, observation

Utility subsection (8 pages):
  synopsis, insert_utility_data, translate_general_utilities, manage_general_utilities, translate_observation_utilities, manage_observation_utilities, translate_observation_utilities_inherit, manage_observation_utilities_inherit

Dataset meta (8 pages):
    synopsis, insert_dataset_meta, translate_dataset, manage_data_source, manage_person, manage_dataset, manage_campaign, manage_sampling_log

Sample subsection (5 pages):
        synopsis, insert_sample_data, translate_geotag_sample, manage_sample_geolocation, manage_sample

Wetlab subsection (5 pages):
        synopsis, insert_wetlab_data, translate_ai4sh_wetlab_data, manage_ai4sh_agrolab_obs_log, manage_ai4sh_wetlab_data

Spectra subsection (13 pages):
        synopsis, manage_foss_ds2500_spectrometer, manage_foss_ds2500_obs_log, manage_foss_ds2500_obs, manage_neospectra_spectrometer, manage_neospectra_obs_log, manage_neospectra_obs, manage_ftir_spectrometer, manage_ftir_obs_log, manage_ftir_obs, manage_libs_spectrometer, manage_libs_obs_log, manage_libs_obs

All subsection (1 page):
        synopsis
```

### Pages in this repo

In the table below I have outlined the expanded subsections and pages that I want to include in this documentation repo compared to `seed_ai4sh_db_docs`.

| Subsection | page | notebook | notebook cell | purpose |
|---|---|---|---|---|
| Utility | synopsis | load_ai4sh_utility_data | NONE | Overview of 1: utility data types, and 2: required sequence of inserting data |
| Utility | insert_utility_data | insert_ai4sh_utility_data | Insert general/observation/observation-inherit utilities | Single-step insert_tabular_data alternative to the 6-cell translate/manage route |
| Utility | translate_general_utilities | load_ai4sh_utility_data | Translate general utilities | List of general utility excel files and content, how to run |
| Utility | manage_general_utilities | load_ai4sh_utility_data | Manage general utilities | show single JSON file example, how to run |
| Utility | translate_observation_utilities | load_ai4sh_utility_data | Translate observation utilities | List of observation utility excel files and content, how to run |
| Utility | manage_observation_utilities | load_ai4sh_utility_data | Manage observation utilities | show single JSON file example, how to run |
| Utility | translate_observation_utilities_inherit | load_ai4sh_utility_data | Translate observation utilities with inheritance | List of observation utility inheritance excel files and content, how to run |
| Utility | manage_observation_utilities_inherit | load_ai4sh_utility_data | Manage observation utilities with inheritance | show single JSON file example, how to run |
|   |   |   |   |   |   
| Dataset meta | synopsis | load_ai4sh_dataset_meta | NONE | Overview of 1: dataset meta files, and 2: required sequence of inserting data |
| Dataset meta | insert_dataset_meta | insert_ai4sh_dataset_meta | Insert data source/persons/dataset/campaigns/sampling logs | Single-step insert_tabular_data alternative, one process_file per table |
| Dataset meta | translate_dataset | load_ai4sh_dataset_meta | Translate dataset | List of dataset excel files and content, how to run |
| Dataset meta | manage_data_source | load_ai4sh_dataset_meta | Manage data source | show single JSON file example, how to run |
| Dataset meta | manage_person | load_ai4sh_dataset_meta | Manage persons | show single JSON file example, existing DB records required|
| Dataset meta | manage_dataset | load_ai4sh_dataset_meta | Manage dataset | show single JSON file example, existing DB records required |
| Dataset meta | manage_campaign | load_ai4sh_dataset_meta | Manage campaign | show single JSON file example, existing DB records required |
| Dataset meta | manage_sampling_log | load_ai4sh_dataset_meta | Manage sampling logs | show single JSON file example, existing DB records required |
|   |   |   |   |   |   
| Sample | synopsis | load_ai4sh_sample_data | NONE | Overview of 1: sample files, and 2: required sequence of inserting data |
| Sample | insert_sample_data | insert_ai4sh_sample_data | Insert sample geotag data | Single-step insert_tabular_data alternative, one combined process_file |
| Sample | translate_geotag_sample | load_ai4sh_sample_data | Translate geotag sample | List of dataset excel files and content, how to run |
| Sample | manage_sample_geolocation | load_ai4sh_sample_data | Manage sample location | show single JSON file example, how to run |
| Sample | manage_sample | load_ai4sh_sample_data | Manage samples | show single JSON file example, existing DB records required |
|   |   |   |   |   |   
| Wetlab | synopsis | load_ai4sh_wetlab_data | NONE | Overview of 1: wetlab files, and 2: required sequence of inserting data |
| Wetlab | insert_wetlab_data | insert_ai4sh_wetlab_data | Insert Agrolab observation log/observation | Single-step insert_tabular_data alternative, one process_file per table |
| Wetlab | translate_ai4sh_wetlab_data | load_ai4sh_wetlab_data | Translate Agrolab (wetlab) Observation log | List of dataset excel files and content, explain the purpose of the observation log (linking to provision and provision_indicators), explain the structure of observation excel file (with "@" signs before the values and how single number entries and arrays are defined) |
| Wetlab | manage_ai4sh_agrolab_obs_log | load_ai4sh_wetlab_data | Manage Agrolab observation log | show single JSON file example, existing DB records required |
| Wetlab | manage_ai4sh_wetlab_data | load_ai4sh_wetlab_data | Manage Agrolab observation | show single JSON file example, existing DB records required |
|   |   |   |   |   |   
| Spectra | synopsis | load_ai4sh_spectral_data | NONE | Overview of spectral data available, all data converted to and stored as ascending reflectance in the database |
| Spectra | manage_foss_ds2500_spectrometer | load_ai4sh_spectral_data | Manage FOSS DS2500 spectrometer | Summary of the FOSS DS2500 spectrometer and its data range |
| Spectra | manage_foss_ds2500_obs_log | load_ai4sh_spectral_data | Manage FOSS DS2500 observation log | show single JSON file example, existing DB records required |
| Spectra | manage_foss_ds2500_obs | load_ai4sh_spectral_data | Manage FOSS DS2500 observation | show single JSON file example and cut the spectral data array to 4 items, existing DB records required |
| Spectra | manage_neospectra_spectrometer | load_ai4sh_spectral_data | Manage Neospectra spectrometer | Summary of neospectra spectrometer and its data range |
| Spectra | manage_neospectra_obs_log | load_ai4sh_spectral_data | Manage Neospectra observation log | show single JSON file example, existing DB records required |
| Spectra | manage_neospectra_obs | load_ai4sh_spectral_data | Manage Neospectra observation | show single JSON file example and cut the spectral data array to 4 items, existing DB records required |
| Spectra | manage_ftir_spectrometer | load_ai4sh_spectral_data | Manage FTIR spectrometer | Summary of FTIR spectrometer and its data range, data stored as ascending reflectance compared to ascending in JSON file |
| Spectra | manage_ftir_obs_log | load_ai4sh_spectral_data | Manage FTIR observation log | show single JSON file example, existing DB records required |
| Spectra | manage_ftir_obs | load_ai4sh_spectral_data | Manage FTIR observation | show single JSON file example and cut the spectral data array to 4 items, existing DB records required |
| Spectra | manage_libs_spectrometer | load_ai4sh_spectral_data | Manage LIBS spectrometer | Summary of LIBS spectrometer and its data range, data stored as ascending reflectance compared to ascending in JSON file |
| Spectra | manage_libs_obs_log | load_ai4sh_spectral_data | Manage LIBS observation log | show single JSON file example, existing DB records required |
| Spectra | manage_libs_obs | load_ai4sh_spectral_data | Manage LIBS observation | show single JSON file example and cut the spectral data array to 4 items, existing DB records required |
|   |   |   |   |   |   
| All | synopsis | load_ai4sh_data | load_ai4sh_data | Loading all data from a single Jupyter Notebook in one go |

### Key Directories

- `_data/` — `navigation.yml` (site nav) and `ui-text.yml` (theme UI strings)
- `_setup_db/` — database setup collection (9 pages)
- `_setup_processes/` — process setup collection (8 pages)
- `_utility/` — utility data loading collection (7 pages)
- `_dataset_meta/` — dataset metadata loading collection (7 pages)
- `_sample/` — sample data loading collection (4 pages)
- `_wetlab/` — wet laboratory data loading collection (4 pages)
- `_spectra/` — spectral data loading collection (13 pages)
- `_all_data/` — single-notebook all-data loading collection (1 page)
- `assets/media/` — logos and schema diagram images
- `_site/` — generated output (do not edit manually)

## Configuration Files

- `_config.yml` — production config; sets site title, URL, collections, plugins
- `_config_local.yml` — local dev override; sets empty URL/baseurl for local serving
- `Gemfile` — single gem dependency: `minimal-mistakes-jekyll 4.27.3`

To serve locally: `bundle exec jekyll serve --config _config.yml,_config_local.yml`

## Content Conventions

- All documentation pages use Jekyll front matter with at minimum: `layout`, `title`, `categories`, `tags`
- Sidebar navigation is set via `sidebar: { nav: "<nav_key>" }` in front matter
- Internal links use Jekyll's `{{ site.baseurl }}` or relative paths
- Code examples are fenced with language identifiers (python, json, sql, bash)
- Database schema diagrams are stored in `assets/media/` and referenced in markdown

## Content Scope

The documentation covers:
1. **Loading AI4SH utility data** — translating and inserting utility data, 7 pages
2. **Loading AI4SH dataset meta data** - translating and inserting dataset meta-data, 7 pages
3. **Loading AI4SH sample data** — translating and inserting sample data, 4 pages
4. **Loading AI4SH wet laboratory data** — translating and inserting wetlab data, 4 pages
5. **Loading AI4SH spectral data** — inserting spectral data, 13 pages
6. **Loading all AI4SH data in one go** Single notebook for adding all data in one go, 1 page

## Licenses

- **Data**: Creative Commons Attribution (CC-BY)
- **Code**: MIT License

## Related Repositories

| Repo | Relationship |
|---|---|
| `xspatula/load_ai4sh_db` | The Python framework being documented; sibling directory `../load_ai4sh_db` |
| `xspatula/setup_core_db` | The core Xspatula framework (code); sibling directory `../setup_core_db` |
| `xspatula/setup_core_db_docs` | Documentation of the core Xspatula framework; sibling directory `../setup_core_db_docs` |
| `xspatula/seed_ai4sh_db` | Earlier version of the AI4SH framework (code); sibling directory `../seed_ai4sh_db` |
| `xspatula/seed_ai4sh_db_docs` | Documentation of earlier AI4SH version; the `_import_data` collection is the embryo for this repo's new collections; sibling directory `../seed_ai4sh_db_docs` |
| `mmistakes/minimal-mistakes` | Jekyll theme used for the site |

## Important notes

`seed_ai4sh_db_docs` includes the collection `_import_data/` that was a first version of the more complete data loading covered in this repo. The repo `load_ai4sh_db` includes minor updates related also to the collections `setup_db` and `setup_processes` as documented in `seed_ai4sh_db_docs`. The content of the collections in `seed_ai4sh_db_docs` are thus partly outdated. The content that is now in `seed_ai4sh_db_docs` should become part of `xspatula_ai4sh_docs`, with updates where required. The outline and general content of the collections in `seed_ai4sh_db_docs` are good; only the details on paths, JSON definitions, and Jupyter notebook syntax structure need updating.

The notebook `load_ai4sh_LUCAS_data.ipynb` in `load_ai4sh_db` is **out of scope** for this documentation site and should be ignored.
