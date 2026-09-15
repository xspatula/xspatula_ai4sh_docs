Implementation Plan

 Step 1: Build new site only locally in "/Users/thomasgumbricht/GitHub_xspatula/xspatula_ai4sh_docs/", copy all required as needed, allows me to compare the new and old before acceping and pushing.

 Step 2: Update _config.yml

 File: _config.yml
 - baseurl: /load_ai4sh_db_docs → baseurl: /ai4sh_xspatula_docs
 - repository: xspatula/load_ai4sh_db_docs → repository: xspatula/ai4sh_xspatula_docs

 Step 3: Add a "Loading data" landing page

 Create _pages/loading_data.md (or a new minimal collection _loading_data/index.md) that:
 - Lists all 6 loading sections (Utility, Dataset meta, Sample, Wetlab, Spectra, All) with brief descriptions and links to their synopsis pages
 - Uses permalink: /loading_data/

 Alternative (simpler): Add an index.md to a new _loading_data/ collection with just one page.

 Step 4: Update _data/navigation.yml

 Replace the 6 individual top-nav entries (Utility, Dataset meta, Sample, Wetlab, Spectra, All) with one:

 main:
   - title: "Setup DB"
     url: /setup_db/
   - title: "Setup processes"
     url: /setup_processes/
   - title: "Loading data"
     url: /loading_data/
   - title: "About"
     url: https://xspatula.github.io/about/

 The sidebar nav keys for each loading section (utility, dataset_meta, sample, wetlab, spectra, all_data) remain unchanged — pages within each section still use
 their own sidebar.

 Step 5: Update _config_local.yml

 Check for any hardcoded load_ai4sh_db_docs references and update.

 Step 6: Update CLAUDE.md

 Update .claude/CLAUDE.md references: repo name, GitHub URL, site URL.

 Step 7: Check for hardcoded URLs in collection files

 Grep for load_ai4sh_db_docs across all .md files and update.

 What Does NOT Change

 - Collection directories (_setup_db/, _utility/, etc.) — no rename needed
 - Individual page content and front matter
 - Sidebar nav definitions for each collection
 - Gemfile, assets, theme

 Verification

 1. bundle exec jekyll serve --config _config.yml,_config_local.yml — confirm local build succeeds
 2. Confirm top navbar shows 5 entries: Setup DB, Setup processes, Loading data, Xspatula, About (+ Core DB docs link)
 3. Confirm "Loading data" links to the landing page, which links to each section
 4. Confirm each section's sidebar nav still works
 5. After GitHub repo rename, confirm https://xspatula.github.io/ai4sh_xspatula_docs/ resolves