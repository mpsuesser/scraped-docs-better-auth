---
url: https://better-auth.com/llms.txt/docs/adapters/mongo
title: "Mongo"
description: ""
access_date: 2026-08-03T18:54:22.481Z
current_date: 2026-08-03T18:54:22.481Z
---

# MongoDB Adapter

Integrate Better Auth with MongoDB.



MongoDB is a popular NoSQL database that is widely used for building scalable and flexible applications. It provides a flexible schema that allows for easy data modeling and querying.

Before getting started, make sure you have MongoDB installed and configured. For more information, see [MongoDB Documentation](https://www.mongodb.com/docs/)

Installation [#installation]

To use the MongoDB adapter, you need to install the `@better-auth/mongo-adapter` package:

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
    npm install @better-auth/mongo-adapter
    ```
  </CodeBlockTab>

  <CodeBlockTab value="pnpm">
    ```bash
    pnpm add @better-auth/mongo-adapter
    ```
  </CodeBlockTab>

  <CodeBlockTab value="yarn">
    ```bash
    yarn add @better-auth/mongo-adapter
    ```
  </CodeBlockTab>

  <CodeBlockTab value="bun">
    ```bash
    bun add @better-auth/mongo-adapter
    ```
  </CodeBlockTab>
</CodeBlockTabs>

Example Usage [#example-usage]

You can use the MongoDB adapter to connect to your database as follows.

```ts title="auth.ts"
import { betterAuth } from "better-auth";
import { MongoClient } from "mongodb";
import { mongodbAdapter } from "better-auth/adapters/mongodb";

const client = new MongoClient("mongodb://localhost:27017/database");
const db = client.db();

export const auth = betterAuth({
  database: mongodbAdapter(db, {
    // Optional: if you don't provide a client, database transactions won't be enabled.
    client
  }),
});
```

Schema generation & migration [#schema-generation--migration]

For MongoDB, we don't need to generate or migrate the schema.

Joins (Experimental) [#joins-experimental]

Database joins is useful when Better-Auth needs to fetch related data from multiple tables in a single query.
Endpoints like `/get-session`, `/get-full-organization` and many others benefit greatly from this feature,
seeing upwards of 2x to 3x performance improvements depending on database latency.

The MongoDB adapter supports joins out of the box since version `1.4.0`.
To enable this feature, you need to set the `experimental.joins` option to `true` in your auth configuration.

```ts title="auth.ts"
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  experimental: { joins: true }
});
```

Additional Information [#additional-information]

* If you're looking for performance improvements or tips, take a look at our guide to <Link href="/docs/guides/optimizing-for-performance">performance optimizations</Link>.
