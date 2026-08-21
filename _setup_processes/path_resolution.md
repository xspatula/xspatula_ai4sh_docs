---
title: "Path Resolution"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Every relative path in a scheme, job, pilot, or process file resolves against the project root — one anchor, one mental model, as of 2026-08-15. This page states the rule and the two exceptions."
permalink: /setup_process/path_resolution/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

Every JSON file in this framework — scheme, job/project, pilot, and individual process files — can reference other files or directories by relative path. Before 2026-08-15, exactly what a given relative path was anchored to depended on which field you were looking at, and sometimes on how deeply a file happened to be nested. That's been harmonised into a single rule.

## The rule

> Every relative path anywhere in the scheme → job(project) → pilot → process chain is relative to the **project root** — the directory `project_path` (in the scheme file) resolves to. This includes `job_folder`, `process_sub_folder`, `pilot_file`, every entry in a pilot list/file, and — as of 2026-08-15 — a process file's own in-parameter paths (`tabular_data_path`, `dst_path` on a `translate_tabular_data` process).

One anchor, one mental model: you never need to count `../` levels based on how deeply a file happens to be nested, and moving a process file to a different folder never silently breaks its own internal paths.

## The two exceptions

Both are, deliberately, upstream of the project root itself — they're what establishes it in the first place, so they can't be relative to it:

- **`scheme_file`** — the path you hand to `Initiate_process`/`Initiate_database` — is resolved relative to the **calling notebook's working directory**.
- **`project_path`** (inside the scheme file) is resolved relative to the **scheme file's own directory**, since that's what establishes the project root.

## One asymmetry that looks like a bug but isn't

A job file's `pilot_file` resolves against `job_folder` alone, while the JSON paths *inside* that pilot file resolve against `job_folder` + `process_sub_folder`. This is deliberate, and consistent everywhere it's used: the pilot `.txt`/list file lives at the `job_folder` level, one directory above the `process_sub_folder` that the individual process JSON files it lists actually live in. True for every collection on this site — `import_data/dataset/`, `user_management/organisation/`, `setup_db/`, `setup_processes/`.

## The five path notations

All notation-aware resolution goes through `Get_project_path`/`Full_path_locate` in `src/lib/pilot.py`. It recognises five styles, all anchored per the rule above:

| Notation | Behaviour |
|---|---|
| `../` | Parent-relative, supports multiple levels (`../../config.json`) |
| `.` or `./` | `.` alone means "the anchor itself"; `./subpath` appends `subpath` to it |
| `~/` | Resolved against the user's home directory |
| `/` | Absolute — returned unchanged |
| bare (no recognised prefix) | Joined onto the anchor directly |

## Worked example

Given a project root at `~/GitHub_xspatula/xspatula_ai4sh/ai4sh/`, a `tabular_data_path` of:

```
import_data/dataset/excel/data_source.xlsx
```

resolves to `~/GitHub_xspatula/xspatula_ai4sh/ai4sh/import_data/dataset/excel/data_source.xlsx` — regardless of which directory the process JSON file that names it happens to live in. This is the form used throughout the `translate_*` pages on this site; see [Translate Processes][setup_process_translate] for the `tabular_data_path`/`dst_path` parameters themselves.

[setup_process_translate]: /setup_process/translate/
