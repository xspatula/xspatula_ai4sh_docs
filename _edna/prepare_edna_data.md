---
title: "Prepare eDNA Data"
layout: single
sidebar:
  nav: "edna"
  nav2: "loading_data"
excerpt: "Run edna_final_results_to_xspatula.py to convert the laboratory's eDNA master workbook into the six xspatula source files, using three hand-maintained translator files."
permalink: /edna/prepare_edna_data/
author_profile: false
date: 2026-09-30 08:00:00 +0200
last_modified_at: 2026-09-30 08:00:00 +0200
---

The eDNA laboratory delivers its results as one master workbook. The workbook uses the laboratory's own sample names, packs 19 indicators under a three-row header, and lists ASVs by lineage in wide format. The conversion CLI turns it into the six files xspatula loads. It also recomputes every diversity index from the ASV tables and reports any mismatch with the laboratory's values.

Think of the CLI as a customs office: the lab's shipment is unpacked, every item is checked against the manifest, relabelled with xspatula names, and repacked in standard containers. If anything cannot be identified, the whole shipment is stopped rather than partly let through.

## The CLI

```
./ai4sh/import_data/prepare_ai4sh_data/edna_final_results_to_xspatula.py
```

Run it from any folder, in the project's Python environment:

```bash
cd ai4sh/import_data/prepare_ai4sh_data
python edna_final_results_to_xspatula.py
```

With no arguments it uses the default paths below, relative to the script's folder.

| Argument | Default | Description |
|---|---|---|
| `source` (positional) | `edna/excel/AI4SH_eDNA_Final results.xlsx` | The laboratory master workbook |
| `--out-dir` | `../eDNA/excel` (= `ai4sh/import_data/eDNA/excel/`) | Where the six xspatula Excel files are written. The loading jobs expect them here |
| `--report-dir` | `edna/report` | Where the check report is written |
| `--translator-dir` | `edna` | Folder with the three translator files |

The script's docstring mentions `../eDNA/source/` and `../eDNA/report` as defaults. That is out of date — the paths in the table above are the ones the code uses (`--help` prints them).

A run takes a couple of minutes.

## The master workbook — you must supply it

**Path**: `./ai4sh/import_data/prepare_ai4sh_data/edna/excel/AI4SH_eDNA_Final results.xlsx`

This file is **not in the repository**:
- The folders `edna/excel/`, `edna/report/` and `ai4sh/import_data/eDNA/excel/` are all gitignored.
- The data is unpublished, and the workbook's content and layout may change as the laboratory's bioinformatics software develops.

Get the current version from the AI4SH eDNA laboratory and place it at the path above, under exactly that file name, including the space. When the laboratory sends an update, overwrite the file and rerun the CLI.

The workbook must hold three sheets:

| Sheet | Layout |
|---|---|
| `Summary` | Three header rows: row 1 is the organism group (`Prokaryotes`/`Fungi`), row 2 the category (`Alpha diversity`/`Functional prediction`), row 3 the indicator. One row per sample. Column E (`Sample_Name`) holds the laboratory's sample name, the key used everywhere else. Indicators start in column F |
| `ASV_prokaryotes` | Header row `ID`, then one column per sample. One row per ASV, labelled by its lineage (e.g. `k__Bacteria\|p__…\|g__…\|s__`), with relative abundances as values. Rows labelled without `__` (e.g. `simpson_alpha`, `shannon_alpha`) are derived values; they are used as checks, not as ASVs |
| `ASV_fungi` | As `ASV_prokaryotes` |

If the laboratory renames a sheet, moves the name column or changes the number of header rows, the CLI fails. Adapt the constants at the top of the script (`SUMMARY_SHEET`, `SUMMARY_HEADER_ROWS`, `SUMMARY_NAME_COL`, `SUMMARY_FIRST_INDICATOR_COL`, `ASV_GROUPS`) to match.

## Support files — the three translators

The translators are small CSV files in `./ai4sh/import_data/prepare_ai4sh_data/edna/`, maintained by hand and tracked in the repository. They map the laboratory's names to xspatula names.

### eDNA_sample_name_translator.csv

One row per laboratory sample name. The laboratory's names (e.g. `DK3662C5`) are home-made and cannot be derived from the xspatula names.

| Column | Required | Description |
|---|---|---|
| `summary_name` | yes | Sample name exactly as in `Summary` column E. Each may occur only once |
| `sampling_log_id__sampling_log_name` | yes | The xspatula sampling log, e.g. `ai4sh_dk_foulum_2024` |
| `sample_name` | yes | The xspatula sample name **including depth**, e.g. `366-2-c5@0-20` |
| `observed_at` | yes, unless excluded | Date of the laboratory analysis as `YYYYMMDD` |
| `exclude` | no | A reason to leave the sample out; blank means include |

Use `exclude` rather than deleting a row, so the reason stays documented. It fits cases like a failed sample (e.g. an extreme outlier in richness and Simpson index) or a sample that is not registered in any sampling log. Excluded samples appear in the conversion report.

### eDNA_indicator_translator.csv

One row per indicator column in `Summary`. The first three columns match the three header rows; the fourth gives the xspatula column name, which is `@` + the indicator alias.

