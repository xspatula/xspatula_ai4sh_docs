# Updates 20260815

With the help of claude I have updated the code base in the sibling directory `xspatula_ai4sh` for which this repo is the documentation and manual.

This document contains an overview of the changes that will support you in a step-wise updating and extension of the documentation in this repo vis-a-vis the changes in code repo.

## ./xspatula_ai4sh/.claude/CLUADE.md

The new overall structure is documented in the sibling directory CLAUDE.md file.

## The five major updates

There are 5 (five) major updates in the code base repo that needs to be adjusted in the documentation.

### edna schema and tables

The schemas edna and edna_utility no longer exist, all tables related to edna are now under the schemas observation and observation_utility.

### audit

A systematic audit system has been added, the tables for the auditing are under the schema audit. Details for the setup and functions for the audit system are found in the sibling repo CLAUDE.md file, with details in the file `./xspatula_ai4sh/notes/instructions_for_auditing_20260815.md`

### Organisation and user handling

The handling (INSERT, UPDATE, DELETE) of organisations and users are changed, except for the initial bootstrap user (the user logged in when adding processes). The new solution for managing users (except for the bootstrap) is now via excel file translation and then insertion as for many other data. For the updates I now ask for, a whole new collection is required, while also updating the instructions under setup_db. Details for the setup and functions for the audit system are found in the sibling repo CLAUDE.md file, with details in the file `./xspatula_ai4sh/notes/notes/instructions_for_adding_comminity_20260815.md`

### Path structure

The relative path structure is harmonised between all commands. The new system is oulined in the sibling directory CLAUDE.md file `./xspatula_ai4sh/.claude/CLAUDE.md`

### Changed from arrays to junction tables
Except for observations that are recorded as arrays (e.g. spectra) and associated data, array columns have been changed to separate tables with links to the main table(s). Details are found in the sibling repo CLAUDE.md file, in the file `./xspatula_ai4sh/notes/instructions_for_documenting_array_to_junction_tables_20260815.md`


## Update navigation.yml

The Audit and the changes in Organisation and user handling, requires additional collections and (perhaps) changes in the top-nav structure. The existing top-nav contains 4 entries, I think that 6 are needed as outlined below. But it could be reduced to perhaps 5 by putting audit under setup_db.

### existing top-nav

main:
  - title: "Setup DB"
    url: /setup_db/
    description: "Setting up the AI4SoilHealth database"
  - title: "Setup processes"
    url: /setup_process/
    description: "Setting up the AI4SoilHealth processes"
  - title: "Loading data"
    url: /loading_data/
    description: "Loading AI4SoilHealth data into the database"
  - title: "About"
    url: https://xspatula.github.io/about/

### Suggested new top-nav

main:
  - title: "DB"
    url: /setup_db/
    description: "Setting up the AI4SoilHealth database"
  - title: "audit"
    url: /audit/
    description: "AI4SoilHealth datbase auditing setup and function"
  - title: "community"
    url: /setup_community/
    description: "Setting up the AI4SoilHealth organisations and users"
  - title: "processes"
    url: /setup_process/
    description: "Setting up the AI4SoilHealth processes"
  - title: "Load"
    url: /loading_data/
    description: "Loading AI4SoilHealth data into the database"
  - title: "About"
    url: https://xspatula.github.io/about/

## Claudes update plan

