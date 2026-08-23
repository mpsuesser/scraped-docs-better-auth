---
url: https://better-auth.com/llms.txt/docs/adapters/prisma
title: "Prisma"
description: ""
access_date: 2026-08-23T16:07:07.064Z
current_date: 2026-08-23T16:07:07.064Z
---

Integrate Better Auth with Prisma.

Prisma ORM is an open-source database toolkit that simplifies database access and management in applications by providing a type-safe query builder and an intuitive data modeling interface.

For an introduction to Prisma ORM, see [Prisma getting started](https://www.prisma.io/docs/getting-started).

## Installation

To use the Prisma adapter, install `@better-auth/prisma-adapter`:

#### npm

```
npm install @better-auth/prisma-adapter
```

#### pnpm

#### yarn

#### bun

The example below uses Prisma 7 and PostgreSQL. If Prisma is not already configured, follow [Prisma's PostgreSQL quickstart](https://www.prisma.io/docs/prisma-orm/quickstart/postgresql) before continuing.

## Setup

The examples below use the following project structure:

.env

prisma.config.ts

schema.prisma

client.ts

prisma.ts

auth.ts

### Configure Prisma

If you are starting a new Prisma project, initialize it with PostgreSQL and an explicit Prisma Client output path:

#### npm

```
npx prisma init --datasource-provider postgresql --output ../src/generated/prisma
```

#### pnpm

#### yarn

#### bun

This creates `prisma/schema.prisma`, `prisma.config.ts`, and `.env`. Set `DATABASE_URL` in `.env` to your PostgreSQL connection string.

If Prisma is already configured in your project, keep your existing datasource and output path and skip this initialization command.

Generate Prisma Client after configuring its output path:

#### npm

```
npx prisma generate
```

#### pnpm

#### yarn

#### bun

### Create the Prisma client

Import `PrismaClient` from the output path configured in your Prisma schema and pass the PostgreSQL driver adapter to it:

```
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "../generated/prisma/client";

const databaseUrl = process.env.DATABASE_URL;

if (!databaseUrl) {
  throw new Error("DATABASE_URL is not set");
}

const adapter = new PrismaPg({
  connectionString: databaseUrl,
});

export const prisma = new PrismaClient({ adapter });
```

Create one `PrismaClient` instance and reuse it across your application. Frameworks with hot reloading or serverless runtimes may require a framework-specific lifecycle pattern.

### Configure Better Auth

Pass the Prisma Client instance to the Better Auth Prisma adapter:

```
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { prisma } from "./prisma";

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "postgresql",
  }),
});
```

## Schema generation & migration

The [Better Auth CLI](https://better-auth.com/docs/concepts/cli) generates the Prisma schema required by your Better Auth configuration and plugins. Use the Prisma CLI to create and apply a migration from the generated schema.

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

The Better Auth CLI updates your Prisma schema but does not apply the migration. Use Prisma to create the database migration, then regenerate Prisma Client:

```
npx prisma migrate dev --name add-better-auth
npx prisma generate
```

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
- [How to use Prisma ORM with Better Auth and Next.js](https://www.prisma.io/docs/guides/authentication/better-auth/nextjs)
- [How to use Prisma ORM with Better Auth and Astro](https://www.prisma.io/docs/guides/authentication/better-auth/astro)
