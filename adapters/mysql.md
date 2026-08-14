---
url: https://better-auth.com/llms.txt/docs/adapters/mysql
title: "Mysql"
description: ""
access_date: 2026-08-14T12:20:50.524Z
current_date: 2026-08-14T12:20:50.524Z
---

Integrate Better Auth with MySQL.

MySQL is a popular open-source relational database management system (RDBMS) that is widely used for building web applications and other types of software. It provides a flexible and scalable database solution that allows for efficient storage and retrieval of data. Read more here: [MySQL](https://www.mysql.com/).

## Example Usage

Make sure you have MySQL installed and configured. Then, you can connect it straight into Better Auth.

```
import { betterAuth } from "better-auth";
import { createPool } from "mysql2/promise";

export const auth = betterAuth({
  database: createPool({
    host: "localhost",
    user: "root",
    password: "password",
    database: "database",
    timezone: "Z", // Important to ensure consistent timezone values
  }),
});
```

## Schema generation & migration

The [Better Auth CLI](https://better-auth.com/docs/concepts/cli) allows you to generate or migrate your database schema based on your Better Auth configuration and plugins.

| MySQL Schema Generation | MySQL Schema Migration |
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

The Kysely MySQL dialect supports joins out of the box since version `1.4.0`.

To enable this feature, you need to set the `experimental.joins` option to `true` in your auth configuration.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  experimental: { joins: true }
});
```

## Additional Information

MySQL is supported under the hood via the [Kysely](https://kysely.dev/) adapter, any database supported by Kysely would also be supported. ([Read more here](https://better-auth.com/docs/adapters/other-relational-databases))

If you're looking for performance improvements or tips, take a look at our guide to [performance optimizations](https://better-auth.com/docs/guides/optimizing-for-performance).
