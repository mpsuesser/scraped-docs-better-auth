---
url: https://better-auth.com/llms.txt/docs/adapters/mongo
title: "Mongo"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Integrate Better Auth with MongoDB.

MongoDB is a popular NoSQL database that is widely used for building scalable and flexible applications. It provides a flexible schema that allows for easy data modeling and querying.

Before getting started, make sure you have MongoDB installed and configured. For more information, see [MongoDB Documentation](https://www.mongodb.com/docs/)

## Installation

To use the MongoDB adapter, you need to install the `@better-auth/mongo-adapter` package:

#### npm

```
npm install @better-auth/mongo-adapter
```

#### pnpm

#### yarn

#### bun

## Example Usage

You can use the MongoDB adapter to connect to your database as follows.

```
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

## Schema generation & migration

For MongoDB, we don't need to generate or migrate the schema.

## Joins (Experimental)

Database joins is useful when Better-Auth needs to fetch related data from multiple tables in a single query. Endpoints like `/get-session`, `/get-full-organization` and many others benefit greatly from this feature, seeing upwards of 2x to 3x performance improvements depending on database latency.

The MongoDB adapter supports joins out of the box since version `1.4.0`. To enable this feature, you need to set the `experimental.joins` option to `true` in your auth configuration.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  experimental: { joins: true }
});
```

## Additional Information

- If you're looking for performance improvements or tips, take a look at our guide to [performance optimizations](https://better-auth.com/docs/guides/optimizing-for-performance).
