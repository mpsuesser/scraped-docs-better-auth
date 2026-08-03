---
url: https://better-auth.com/llms.txt/docs/adapters/mssql
title: "Mssql"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Integrate Better Auth with MS SQL.

Microsoft SQL Server is a relational database management system developed by Microsoft, designed for enterprise-level data storage, management, and analytics with robust security and scalability features. Read more about [Microsoft SQL Server](https://en.wikipedia.org/wiki/Microsoft_SQL_Server).

## Example Usage

Make sure you have MS SQL installed and configured. Then, you can connect it straight into Better Auth.

```
import { betterAuth } from "better-auth";
import { MssqlDialect } from "kysely";
import * as Tedious from 'tedious'
import * as Tarn from 'tarn'

const dialect = new MssqlDialect({
  tarn: {
    ...Tarn,
    options: {
      min: 0,
      max: 10,
    },
  },
  tedious: {
    ...Tedious,
    connectionFactory: () => new Tedious.Connection({
      authentication: {
        options: {
          password: 'password',
          userName: 'username',
        },
        type: 'default',
      },
      options: {
        database: 'some_db',
        port: 1433,
        trustServerCertificate: true,
      },
      server: 'localhost',
    }),
  },
  TYPES: {
        ...Tedious.TYPES,
        DateTime: Tedious.TYPES.DateTime2,
    },
})

export const auth = betterAuth({
  database: {
    dialect,
    type: "mssql"
  }
});
```

## Schema generation & migration

The [Better Auth CLI](https://better-auth.com/docs/concepts/cli) allows you to generate or migrate your database schema based on your Better Auth configuration and plugins.

| MS SQL Schema Generation | MS SQL Schema Migration |
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

Database joins is useful when Better-Auth needs to fetch related data from multiple tables in a single query. Endpoints like `/get-session`, `/get-full-organization` and many others benefit greatly from this feature, seeing upwards of 2x to 3x performance improvements depending on database latency.

The Kysely MS SQL dialect supports joins out of the box since version `1.4.0`. To enable this feature, you need to set the `experimental.joins` option to `true` in your auth configuration.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  experimental: { joins: true }
});
```

## Additional Information

MS SQL is supported under the hood via the [Kysely](https://kysely.dev/) adapter, any database supported by Kysely would also be supported. ([Read more here](https://better-auth.com/docs/adapters/other-relational-databases))

If you're looking for performance improvements or tips, take a look at our guide to [performance optimizations](https://better-auth.com/docs/guides/optimizing-for-performance).
