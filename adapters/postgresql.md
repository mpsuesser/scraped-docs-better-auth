---
url: https://better-auth.com/llms.txt/docs/adapters/postgresql
title: "Postgresql"
description: ""
access_date: 2026-09-14T18:07:43.919Z
current_date: 2026-09-14T18:07:43.919Z
---

Integrate Better Auth with PostgreSQL.

PostgreSQL is a powerful, open-source relational database management system known for its advanced features, extensibility, and support for complex queries and large datasets. Read more about [PostgreSQL](https://www.postgresql.org/).

## Example Usage

Make sure you have PostgreSQL installed and configured. Then, you can connect it straight into Better Auth.

```
import { betterAuth } from "better-auth";
import { Pool } from "pg";

export const auth = betterAuth({
  database: new Pool({
    connectionString: "postgres://user:password@localhost:5432/database",
  }),
});
```

## Schema generation & migration

The [Better Auth CLI](https://better-auth.com/docs/concepts/cli) allows you to generate or migrate your database schema based on your Better Auth configuration and plugins.

| PostgreSQL Schema Generation | PostgreSQL Schema Migration |
| --- | --- |
| ✅ Supported | ✅ Supported |

#### migrate

#### npm

#### generate

```
npx auth@latest migrate
```

#### pnpm

#### yarn

#### bun

## Joins

Database joins are useful when Better-Auth needs to fetch related data from multiple tables in a single query. Endpoints like `/get-session`, `/get-full-organization` and many others benefit greatly from this feature, seeing upwards of 2x to 3x performance improvements depending on database latency.

The Kysely PostgreSQL dialect supports joins out of the box since version `1.4.0`. To enable this feature, set `advanced.database.joins` to `true` in your auth configuration.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  advanced: {
    database: {
      joins: true,
    },
  },
});
```

## Use a non-default schema

PostgreSQL uses the `public` schema by default. You can select another schema explicitly with `database.schemaName` or configure PostgreSQL's `search_path`.

When both are configured, Better Auth uses `database.schemaName`. Other unqualified queries continue to use the connection's `search_path`.

### Set database.schemaName

#### Kysely dialect

```
import { betterAuth } from "better-auth";
import { PostgresDialect } from "kysely";
import { Pool } from "pg";

export const auth = betterAuth({
  database: {
    dialect: new PostgresDialect({
      pool: new Pool({
        connectionString: "postgres://user:password@localhost:5432/database",
      }),
    }),
    type: "postgres",
    schemaName: "auth", 
  },
});
```

#### Kysely instance

The schema applies to runtime queries and CLI migrations. `npx auth@latest migrate` creates it when needed and ignores same-named tables in other schemas.

`npx auth@latest generate` starts the generated migration by creating the schema:

```
create schema if not exists "auth";
```

All subsequent statements use schema-qualified table names, such as `"auth"."user"`.

The PostgreSQL role used by Better Auth must be able to create the schema and its tables. Otherwise, create the schema and grant access to that role before running `npx auth@latest migrate`.

### Set search\_path

Use PostgreSQL's `search_path` instead when passing a `pg.Pool` directly, or when every unqualified query on the connection should use the same schema.

When using `search_path`, the Better Auth CLI expects the schema to already exist. Create it and grant access to the connection role before running `npx auth@latest migrate`.

#### Pool options

```
import { betterAuth } from "better-auth";
import { Pool } from "pg";

export const auth = betterAuth({
  database: new Pool({
    connectionString: "postgres://user:password@localhost:5432/database",
    options: "-c search_path=auth",
  }),
});
```

#### Connection string

To make the schema the default when a PostgreSQL role connects to a specific database:

```
ALTER ROLE your_role IN DATABASE your_database
SET search_path TO auth;
```

Reconnect after changing this default. Run `SHOW search_path` to verify the active value.

## Additional Information

PostgreSQL is supported under the hood via the [Kysely](https://kysely.dev/) adapter, any database supported by Kysely would also be supported. ([Read more here](https://better-auth.com/docs/adapters/other-relational-databases))

If you're looking for performance improvements or tips, take a look at our guide to [performance optimizations](https://better-auth.com/docs/guides/optimizing-for-performance).
