---
url: https://better-auth.com/llms.txt/docs/adapters/prisma
title: "Prisma"
description: ""
access_date: 2026-08-03T18:54:22.481Z
current_date: 2026-08-03T18:54:22.481Z
---

# Prisma

Integrate Better Auth with Prisma.



Prisma ORM is an open-source database toolkit that simplifies database access and management in applications by providing a type-safe query builder and an intuitive data modeling interface.

Before getting started, make sure you have Prisma installed and configured. For more information, see [Prisma Documentation](https://www.prisma.io/docs/)

Installation [#installation]

To use the Prisma adapter, you need to install the `@better-auth/prisma-adapter` package:

<CodeBlockTabs defaultValue="npm" groupId="persist-install" persist>
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="npm">
      npm
    </CodeBlockTabsTrigger>

    <CodeBlockTabsTrigger value="pnpm">
      pnpm
    </CodeBlockTabsTrigger>

    <CodeBlockTabsTrigger value="yarn">
      yarn
    </CodeBlockTabsTrigger>

    <CodeBlockTabsTrigger value="bun">
      bun
    </CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="npm">
    ```bash
    npm install @better-auth/prisma-adapter
    ```
  </CodeBlockTab>

  <CodeBlockTab value="pnpm">
    ```bash
    pnpm add @better-auth/prisma-adapter
    ```
  </CodeBlockTab>

  <CodeBlockTab value="yarn">
    ```bash
    yarn add @better-auth/prisma-adapter
    ```
  </CodeBlockTab>

  <CodeBlockTab value="bun">
    ```bash
    bun add @better-auth/prisma-adapter
    ```
  </CodeBlockTab>
</CodeBlockTabs>

Example Usage [#example-usage]

You can use the Prisma adapter to connect to your database as follows.

```ts title="auth.ts"
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "sqlite",
  }),
});
```

<Callout type="warning">
  Starting from Prisma 7, the `output` path field is required. If you have configured a custom output path in your `schema.prisma` file (e.g., `output = "../src/generated/prisma"`), make sure to [import the Prisma client from that location](https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/generating-prisma-client#the-location-of-prisma-client) instead of `@prisma/client`.
</Callout>

Schema generation & migration [#schema-generation--migration]

The [Better Auth CLI](/docs/concepts/cli) allows you to generate or migrate
your database schema based on your Better Auth configuration and plugins.

<table>
  <thead>
    <tr className="border-b">
      <th>
        <p className="font-bold text-[16px] mb-1">
          Prisma Schema Generation
        </p>
      </th>

      <th>
        <p className="font-bold text-[16px] mb-1">
          Prisma Schema Migration
        </p>
      </th>
    </tr>
  </thead>

  <tbody>
    <tr className="h-10">
      <td>
        ✅ Supported
      </td>

      <td>
        ❌ Not Supported
      </td>
    </tr>
  </tbody>
</table>

<CodeBlockTabs defaultValue="npm" groupId="persist-install" persist>
  <CodeBlockTabsList>
    <CodeBlockTabsTrigger value="npm">
      npm
    </CodeBlockTabsTrigger>

    <CodeBlockTabsTrigger value="pnpm">
      pnpm
    </CodeBlockTabsTrigger>

    <CodeBlockTabsTrigger value="yarn">
      yarn
    </CodeBlockTabsTrigger>

    <CodeBlockTabsTrigger value="bun">
      bun
    </CodeBlockTabsTrigger>
  </CodeBlockTabsList>

  <CodeBlockTab value="npm">
    ```bash title="Schema Generation"
    npx auth@latest generate
    ```
  </CodeBlockTab>

  <CodeBlockTab value="pnpm">
    ```bash title="Schema Generation"
    pnpm dlx auth@latest generate
    ```
  </CodeBlockTab>

  <CodeBlockTab value="yarn">
    ```bash title="Schema Generation"
    yarn dlx auth@latest generate
    ```
  </CodeBlockTab>

  <CodeBlockTab value="bun">
    ```bash title="Schema Generation"
    bun x auth@latest generate
    ```
  </CodeBlockTab>
</CodeBlockTabs>

Joins (Experimental) [#joins-experimental]

Database joins is useful when Better-Auth needs to fetch related data from multiple tables in a single query.
Endpoints like `/get-session`, `/get-full-organization` and many others benefit greatly from this feature,
seeing upwards of 2x to 3x performance improvements depending on database latency.

The Prisma adapter supports joins out of the box since version `1.4.0`.
To enable this feature, you need to set the `experimental.joins` option to `true` in your auth configuration.

```ts title="auth.ts"
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  experimental: { joins: true }
});
```

<Callout type="warn">
  Please make sure that your Prisma schema has the necessary relations defined.
  If you do not see any relations in your Prisma schema, you can manually add them using the `@relation` directive
  or run our latest CLI version `npx auth@latest generate` to generate a new Prisma schema with the relations.
</Callout>

Additional Information [#additional-information]

* If you're looking for performance improvements or tips, take a look at our guide to <Link href="/docs/guides/optimizing-for-performance">performance optimizations</Link>.
* [How to use Prisma ORM with Better Auth and Next.js](https://www.prisma.io/docs/guides/betterauth-nextjs)
