---
title: "Bootstrap User"
layout: single
sidebar:
  nav: "setup_community"
excerpt: "The very first community.user can't be created through the Excel-intake workflow, because that workflow requires an already-logged-in user. It still has to be hand-seeded — here's how, now that passwords are stored as bcrypt hashes."
permalink: /setup_community/bootstrap_user/
author_profile: false
date: 2026-08-15 08:00:00 +0200
last_modified_at: 2026-08-15 08:00:00 +0200
---

## The chicken-and-egg problem

Registering a new user through [Excel intake][setup_community_excel_intake] and [Register notebook][setup_community_register_notebook] runs through a registered process, `manage_user` — and running any process requires an already-logged-in `community.user`. That's fine for the second user onwards, but the *first* user can't log in before they exist. Someone has to be seeded directly.

That first-user seed still happens the old way: hand-editing
`setup/zzz/ai4sh/setup_db/json_ai4sh/community/user_records_v10_sql.json` and running `setup_db.ipynb`.

## Passwords are bcrypt hashes now, not plaintext

`community.user.password` used to be compared as plaintext (`WHERE password = %s`). It now stores a bcrypt hash, verified in Python after the row is fetched. That's safer, but it also means there's no way to just type a password directly into `user_records_v10_sql.json` any more — only a hash belongs in that column.

## Hashing a password by hand: setup/hash_password.py

A small standalone CLI wraps the same `Hash_password` function used everywhere else in the framework, so you can generate a hash outside the database and paste it in. Open a terminal and run the command:

```bash
python3 setup/hash_password.py
```

This prompts for the password twice, with hidden input, and prints a bcrypt hash:

```
Password to hash:
Repeat password:
$2b$12$KIx9m3v0eR5t8...
```

You can also pass the password as an argument (`python3 setup/hash_password.py 'some-password'`), but avoid that on a shared machine — the plaintext password then briefly appears in shell history and the process list.

## Applying the hash

Two ways to get the hash into the database, depending on where you are in the setup lifecycle:

- **Fresh rebuild** — paste the hash into the `password` value inside `user_records_v10_sql.json`, then run `delete_db.ipynb` followed by `setup_db.ipynb`.
- **Live database, no rebuild** — run directly against the database:
  ```sql
  UPDATE community.user SET password = '<hash>' WHERE user_name = '...';
  ```

## Scope

This is a manual, admin-only escape hatch for bootstrapping — not something wired into the Excel intake, and not something an ordinary user ever touches. The Excel-based registration flow deliberately never sees or sets a plaintext password at all; every subsequent user gets one generated and emailed automatically, covered in [Welcome email][setup_community_welcome_email].

[setup_community_excel_intake]: /setup_community/excel_intake/
[setup_community_register_notebook]: /setup_community/register_notebook/
[setup_community_welcome_email]: /setup_community/welcome_email/
