# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

`xspatula_ai4sh_docs` is a documentation site written in Markdown using the Jekyll theme Minimal Mistakes (https://mmistakes.github.io/minimal-mistakes/) for the database-integrated Xspatula framework written in python in the sibling repo `xspatula_ai4sh`. This site (`xspatula_ai4sh_docs`) is intended as instructions for how to seed and load soil data for the AI4SH (AI4SoilHealth) project using the Xspatula framework introduced in the sibling directory `xspatula_core_docs`. The earlier version of the CLAUDE.md file is available in `CLAUDE_20260901.md`.


## Session tasks 1. Expandable figures - DONE

Figures are small and difficult to read. Please add expansion settings to all figures using the syntax solution I created for the figure Excel columns to utility.territory in `_setup_processes/utility.md`:

[![Excel columns to utility.territory]({{ "/assets/media/process_mapping/territory.png" | relative_url }})]({{ "/assets/media/process_mapping/territory.png" | relative_url }})

## Session task 2. Table layout

I made a different layout ot the a table in  `_setup_db/observation_utility.md`. It became much more readable for us that use eyes (smaller fonts that make the columns fit better). Please implement the same solution for all tables across the whole site. Here is the principle I used:

{% capture notice-2 %}
| File | Tables created | Description |
|---|---|---|
| `preservation_v10_sql.json` | `preservation` | Sample preservation method (cold, frozen, chemical shield, etc.) |

{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>
