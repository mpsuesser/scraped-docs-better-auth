---
url: https://better-auth.com/llms.txt/docs/adapters/postgresql
title: "Postgresql"
description: ""
access_date: 2026-08-18T00:08:46.984Z
current_date: 2026-08-18T00:08:46.984Z
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

In most cases, the default schema is `public`. To have Better Auth use a non-default schema (e.g., `auth`) for its tables, you have several options:

### Option 1: Set search\_path in connection string (Recommended)

Append the `options` parameter to your connection URI:

```
import { betterAuth } from "better-auth";
import { Pool } from "pg";

export const auth = betterAuth({
  database: new Pool({
    connectionString: "postgres://user:password@localhost:5432/database?options=-c search_path=auth",
  }),
});
```

URL-encode if needed: `?options=-c%20search_path%3Dauth`.

### Option 2: Set search\_path using Pool options

```
import { betterAuth } from "better-auth";
import { Pool } from "pg";

export const auth = betterAuth({
  database: new Pool({
    host: "localhost",
    port: 5432,
    user: "postgres",
    password: "password",
    database: "my-db",
    options: "-c search_path=auth",
  }),
});
```

### Option 3: Set default schema for database user

Set the PostgreSQL user's default schema:

```
ALTER USER your_user SET search_path TO auth;
```

After setting this, reconnect to apply the changes.

### Prerequisites

Before using a non-default schema, ensure:

1. **The schema exists:**
	```
	CREATE SCHEMA IF NOT EXISTS auth;
	```
2. **The user has appropriate permissions:**
	```
	GRANT ALL PRIVILEGES ON SCHEMA auth TO your_user;
	GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA auth TO your_user;
	ALTER DEFAULT PRIVILEGES IN SCHEMA auth GRANT ALL ON TABLES TO your_user;
	```

### How it works

The Better Auth CLI migration system automatically detects your configured `search_path`:

- When running `npx auth migrate`, it inspects only the tables in your configured schema
- Tables in other schemas (e.g., `public`) are ignored, preventing conflicts
- All new tables are created in your specified schema

### Troubleshooting

## Additional Information

PostgreSQL is supported under the hood via the [Kysely](https://kysely.dev/) adapter, any database supported by Kysely would also be supported. ([Read more here](https://better-auth.com/docs/adapters/other-relational-databases))

If you're looking for performance improvements or tips, take a look at our guide to [performance optimizations](https://better-auth.com/docs/guides/optimizing-for-performance).
