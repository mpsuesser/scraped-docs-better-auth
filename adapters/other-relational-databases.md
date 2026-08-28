---
url: https://better-auth.com/llms.txt/docs/adapters/other-relational-databases
title: "Other Relational Databases"
description: ""
access_date: 2026-08-28T22:43:46.051Z
current_date: 2026-08-28T22:43:46.051Z
---

Integrate Better Auth with other relational databases.

Better Auth supports a wide range of database dialects out of the box thanks to [Kysely](https://kysely.dev/).

Any dialect supported by Kysely can be utilized with Better Auth, including capabilities for generating and migrating database schemas through the [CLI](https://better-auth.com/docs/concepts/cli).

When using the CLI with a custom dialect, set `database.type` alongside `database.dialect` so migrations emit SQL for the correct database.

## Core Dialects

- [MySQL](https://better-auth.com/docs/adapters/mysql)
- [SQLite](https://better-auth.com/docs/adapters/sqlite)
- [PostgreSQL](https://better-auth.com/docs/adapters/postgresql)
- [MS SQL](https://better-auth.com/docs/adapters/mssql)

## Kysely Organization Dialects

- [Postgres.js](https://github.com/kysely-org/kysely-postgres-js)
- [SingleStore Data API](https://github.com/kysely-org/kysely-singlestore)
- [Supabase](https://github.com/kysely-org/kysely-supabase)