| summary_group | summary_category | summary_indicator | xspatula_column |
|---|---|---|---|
| Prokaryotes | Alpha diversity | observed richness | `@prokaryotes alpha richness` |
| Prokaryotes | Alpha diversity | chao1_estimated richness | `@prokaryotes alpha chao1 richness` |
| Prokaryotes | Alpha diversity | dominance | `@prokaryotes alpha dominance` |
| Prokaryotes | Alpha diversity | pielou_e | `@prokaryotes alpha pielou e` |
| Prokaryotes | Alpha diversity | shannon | `@prokaryotes alpha shannon` |
| Prokaryotes | Alpha diversity | simpson | `@prokaryotes alpha simpson` |
| Prokaryotes | Functional prediction | chemoheterotrophy | `@prokaryotes chemoheterotrophy` |
| Prokaryotes | Functional prediction | human_pathogens_all | `@prokaryotes human pathogens all` |
| Prokaryotes | Functional prediction | nitrogen_fixation | `@prokaryotes nitrogen fixation` |
| Fungi | Alpha diversity | observed richness … simpson | `@fungi alpha richness` … `@fungi alpha simpson` (6 rows) |
| Fungi | Functional prediction | Ectomycorrhizal fungi | `@ectomycorrhizal fungi function` |
| Fungi | Functional prediction | Arbuscular mycorrhizal fungi | `@arbuscular mycorrhizal fungi function` |
| Fungi | Functional prediction | Fungal saprotrophs | `@saprotrophic fungal function` |
| Fungi | Functional prediction | Fungal plant pathogens | `@plant pathogenic fungal function` |

Matching ignores case and treats `_` as a space. It also accepts the laboratory's misspelling "Funtional". Every row must match a `Summary` column, and every `Summary` column must match a row. If a new indicator appears, add it here **and** register it (indicator + provision_indicator) in the utility catalogues before loading.

### eDNA_observation_log_template.csv

Exactly **one** data row, holding the values shared by every eDNA observation log:

| Column | Example value |
|---|---|
| `provision_id__provision_name` | `ai4sh-metabarcoding` |
| `contact_name`, `contact_email` | The laboratory contact |
| `abstract` | blank |
| `preparation_id__preparation_name` | `none` |
| `preservation_id__preservation_name` | `dna/rna shield` |
| `storage_id__storage_name` | `cold` |
| `transportation_id__transportation_name` | `cold` |
| `laboratory` | `True` |

## When the CLI stops

The run aborts with exit code 1 and **writes no files** if:

- a `Summary` sample is missing from the sample translator, or a `summary_name` occurs twice;
- a `Summary` indicator header is not in the indicator translator, or a translator row has no matching `Summary` column;
- a sample column in an ASV sheet is not in `Summary`;
- a sample that is not excluded has no `observed_at`;
- the observation log template does not have exactly one data row.

Mismatches between the laboratory's indices and the recomputed ones are **reported, not fatal**.

## Output

Six Excel files (sheet `Sheet1`) are written to `./ai4sh/import_data/eDNA/excel/`:

| File | Content |
|---|---|
| `AI4SH_observation_log_eDNA.xlsx` | One observation log per sampling log: `sampling_log_id__sampling_log_name`, the template columns, and `name` = `auto` (resolves to `<sampling log>@ai4sh-metabarcoding`) |
| `AI4SH_observation_eDNA.xlsx` | One row per included sample: `observation_log_id__observation_log_name`, `sample_id__sample_name`, `observed_at`, and the 19 `@indicator` columns |
| `taxa_prokaryotes_singlecolumn.xlsx` | Column `Taxa`: one lineage per prokaryote ASV row |
| `taxa_fungi_singlecolumn.xlsx` | As above for fungi |
| `AI4SH_asv_prokaryotes_eDNA.xlsx` | Long format, non-zero values only: `observation_log_id__observation_log_name`, `sample_id__sample_name`, `asv`, `taxa`, `rel_abundance`, `read_count` |
| `AI4SH_asv_fungi_eDNA.xlsx` | As above for fungi |

The `asv` key is `<group>-<row number>` (e.g. `fungi-000001`), because the workbook has no ASV ids or sequences. `read_count` is `rel_abundance × rarefaction depth`. It is filled only when every abundance of the sample is an exact multiple of 1/depth, which confirms the sample was rarefied as stated.

## The check report

Two files are written to `edna/report/`:

- **`edna_conversion_report.md`** — read this after every run. It lists row counts per output file, excluded samples with their reasons, samples present in `Summary` but missing from an ASV sheet, the detected rarefaction depth, and which samples differ in which recomputed index.
- **`edna_index_checks.csv`** — one row per sample and check (`group`, `summary_name`, `check`, `reported`, `computed`, `status`). Checks cover column sum, rarefaction depth, observed richness, Chao1, dominance, Pielou evenness, Shannon (log2) and Simpson, and the derived rows of the ASV sheets.

Functional predictions (FAPROTAX, FUNGuild) cannot be recomputed from the delivered data and are not checked.

A sample flagged with *no exact rarefaction depth* is suspect: its abundances do not add up to whole reads at 20,000. Consider excluding it in the sample translator and asking the laboratory.

## Next step

Proceed to [Insert eDNA observations].

[Insert eDNA observations]: /edna/insert_edna_observation/
