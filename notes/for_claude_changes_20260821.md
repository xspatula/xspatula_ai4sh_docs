# Updates 20260821

With the help of claude I have updated the code base in the sibling directory `xspatula_ai4sh` for which this repo is the documentation and manual.

This document contains an overview of the changes on top of the changes in `for_claude_changes_20260615.md`, changes that we already managed.

### audit

A semi-automated audit setup system has been created. This requires at least one more page under the collection "auditing" (called e.g. setup or audit setup). Details for the original setup and functions for the audit system are found in the sibling repo CLAUDE.md file, with details in the file `./xspatula_ai4sh/notes/instructions_for_auditing_20260815.md`; the update with a semi-automated setup is in the file `./xspatula_ai4sh/notes/instructions_for_auditing_20260819.md`; claudes plan for building the semi-automated audit setup system is in `./xspatula_ai4sh/notes/claudes_plan_automatic_audit_setup.md`.

### Single step tabular insert

The 2-step process of loading data from excel was complemented with a single step process (at least as being performed from a user perspective). The new single step system should be the main route used in the documentation under all pages in the collection loading_data. But then also add one more page that describes both systems (single step vs dual step) for loading_data - as some data come as excel/csv and some other data as prepared JSON files. The instructions for the documentation of this part are in the file `./xspatula_ai4sh/notes/instructions_for_tabular_insert_20260819.md`.
