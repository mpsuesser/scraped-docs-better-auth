---
url: https://better-auth.com/llms.txt/docs/adapters/mssql
title: "Mssql"
description: ""
access_date: 2026-08-03T19:38:28.543Z
current_date: 2026-08-03T19:38:28.543Z
---

# MS SQL

Integrate Better Auth with MS SQL.



Microsoft SQL Server is a relational database management system developed by Microsoft, designed for enterprise-level data storage, management, and analytics with robust security and scalability features.
Read more about [Microsoft SQL Server](https://en.wikipedia.org/wiki/Microsoft_SQL_Server).

Example Usage [#example-usage]

Make sure you have MS SQL installed and configured.
Then, you can connect it straight into Better Auth.

```ts title="auth.ts"
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

<Callout>
  For more information, read Kysely's documentation to the [MssqlDialect](https://kysely-org.github.io/kysely-apidoc/classes/MssqlDialect.html).
</Callout>

Schema generation & migration [#schema-generation--migration]

The [Better Auth CLI](/docs/concepts/cli) allows you to generate or migrate
your database schema based on your Better Auth configuration and plugins.

<table>
  <thead>
    <tr className="border-b">
      <th>
        <p className="font-bold text-[16px] mb-1">
          MS SQL Schema Generation
        </p>
      </th>

      <th>
        <p className="font-bold text-[16px] mb-1">
          MS SQL Schema Migration
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
        ✅ Supported
      </td>
    </tr>
  </tbody>
</table>

<Tabs items={["migrate", "generate"]}>
  <Tab value="migrate">
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
        npx auth@latest migrate
        ```
      </CodeBlockTab>

      <CodeBlockTab value="pnpm">
        ```bash
        pnpm dlx auth@latest migrate
        ```
      </CodeBlockTab>

      <CodeBlockTab value="yarn">
        ```bash
        yarn dlx auth@latest migrate
        ```
      </CodeBlockTab>

      <CodeBlockTab value="bun">
        ```bash
        bun x auth@latest migrate
        ```
      </CodeBlockTab>
    </CodeBlockTabs>
  </Tab>

  <Tab value="generate">
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
        npx auth@latest generate
        ```
      </CodeBlockTab>

      <CodeBlockTab value="pnpm">
        ```bash
        pnpm dlx auth@latest generate
        ```
      </CodeBlockTab>

      <CodeBlockTab value="yarn">
        ```bash
        yarn dlx auth@latest generate
        ```
      </CodeBlockTab>

      <CodeBlockTab value="bun">
        ```bash
        bun x auth@latest generate
        ```
      </CodeBlockTab>
    </CodeBlockTabs>
  </Tab>
</Tabs>

Joins (Experimental) [#joins-experimental]

Database joins is useful when Better-Auth needs to fetch related data from multiple tables in a single query.
Endpoints like `/get-session`, `/get-full-organization` and many others benefit greatly from this feature,
seeing upwards of 2x to 3x performance improvements depending on database latency.

The Kysely MS SQL dialect supports joins out of the box since version `1.4.0`.
To enable this feature, you need to set the `experimental.joins` option to `true` in your auth configuration.

```ts title="auth.ts"
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  experimental: { joins: true }
});
```

<Callout type="warn">
  It's possible that you may need to run migrations after enabling this feature.
</Callout>

Additional Information [#additional-information]

MS SQL is supported under the hood via the [Kysely](https://kysely.dev/) adapter, any database supported by Kysely would also be supported. (<Link href="/docs/adapters/other-relational-databases">Read more here</Link>)

If you're looking for performance improvements or tips, take a look at our guide to <Link href="/docs/guides/optimizing-for-performance">performance optimizations</Link>.
