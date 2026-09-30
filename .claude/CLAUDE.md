# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

`xspatula_ai4sh_docs` is a documentation site written in Markdown using the Jekyll theme Minimal Mistakes (https://mmistakes.github.io/minimal-mistakes/) for the database-integrated Xspatula framework written in python in the sibling repo `xspatula_ai4sh`. This site (`xspatula_ai4sh_docs`) is intended as instructions for how to seed and load soil data for the AI4SH (AI4SoilHealth) project using the Xspatula framework introduced in the sibling directory `xspatula_core_docs`. The earlier version of the CLAUDE.md file is available in `CLAUDE_20260901.md`.


## Session tasks

### Previous and Next buttons

Fix the page order for previous and in the sibling `xspatula_lucas_docs` (`_data/story_order.yml`). Let them follow the collection order.

### Harmonise links for `_setup_db/schemas.md`

 `_setup_db/schemas.md` - under schema overview, links for organism/_utility, but no other. Either links for all or none.

### Add DB diagrams in some setup_db pages (if possible)

 Create DB diagram figures of selected parts of the database for more pages (as already done in e.g. `community`:
 - 1 image for the `processes` (_setup_db/process.md)
 - 1 (more) image for eDNA method catalogues and method pipelines in _setup_db/observation_utility.md
- 1 (more) image for eDNA observation tables in in _setup_db/observation.md
- 1 image for organism_utility schema in _setup_db/organism_utility.md.
- 2 images, one for landscape_utility and one for landscape in _setup_db/landscape.md.
- 1 image converting "How the tables connect" to a diagram in _setup_db/edna_metabarcoding.md
