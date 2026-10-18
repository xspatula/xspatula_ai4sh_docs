# Updates per page 20260815

This repo is the documentation for the sibling code page `xspatula_ai4sh`.

This document contains a page by page review of what needs to be updated in this documentation repo regarding setting up the database.

## General stuff for you to remember

1. audit is a framework default schema,
2. the page for GitHub downloading should still be https://github.com/xspatula/load_ai4sh_db


## index.md (landing page)

- Outline of the AI4SH postgreSQL database: add audit schema to list and correct the number of schemas.
- Data loading overview: Change the references to notebooks to reflect the new "insert_ai4sh_<>.ipynb" solution.
- Two-step pattern: Expand to include the new single-step solution and then just link to the page you created for the 2-step process (_pages/insert_vs_translate.md).

## _setup_db/introduction.md

- The setup notebook: this notebook now has three code block.
- The pilot file: There are now 2 pilot files, one for the core DB and one for the audit table. In the core DB pilot file check the number of schemas; add a short paragraph on how the audit table pilot file and commands are auto-generated when running the core DB cell and then the option of creating the audit structure by running the last cell in the notebook. My idea for solution: an introductory paragraph on the two cells, then keep the walk through of the core DN and then add the details on the audit system and notebook cell.

## _setup_db/schemas.md

- AI4SH Database Schemas: check number of schemas
- The schema process file: check the code block

## _setup_db/schemas.md

- Community Schema: add a paragraph on the dual way to add users: at setup_db and through excel/csv files as outlined in the setup_community collection - and link to main Setup Community introduction page (_setup_community/introduction.md.

## _setup_db/process.md
- Process files — edit before runningPermalink: Check the number of tables (10 I think, not 8)

## _setup_db/observation_utility.md

- Observation Utility Schema: check the number of tables, are there still 38 tables (excluding eDNA that is listed later)?
- Process files — independent tables: check the list and add missing if found.
- Process files — tables with internal dependencies: check the list and add missing if found.
- eDNA reference cataloguesPermalink: check the list and add missing if found.
- Key concepts - Provision: can you draw a diagram and insert as image?
- Key concepts - Indicator: can you draw a diagram and insert as image, or do one image that covers both provision and indicator?

If you can draw an image put it under a new folder assets/media/<collection>/image.png (or similar solution).

## _setup_db/observation.md

- Observation Schema: expand to explain how observation_log is also linked, perhaps you need to make 2 separate paragraphs. If possible also draw a diagram (or 2 if sampling_log and observation_log requires that). Put any drawing as suggested above.
- Process files: check the list and add missing if found.
- eDNA observation tables: check the list and add missing if found.
- Not yet wired into process setup: check also these files, otherwise keep as is.
- Key tables in detail - campaign: check the table companion tables after the _array breakout we did.
- Specialised observation tables: add spectral data, or create a specific heading that describes spectral data as it is different; also check the edna_<> tables that have been altered.

## _setup_db/landscape.md

- Process files: update the process files - there are now 6 instead of 2
- Schema: landscape_utilityPermalink: expand to reflect the added tables
- Schema: landscape: expand to reflect the added tables

## _auditing/introduction.md

- Auditing: Start with a one line paragraph that explains what auditing is, then keep the rest as is.

## _auditing/setup.md

- Auditing Setup: skip the history (only me ever saw it) and jump directly to the automated system
- Config files (generated, not hand-edited): Heading misnomer - There are 3 audit files included by default in xspatula (audit_table_v10_sql.json, audit_function_v10_sql.json and audit_triggers_audit_v10_sql.json), when running setup_db the audit JSOm command files are automatically written per schema (audit_triggers_<schema>_v10_sql.json). This is not clear. I suggest you do 2 tables, one with the default included JSON command files, and a table with those actually generated when running setup_db.
