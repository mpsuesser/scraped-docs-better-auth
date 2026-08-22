---
url: https://better-auth.com/llms.txt/docs/adapters/prisma
title: "Prisma"
description: ""
access_date: 2026-08-22T14:33:01.102Z
current_date: 2026-08-22T14:33:01.102Z
---

Integrate Better Auth with Prisma.

Prisma ORM is an open-source database toolkit that simplifies database access and management in applications by providing a type-safe query builder and an intuitive data modeling interface.

Before getting started, make sure you have Prisma installed and configured. For more information, see [Prisma Documentation](https://www.prisma.io/docs/)

## Installation

To use the Prisma adapter, you need to install the `@better-auth/prisma-adapter` package:

#### npm

```
npm install @better-auth/prisma-adapter
```

#### pnpm

#### yarn

#### bun

Prisma 7 with SQLite requires the SQLite driver adapter:

#### npm

```
npm install @prisma/adapter-better-sqlite3
```

#### pnpm

#### yarn

#### bun

## Example Usage

You can use the Prisma adapter to connect to your database as follows.

```
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { PrismaClient } from "@prisma/client";
import { PrismaBetterSqlite3 } from "@prisma/adapter-better-sqlite3";

const databaseUrl = process.env.DATABASE_URL;

if (!databaseUrl) {
  throw new Error("DATABASE_URL is not set");
}

const adapter = new PrismaBetterSqlite3({
  url: databaseUrl,
});
const prisma = new PrismaClient({ adapter });

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "sqlite",
  }),
});
```

## Schema generation & migration

The [Better Auth CLI](https://better-auth.com/docs/concepts/cli) allows you to generate or migrate your database schema based on your Better Auth configuration and plugins.

| Prisma Schema Generation | Prisma Schema Migration |
| --- | --- |
| ✅ Supported | ❌ Not Supported |

#### npm

```
npx auth@latest generate
```

#### pnpm

#### yarn

#### bun

## Joins

Database joins are useful when Better-Auth needs to fetch related data from multiple tables in a single query. Endpoints like `/get-session`, `/get-full-organization` and many others benefit greatly from this feature, seeing upwards of 2x to 3x performance improvements depending on database latency.

The Prisma adapter supports joins out of the box since version `1.4.0`. To enable this feature, set `advanced.database.joins` to `true` in your auth configuration.

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

## Additional Information

- If you're looking for performance improvements or tips, take a look at our guide to [performance optimizations](https://better-auth.com/docs/guides/optimizing-for-performance).
- [How to use Prisma ORM with Better Auth and Next.js](https://www.prisma.io/docs/guides/betterauth-nextjs)
