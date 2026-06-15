---
title: "Community Processes"
layout: single
sidebar:
  nav: "setup_processes"
excerpt: "Community schema processes handle organisations and users. In the AI4SH setup, community management uses the default framework processes defined during the database setup stage rather than separate process registration files."
permalink: /setup_process/community/
author_profile: false
date: 2026-03-31 08:00:00 +0200
last_modified_at: 2026-03-31 08:00:00 +0200
---

The `community` schema manages organisations and users. Community management in the AI4SH database relies on the default framework processes that are registered by the `process/processes_records_v10_sql.json` file during the [database setup stage][setup_db_process]. No separate community process registration files are included in the AI4SH `setup_processes` workflow.

## What the framework provides by default

After running `processes_records_v10_sql.json`, the framework has built-in processes for:

- Adding organisations (`community.organisation`)
- Adding user categories (`community.user_categories`)
- Adding users (`community.user`)
- Updating user details
- Managing user activity

These bootstrap processes are part of the core framework and do not need to be registered again for AI4SH.

## Managing community data after setup

Once the database is running, community records (organisations and users) are managed through scheme files and process files referencing the default community processes. To add a new user:

1. Create a process file calling `manage_user` (or the equivalent community process)
2. Supply the required parameters: `organisation_id`, `user_name`, `password`, `stratum_code`, and any optional personal details
3. Run the process via the appropriate notebook

The `stratum_code` determines which `.env` credentials the user logs in with and thus which database privileges they have. The 8 default strata range from 0 (most restricted) to 5 (most permissive), plus `community_admin` and `login_evaluation` for administrative functions.

## Cross-schema relationship

Community records are referenced by:

- `observation.data_source` — simplified passive data supplier (not a full framework user)
- `observation.person` — non-registered individual attributed to data (GDPR consideration)

These two tables in the `observation` schema are intentionally decoupled from the full `community.user` table, allowing attribution of data to people who are not system users.


[setup_db_process]: /setup_db/process/
