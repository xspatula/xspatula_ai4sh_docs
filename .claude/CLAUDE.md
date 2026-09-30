# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

`xspatula_ai4sh_docs` is a documentation site written in Markdown using the Jekyll theme Minimal Mistakes (https://mmistakes.github.io/minimal-mistakes/) for the database-integrated Xspatula framework written in python in the sibling repo `xspatula_ai4sh`. This site (`xspatula_ai4sh_docs`) is intended as instructions for how to seed and load soil data for the AI4SH (AI4SoilHealth) project using the Xspatula framework introduced in the sibling directory `xspatula_core_docs`. The earlier version of the CLAUDE.md file is available in `CLAUDE_20260901.md`.

When starting the repo `xspatula_ai4sh` I included public data from the LUCAS sampling campaign with the AI4SH project data. I then created a specific project for the LUCAS data as a public GitHub repo in the sibling folder `xspatula_lucas` with documentation in the sibling repo `xspatula_lucas_docs`. To seed and load this project with LUCAS data is thus no longer needed. The user can simply seed and load LUCAS data from the sibling repo - all that is required is that the same database is used.

## Session tasks

### 1. Remove all LUCAS specific documentation

Remove all LUCAS specific documentation but keep all references to LUCAS in those parts where the documentation in this repo is scanty and refers to the more detailed workflow in `xspatula_lucas_docs`. Also add a section in the landing page (`index.md`) that outlines how to include the LUCAS data (`xspatula_lucas`) with `xspatula_ai4sh` - as outlined under ## Project Purpose.

### 2. Update collection setup_db

Since the last update I added the complete pipeline for defining the database, processes and import functions for eDNA metabarcoding data. This is a very complex process and you find all the steps undertaken under `notes`, in the files with the pattern sibling `edna_bioinformatics*.md` in the sibling folder `xspatula_ai4sh`. The documentation needs a dedicated page to `eDNA metabarcoding` under the setup_db collection - put it last. To fill in the various excel sheets required is not an easy task and perhaps needs some details - after an initial explanation of eDNA and overview of how it is handles in `xspatula_ai4sh`.

Also note there are 2 more schemas (`organism_utlity` and `organism`) that must be added to the documentation.

The inclusion of eDNA metabarcoding also led to changes in the tables and table structure under `observation_utility` (`setup/zzz/ai4sh/setup_db/json/observation_utility` in the sibling repo `xspatula_ai4sh`) as outlined in the page `_setup_db/observation_utility.md`. And also under `observation` (sibling repo) as outlined in the page `_setup_db/observation.md`.

In addition the database setup now also includes tables for the new schemas `organism_utility` and `organism` that should be written in 2 new pages.

### 3. Update collection setup_processes

The changes outlined above (### 2. Update collection setup_db) also affect the content in the collection setup_processes. The pages affected include:

- utility
- observation_utility
- observation

and then there are processes related to the schemas `organism_utility` and `organism` that should be written in 2 new pages.

### 4. Update _pages/loading_data.md

The new eDNA metabarcoding must be added to the landing page for Loading AI4SH data (_pages/loading_data.md).

### 5. New collection for loading eDNA metabarcoding

Following the structure of the "sub-collections" wetlab and spectra also eDNA metabarcoding (simply named `edna`) must be created and added alongside wetlab and spectra.

Note that this "sub-collection" must start with a page (after "synopsis") that describes how to run the CLI `ai4sh/import_data/prepare_ai4sh_data/edna_final_results_to_xspatula.py` in the sibling `xspatula_ai4sh` to generate the required source files that `xspatula_ai4sh` uses. In that page also include which support files are required. I will not supply the actual datafile `ai4sh/import_data/prepare_ai4sh_data/edna/excel/AI4SH_eDNA_Final results.xlsx` in the repo - that file must be put in place by the user. This is a sensitive file with data that is not public and can (will) also change as the software libraries it depends on develop.
