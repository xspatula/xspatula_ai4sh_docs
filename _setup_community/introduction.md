---
title: "Setup Community"
layout: single
sidebar:
  nav: "setup_community"
excerpt: "Registering organisations and users used to mean hand-editing a JSON seed file and rerunning setup_db.ipynb. This collection covers the Excel-intake workflow that replaces that for anyone who isn't the database superuser."
permalink: /setup_community/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

Without this stage, the framework has a working database and registered processes, but only one practical way to add a `community.organisation` or `community.user` row: hand-edit a JSON seed file and rerun `setup_db.ipynb` as the database superuser. That doesn't scale beyond the person running setup — it can't be how a workshop participant, or any ordinary contributor, gets an account.

This collection covers the workshop-scale replacement: organisations and users fill in an Excel spreadsheet, an admin reviews and curates it, and a notebook translates and inserts the accepted rows through the normal generic Excel-intake pipeline already used elsewhere in this site for [dataset metadata][dataset_meta] and other data types, reused here as-is.

## Prerequisite

`setup_db.ipynb` and `setup_processes.ipynb` must already have been run — including the community process registration described in [Community Processes][setup_process_community] — before anything in this collection works. The registration notebook covered here logs in as a real `community.user` to perform its inserts, so a working login (see [Bootstrap user][setup_community_bootstrap_user]) has to exist first.

## Pages in this collection

1. [Bootstrap user][setup_community_bootstrap_user] — the chicken-and-egg problem: the very first user can't be created through the Excel flow, since that flow requires an already-logged-in user
2. [SMTP email][setup_community_smtp_email] — the outgoing-email setup new users' passwords are delivered through
3. [Excel intake][setup_community_excel_intake] — the two spreadsheets, `organisation.xlsx` and `user.xlsx`, and their columns
4. [Register notebook][setup_community_register_notebook] — the admin's review-and-run workflow
5. [Welcome email][setup_community_welcome_email] — what a newly registered user actually receives

[dataset_meta]: /dataset_meta/
[setup_process_community]: /setup_process/community/
[setup_community_bootstrap_user]: /setup_community/bootstrap_user/
[setup_community_smtp_email]: /setup_community/smtp_email/
[setup_community_excel_intake]: /setup_community/excel_intake/
[setup_community_register_notebook]: /setup_community/register_notebook/
[setup_community_welcome_email]: /setup_community/welcome_email/