Plan: reflect the 5 code-base updates in xspatula_ai4sh_docs

 Context

 The sibling code repo (now xspatula_ai4sh, formerly load_ai4sh_db) went through 5
 major changes on 2026-08-14/15: the edna/edna_utility schemas were folded into
 observation/observation_utility, a generic audit system was added, organisation/user
 registration moved from hand-edited JSON to an Excel-intake workflow, all relative paths
 were harmonised to resolve against the project root, and FK-shaped array columns were
 replaced with junction tables. for_claude_changes_20260615.md (repo root) is the
 user's own summary of this; three of the five already have complete, ready-to-execute
 authoring briefs written into the code repo's notes/ directory. This plan sequences all
 five into the doc site, reusing those briefs verbatim where they exist.

 User decisions locked in: follow the 3 existing briefs as authoritative specs (spot-check
 referenced source, don't re-derive); the array→junction update becomes a single reference
 page, not a new collection; the top navbar gets 6 entries (DB, Audit, Community,
 Processes, Load, About).

### Work item 1 — edna fold-in

(no brief; do first, it's small and unblocks nothing else)

 Retire the standalone edna pages, merge their content into observation/observation_utility
 docs. Source of truth: commit d5bbc3509 in xspatula_ai4sh folded edna/edna_utility
 into observation/observation_utility — same table names, only the schema changed.
 Tables now created by setup_db/json_ai4sh/observation_utility/edna_v10_sql.json
 (sequence_library, nucleotide_archive, extraction, amplification, purification,
 sequencing, metabarcoding_pipeline, bioinformatics) and
 setup_db/json_ai4sh/observation/edna_v10_sql.json (edna_nucleotide,
 taxa_bioinformatics, edna_measurement, taxa_biodiversity_measurement). Note: the
 5 edna_* setup_processes JSON files exist under
 setup_processes/json_ai4sh/observation_utility/ but are not yet in the process pilot
 list (ai4sh_setup_processes.txt) — document what exists, flag the gap rather than
 inventing process registration that isn't there yet.

 Files to change:
 - Delete _setup_db/edna.md and _setup_processes/edna.md.
 - _setup_db/observation_utility.md — add a new section for the edna reference tables,
 matching the existing macrofauna/monolith_extraction section pattern.
 - _setup_db/observation.md — add a new section for the edna observation tables,
 matching the existing macrofauna/infiltration_beerkan section pattern.
 - _setup_processes/observation_utility.md and _setup_processes/observation.md — add
 corresponding process notes (including the "not yet in pilot list" caveat).
 - _setup_db/schemas.md — remove the create_schema rows/table rows for
 edna/edna_utility (lines ~31-32, 48-49) and the edna_utility ← edna dependency
 line (~61).
 - _setup_db/introduction.md (line ~137) and _setup_processes/introduction.md (line
 ~52) — drop the standalone edna/ folder bullet.
 - index.md — remove the edna_utility/edna schema bullets.
 - _data/navigation.yml — remove the "eDNA" sidebar entries under setup_db: and
 setup_processes:.
 - .claude/CLAUDE.md (this repo's, not the code repo's) — drop edna from the
 setup_db/setup_processes page lists in the Navigation Structure section.

### Work item 2 — Auditing collection

 Brief: ../xspatula_ai4sh/notes/instructions_for_auditing_20260815.md (already read in
 full). New collection auditing (_auditing/), 2 pages minimum, mirroring
 setup_db/setup_processes frontmatter shape. Page 1: the mechanism (native Postgres
 triggers, audit.if_modified_func(), audit.logged_actions columns including the new
 changed_by_user_id, the two-tier INSERT-coverage table, the self-audit recursion
 gotcha, login_evaluation grants, config file map). Page 2: DBeaver-ready SQL queries,
 7 in order of the brief (recent changes, per-table, real-changes-only via
 IS DISTINCT FROM, JSONB field diff, who-did-it join, per-transaction, what's excluded).

 Site plumbing: _config.yml — add auditing: { output: true } to collections: plus a
 defaults scope entry copying the setup_processes block. _data/navigation.yml — new
 auditing: sidebar block, one child per page; add "Audit" to the top main: navbar
 (after "Setup DB", per the 6-entry decision).

 Work item 3 — Community/user registration collection

 Brief: ../xspatula_ai4sh/notes/instructions_for_adding_comminity_20260815.md (already
 read in full). New collection setup_community (_setup_community/): introduction.md
 plus bootstrap_user.md, smtp_email.md, excel_intake.md, register_notebook.md,
 welcome_email.md (5 pages). Source material already verified present:
 ../xspatula_ai4sh/notes/ai4sh_organisation+user+password_setting.md (bootstrap password
 section at line 170) and src/community/registration.py for the welcome-email template —
 read these two files in full when writing those two pages.

 Also fix _setup_processes/community.md: it currently claims no separate community
 process registration exists — that's stale. Update per the brief section 2 (new
 organisation_v10_sql.json/user_v10_sql.json registrations, wired into the pilot list
 under a ### COMMUNITY ### section), and add a forward pointer to the new
 setup_community collection.

 Site plumbing: _config.yml — add setup_community: { output: true } +
 defaults scope entry. _data/navigation.yml — new setup_community: sidebar block (one
 child per page) and a "Community" main-navbar entry (after "Audit", before "Setup
 processes", per the 6-entry decision). Permalink /setup_community/ (brief explicitly
 flags not to propagate the site's existing singular/plural inconsistency).

### Work item 4 — Path resolution harmonisation

 No collection-level brief; based on my own Explore findings. Authoritative rule to quote
 verbatim from ../xspatula_ai4sh/.claude/CLAUDE.md ("### Path resolution" section):
 every relative path in the scheme → job → pilot → process chain resolves against the
 project root (including, as of 2026-08-15, a process file's own tabular_data_path/
 dst_path), except scheme_file (resolves against the calling notebook's cwd) and
 project_path inside the scheme file (resolves against the scheme file's own directory).
 Also document the pilot_file-vs-in-pilot-paths asymmetry. src/lib/pilot.py's
 Get_project_path docstring (lines 43-114) is good source material for the 5 notation
 styles (../, ./, ~/, absolute, bare).

 New page _setup_processes/path_resolution.md, linked from _setup_processes/translate.md
 (whose parameter-hint table currently just says "Absolute or relative path" with no
 anchor stated — expand that line to point at the new page). Add a sidebar child entry
 under the existing setup_processes: nav block; no new navbar entry needed.

 Fix stale ../-depth-counted path examples (now wrong under the project-root rule) in:
 - _dataset_meta/translate_dataset.md
 - _wetlab/translate_ai4sh_wetlab_data.md
 - _sample/translate_geotag_sample.md
 - _utility/translate_general_utilities.md
 - _utility/translate_observation_utilities.md
 - _utility/translate_observation_utilities_inherit.md

 Rewrite each example's tabular_data_path/dst_path as a bare project-root-relative
 path (e.g. import_data/dataset/excel/data_source.xlsx), matching how job_folder is
 already shown correctly in the same pages.

### Work item 5 — Array-to-junction-tables reference page

 Brief: ../xspatula_ai4sh/notes/instructions_for_documenting_array_to_junction_tables_20260815.md
 (already read in full). Per user decision, one new page (not a collection) — place under
 _setup_processes/ (e.g. _setup_processes/schema_conventions.md), since it's the
 existing home for process/schema-design conventions and needs no new _config.yml/nav
 scaffolding beyond a sidebar child entry.

 Cover per the brief: the referential-integrity problem with FK-shaped arrays, the
 junction-table fix, the boolean-checkbox special case
 (landscape.cultivation_soil_preparation), free-text tags getting the same treatment
 (campaign_tag/dataset_tag), what's deliberately excluded (ordered numeric series:
 observation_measurement_array, spectral wavelength/signal arrays) and why, the 4-table
 reference table from the brief, why independent manage_* processes rather than the
 _array-parameter fan-out mechanism, pilot-list wiring notes, and the UPDATE/DELETE-only
 audit coverage (cross-link to the new auditing collection instead of re-explaining).

### Sequencing

 Do them in list order (1 → 5): edna fold-in is small and independent; auditing and
 community are the two full collections with the most explicit briefs; path resolution
 touches many existing pages so benefits from the new collections already existing to
 cross-link into; the junction-tables page can cross-link to auditing, so goes last.

### Verification

 - bundle exec jekyll serve --config _config.yml,_config_local.yml after each work item
 (or once at the end) — confirm the site builds without Liquid/front-matter errors.
 - Confirm the top navbar renders all 6 entries in order: Setup DB, Audit, Community,
 Setup processes, Loading data, About.
 - Confirm every new/edited sidebar nav key in _data/navigation.yml matches a page's
 sidebar.nav front matter value exactly (typo here silently drops the sidebar).
 - Grep the whole repo for edna after work item 1 and confirm zero remaining hits
 outside _site/ (build output) and for_claude_changes_20260615.md (the source brief,
 left as-is).
 - Spot-check 2-3 rewritten path examples against src/lib/pilot.py's actual resolution
 logic in the code repo to confirm the project-root-relative rewrite is correct.
