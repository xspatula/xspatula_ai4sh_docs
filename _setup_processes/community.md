---
title: "Community Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Community schema processes handle organisations and users. A dedicated registration step now registers manage_organisation and manage_user, on top of the default framework processes defined during database setup."
permalink: /setup_process/community/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The `community` schema manages organisations and users. Some of this relies on the default framework processes that are registered by the `process/processes_records_v10_sql.json` file during the [database setup stage][setup_db_process] — but as of 2026-08-14/15 there is also a real, AI4SH-specific registration step for the two everyday community tables:

- `community/organisation_v10_sql.json` — registers `manage_organisation` against `community.organisation`
- `community/user_v10_sql.json` — registers `manage_user` against `community.user`

Both follow the same `add_process` pattern documented for `manage_person` in [Observation Processes][setup_process_observation], and are wired into the pilot list `ai4sh_setup_processes.txt` under a `### COMMUNITY ###` section, right after `utility/territory_v10_sql.json` and before `### OBSERVATION UTILITIES ###`.

## What the framework provides by default

After running `processes_records_v10_sql.json`, the framework has built-in processes for:

- Adding user categories (`community.user_categories`)
- Updating user details
- Managing user activity

These bootstrap processes are part of the core framework and do not need to be registered again for AI4SH. Organisations and users themselves now go through the AI4SH-specific `manage_organisation`/`manage_user` processes above instead.

## Managing community data after setup

This page covers process *registration* only — for the actual day-to-day workflow of adding organisations and users (the Excel spreadsheets, the admin review step, password provisioning and email), see the [Setup community][setup_community] collection. In short: participants fill in a spreadsheet, an admin reviews and curates it, then a notebook translates and runs it through the `manage_organisation`/`manage_user` processes registered here.

The `stratum_code` determines which `.env` credentials a user logs in with and thus which database privileges they have. The 8 default strata range from 0 (most restricted) to 5 (most permissive), plus `community_admin` and `login_evaluation` for administrative functions.

## Cross-schema relationship

Community records are referenced by:

- `observation.data_source` — simplified passive data supplier (not a full framework user)
- `observation.person` — non-registered individual attributed to data (GDPR consideration)

These two tables in the `observation` schema are intentionally decoupled from the full `community.user` table, allowing attribution of data to people who are not system users.


[setup_db_process]: /setup_db/process/
[setup_process_observation]: /setup_process/observation/
[setup_community]: /setup_community/
