---
url: https://better-auth.com/llms.txt/docs/adapters/sqlite
title: "Sqlite"
description: ""
access_date: 2026-08-14T12:20:50.524Z
current_date: 2026-08-14T12:20:50.524Z
---

Integrate Better Auth with SQLite.

SQLite is a lightweight, serverless, self-contained SQL database engine that is widely used for local data storage in applications. Read more [here.](https://www.sqlite.org/)

## Example Usage

Better Auth supports multiple SQLite drivers. Choose the one that best fits your environment:

### Better-SQLite3 (Recommended)

The most popular and stable SQLite driver for Node.js:

```
import { betterAuth } from "better-auth";
import Database from "better-sqlite3";

export const auth = betterAuth({
  database: new Database("database.sqlite"),
});
```

### Node.js Built-in SQLite (Release Candidate)

Starting from Node.js 22.5.0, you can use the built-in [SQLite](https://nodejs.org/api/sqlite.html) module:

```
import { betterAuth } from "better-auth";
import { DatabaseSync } from "node:sqlite";

export const auth = betterAuth({
  database: new DatabaseSync("database.sqlite"),
});
```

To run your application with Node.js SQLite:

```
node your-app.js
```

### Bun Built-in SQLite

You can also use the built-in [SQLite](https://bun.com/docs/api/sqlite) module in Bun, which is similar to the Node.js version:

```
import { betterAuth } from "better-auth";
import { Database } from "bun:sqlite";

export const auth = betterAuth({
  database: new Database("database.sqlite"),
});
```

## Schema generation & migration

The [Better Auth CLI](https://better-auth.com/docs/concepts/cli) allows you to generate or migrate your database schema based on your Better Auth configuration and plugins.

| SQLite Schema Generation | SQLite Schema Migration |
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

## Joins (Experimental)

Database joins are useful when Better-Auth needs to fetch related data from multiple tables in a single query. Endpoints like `/get-session`, `/get-full-organization` and many others benefit greatly from this feature, seeing upwards of 2x to 3x performance improvements depending on database latency.

The Kysely SQLite dialect supports joins out of the box since version `1.4.0`. To enable this feature, you need to set the `experimental.joins` option to `true` in your auth configuration.

```
export const auth = betterAuth({
  experimental: { joins: true }
});
```

## Additional Information

SQLite is supported under the hood via the [Kysely](https://kysely.dev/) adapter, any database supported by Kysely would also be supported. ([Read more here](https://better-auth.com/docs/adapters/other-relational-databases))

If you're looking for performance improvements or tips, take a look at our guide to [performance optimizations](https://better-auth.com/docs/guides/optimizing-for-performance).
