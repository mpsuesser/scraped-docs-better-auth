---
url: https://better-auth.com/llms.txt/docs/concepts/database
title: "Database"
description: ""
access_date: 2026-08-24T11:54:16.860Z
current_date: 2026-08-24T11:54:16.860Z
---

Learn about database adapters, migrations, secondary storage with Redis, core schema (user, session, account, verification), custom tables, extending schemas, ID generation, database hooks, and plugin schemas.

## Adapters

Better Auth connects to a database to store data. The database will be used to store data such as users, sessions, and more. Plugins can also define their own database tables to store data.

You can pass a database connection to Better Auth by passing a supported database instance in the database options. You can learn more about supported database adapters in the [Other relational databases](https://better-auth.com/docs/adapters/other-relational-databases) documentation.

## CLI

Better Auth comes with a CLI tool to manage database migrations and generate schema.

### Running Migrations

The cli checks your database and prompts you to add missing tables or update existing ones with new columns. This is only supported for the built-in Kysely adapter. For other adapters, you can use the `generate` command to create the schema and handle the migration through your ORM.

#### npm

```
npx auth@latest migrate
```

#### pnpm

#### yarn

#### bun

### Generating Schema

Better Auth also provides a `generate` command to generate the schema required by Better Auth. If you're using a database adapter like Prisma or Drizzle, this command will generate the right schema for your ORM. If you're using the built-in Kysely adapter, it will generate an SQL file you can run directly on your database.

#### npm

```
npx auth@latest generate
```

#### pnpm

#### yarn

#### bun

See the [CLI](https://better-auth.com/docs/concepts/cli) documentation for more information on the CLI.

### Programmatic Migrations

In environments where the CLI isn't available (e.g. Cloudflare Workers, serverless functions), you can run migrations programmatically using `getMigrations` from `better-auth/db/migration`.

```
import { getMigrations } from "better-auth/db/migration";
import { auth } from "./auth";

const { toBeCreated, toBeAdded, runMigrations } = await getMigrations(auth.options);

await runMigrations();
```

## Secondary Storage

Secondary storage in Better Auth allows you to use key-value stores for managing session data, verification records, rate limiting counters, and other short-lived auth data. This can be useful when you want to offload the storage of intensive records to a high performance storage or even RAM.

### Implementation

To use secondary storage, implement the `SecondaryStorage` interface:

```
interface SecondaryStorage {
  get: (key: string) => Promise<unknown>;
  getAndDelete: (key: string) => Promise<unknown>;
  increment: (key: string, ttl: number) => Promise<number>;
  set: (key: string, value: string, ttl?: number) => Promise<void>;
  delete: (key: string) => Promise<void>;
}
```

Then, provide your implementation to the `betterAuth` function:

```
import { betterAuth } from "better-auth";

betterAuth({
  // ... other options
  secondaryStorage: {
    // Your implementation here
  },
});
```

### Redis Storage

For most applications, we recommend using the official Redis storage package, which uses [ioredis](https://github.com/redis/ioredis).

#### Official Redis Storage

#### npm

```
npm install @better-auth/redis-storage ioredis
```

#### pnpm

#### yarn

#### bun

```
import { betterAuth } from "better-auth";
import { Redis } from "ioredis";
import { redisStorage } from "@better-auth/redis-storage";

const redis = new Redis(process.env.REDIS_URL!);

export const auth = betterAuth({
    // ... other options
    secondaryStorage: redisStorage({
        client: redis,
        keyPrefix: "better-auth:", // optional, defaults to "better-auth:"
    }),
});
```

#### Custom Implementations

#### node-redis

If you're using Redis 7 or later, you can implement secondary storage with [node-redis](https://github.com/redis/node-redis):

#### npm

#### Upstash Redis

```
npm install redis
```

#### pnpm

#### yarn

#### bun

```
import { betterAuth, type SecondaryStorage } from "better-auth";
import { createClient } from "redis";

const redis = createClient();
await redis.connect();

const redisSecondaryStorage: SecondaryStorage = {
  get(key) {
    return redis.get(key);
  },

  getAndDelete(key) {
    return redis.getDel(key);
  },

  async increment(key, ttl) {
    if (!Number.isInteger(ttl) || ttl <= 0) {
      throw new TypeError("Redis increment TTL must be a positive integer");
    }

    const [value] = await redis
      .multi()
      .incr(key)
      .expire(key, ttl, "NX")
      .execTyped();

    return value;
  },

  async set(key, value, ttl) {
    if (ttl) await redis.set(key, value, { EX: ttl });
    else await redis.set(key, value);
  },

  async delete(key) {
    await redis.del(key);
  },
};

export const auth = betterAuth({
  // ... other options
  secondaryStorage: redisSecondaryStorage,
});
```

When implementing secondary storage directly, prefix its keys to avoid collisions with other applications sharing the same Redis database.

## Core Schema

Better Auth requires the following tables to be present in the database. The types are in `typescript` format. You can use corresponding types in your database.

### User

Table Name: `user`

Table

Field

Type

Key

Description

id

string

PK

Unique identifier for each user

name

string

\-

User's chosen display name

emailVerified

boolean

\-

Whether the user's email is verified

image?

string

\-

User's image url

createdAt

Date

\-

Timestamp of when the user account was created

updatedAt

Date

\-

Timestamp of the last update to the user's information

### Session

Table Name: `session`

Table

Field

Type

Key

Description

id

string

PK

Unique identifier for each session

userId

string

FK

The ID of the user

token

string

\-

The unique session token

expiresAt

Date

\-

The time when the session expires

ipAddress?

string

\-

The IP address of the device

userAgent?

string

\-

The user agent information of the device

createdAt

Date

\-

Timestamp of when the session was created

updatedAt

Date

\-

Timestamp of when the session was updated

### Account

Table Name: `account`

An account represents one authentication method linked to a user. Better Auth recognizes the provider-side identity by the unique pair of `issuer` and `accountId`, while `id` identifies the local account row. Use `id` when an account API asks for an `accountId`.

Table

Field

Type

Key

Description

id

string

PK

Unique identifier for the local account record

userId

string

FK

The ID of the user

issuer

string

\-

The trusted authority that issued the provider account identifier

accountId

string

\-

The stable account identifier within the issuer namespace

providerId

string

\-

The provider configuration used to authenticate the account

accessToken?

string

\-

The access token of the account. Returned by the provider

refreshToken?

string

\-

The refresh token of the account. Returned by the provider

accessTokenExpiresAt?

Date

\-

The time when the access token expires

refreshTokenExpiresAt?

Date

\-

The time when the refresh token expires

scope?

string

\-

The scope of the account. Returned by the provider

idToken?

string

\-

The ID token returned from the provider

password?

string

\-

The password of the account. Mainly used for email and password authentication

createdAt

Date

\-

Timestamp of when the account was created

updatedAt

Date

\-

Timestamp of when the account was updated

The database enforces a unique compound index on `issuer` and `accountId`. Providers with a trusted issuer use that authority, including the verified issuer for OpenID Connect. OAuth providers without an issuer use the synthetic `local:oauth:<encoded providerId>` namespace, with the provider ID segment percent-encoded. Credential accounts use `local:credential` and the linked user's stable `id`.

### Verification

Table Name: `verification`

Table

Field

Type

Key

Description

id

string

PK

Unique identifier for each verification

identifier

string

\-

The identifier for the verification request

value

string

\-

The value to be verified

expiresAt

Date

\-

The time when the verification request expires

createdAt

Date

\-

Timestamp of when the verification request was created

updatedAt

Date

\-

Timestamp of when the verification request was updated

## Custom Tables

Better Auth allows you to customize the table names and column names for the core schema. You can also extend the core schema by adding additional fields to the user and session tables.

### Custom Table Names

You can customize the table names and column names for the core schema by using the `modelName` and `fields` properties in your auth config:

```
export const auth = betterAuth({
  user: {
    modelName: "users",
    fields: {
      name: "full_name",
      email: "email_address",
    },
  },
  session: {
    modelName: "user_sessions",
    fields: {
      userId: "user_id",
    },
  },
});
```

To customize table names and column name for plugins, you can use the `schema` property in the plugin config:

```
import { betterAuth } from "better-auth";
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    twoFactor({
      schema: {
        user: {
          fields: {
            twoFactorEnabled: "two_factor_enabled",
            secret: "two_factor_secret",
          },
        },
      },
    }),
  ],
});
```

### Extending Core Schema

Better Auth provides a type-safe way to extend the `user` and `session` schemas. You can add custom fields to your auth config, and the CLI will automatically update the database schema. These additional fields will be properly inferred in functions like `useSession`, `signUp.email`, and other endpoints that work with user or session objects.

To add custom fields, use the `additionalFields` property in the `user` or `session` object of your auth config. The `additionalFields` object uses field names as keys, with each value being a `FieldAttributes` object containing:

- `type`: The data type of the field (e.g., "string", "number", "boolean").
- `required`: A boolean indicating if the field is mandatory.
- `defaultValue`: The default value for the field (note: this only applies in the JavaScript layer; in the database, the field will be optional).
- `input`: Whether Better Auth accepts the field when creating or updating a record (default: `true`). For user fields, this includes values from API input and `mapProfileToUser`. Set this to `false` for server-owned fields such as `role`.
- `returned`: Whether Better Auth includes the stored field in response bodies (default: `true`). This does not affect whether input can write the field.

Here's an example of how to extend the user schema with additional fields:

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  user: {
    additionalFields: {
      role: {
        type: ["user", "admin"],
        required: false,
        defaultValue: "user",
        input: false, // don't allow user to set role
      },
      lang: {
        type: "string",
        required: false,
        defaultValue: "en",
      },
    },
  },
});
```

Now you can access the additional fields in your application logic.

```
//on signup
const res = await auth.api.signUpEmail({
    body: {
        email: 'test@example.com',
        password: 'password',
        name: 'John Doe',
        lang: 'fr',
    },
});

//user object
res.user.role; // > "user"
res.user.lang; // > "fr"
```

For `user.additionalFields`, `input` and `returned` are independent:

| Configuration | Input behavior | Response behavior |
| --- | --- | --- |
| `{ input: true, returned: true }` | API input and provider profile mapping can supply the field. | The stored field is included. |
| `{ input: true, returned: false }` | API input and provider profile mapping can supply the field. | The stored field is omitted. |
| `{ input: false, returned: true }` | API input and provider profile mapping cannot supply the field. A configured `defaultValue` can initialize it; otherwise use an application-owned database write. | The stored field is included. |
| `{ input: false, returned: false }` | API input and provider profile mapping cannot supply the field. A configured `defaultValue` can initialize it; otherwise use an application-owned database write. | The stored field is omitted. |

If you're using a social or OAuth provider, `mapProfileToUser` can populate additional fields from the provider profile when those fields allow input. The mapper runs on your server, but Better Auth still treats its return value as provider input. Fields marked `input: false` stay server-owned, provider values for those fields are ignored, and configured `defaultValue` values still apply when OAuth creates a user. See [server-owned fields and authorization claims](https://better-auth.com/docs/concepts/oauth#server-owned-fields-and-authorization-claims) for secure handling patterns.

**Example: Mapping Profile to User For `firstName` and `lastName`**

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    github: {
      clientId: "YOUR_GITHUB_CLIENT_ID",
      clientSecret: "YOUR_GITHUB_CLIENT_SECRET",
      mapProfileToUser: (profile) => {
        return {
          firstName: profile.name.split(" ")[0],
          lastName: profile.name.split(" ")[1],
        };
      },
    },
    google: {
      clientId: "YOUR_GOOGLE_CLIENT_ID",
      clientSecret: "YOUR_GOOGLE_CLIENT_SECRET",
      mapProfileToUser: (profile) => {
        return {
          firstName: profile.given_name,
          lastName: profile.family_name,
        };
      },
    },
  },
});
```

### ID Generation

Better Auth by default will generate unique IDs for users, sessions, and other entities. You can customize ID generation behavior using the `advanced.database.generateId` option.

#### Option 1: Let Database Generate IDs

Setting `generateId` to `false` allows your database handle all ID generation: (outside of `generateId` being `serial` and some cases of `generateId` being `uuid`)

```
import { betterAuth } from "better-auth";
import { db } from "./db";

export const auth = betterAuth({
  database: db,
  advanced: {
    database: {
      generateId: false, // "serial" for auto-incrementing numeric IDs
    },
  },
});
```

#### Option 2: Custom ID Generation Function

Use a function to generate IDs. You can return `false` or `undefined` from the function to let the database generate the ID for specific models:

```
import { betterAuth } from "better-auth";
import { db } from "./db";

export const auth = betterAuth({
  database: db,
  advanced: {
    database: {
      generateId: (options) => {
        // Let database auto-generate for specific models
        if (options.model === "user" || options.model === "users") {
          return false; // Let database generate ID
        }
        // Generate UUIDs for other tables
        return crypto.randomUUID();
      },
    },
  },
});
```

#### Option 3: Consistent Custom ID Generator

Generate the same type of ID for all tables:

```
import { betterAuth } from "better-auth";
import { db } from "./db";

export const auth = betterAuth({
  database: db,
  advanced: {
    database: {
      generateId: () => crypto.randomUUID(),
    },
  },
});
```

### Numeric IDs

If you prefer auto-incrementing numeric IDs, you can set the `advanced.database.generateId` option to `"serial"`. Doing this will disable Better-Auth from generating IDs for any table, and will assume your database will generate the numeric ID automatically.

When enabled, the Better-Auth CLI will generate or migrate the schema with the `id` field as a numeric type for your database with auto-incrementing attributes associated with it.

```
import { betterAuth } from "better-auth";
import { db } from "./db";

export const auth = betterAuth({
  database: db,
  advanced: {
    database: {
      generateId: "serial",
    },
  },
});
```

### UUIDs

If you prefer UUIDs for the `id` field, you can set the `advanced.database.generateId` option to `"uuid"`. By default, Better-Auth will generate UUIDs for the `id` field for all tables, except adapters that use `PostgreSQL` where we allow the database to generate the UUID automatically.

By enabling this option, the Better-Auth CLI will generate or migrate the schema with the `id` field as a UUID type for your database. If the `uuid` type is not supported, we will generate a normal `string` type for the `id` field.

### Mixed ID Types

If you need different ID types across tables (e.g., integer IDs for users, UUID strings for sessions/accounts/verification), use a `generateId` callback function.

```
import { betterAuth } from "better-auth";
import { db } from "./db";

export const auth = betterAuth({
  database: db,
  user: {
    modelName: "users", // PostgreSQL: id serial primary key
  },
  session: {
    modelName: "session", // PostgreSQL: id text primary key
  },
  advanced: {
    database: {
      // Do NOT set useNumberId - it's global and affects all tables
      generateId: (options) => {
        if (options.model === "user" || options.model === "users") {
          return false; // Let PostgreSQL serial generate it
        }
        return crypto.randomUUID(); // UUIDs for session, account, verification
      },
    },
  },
});
```

This configuration allows you to:

- Use database auto-increment (serial, auto\_increment, etc.) for the users table
- Generate UUIDs for all other tables (session, account, verification)
- Maintain compatibility with existing schemas that use different ID types

### Database Hooks

Database hooks allow you to define custom logic that can be executed during the lifecycle of core database operations in Better Auth. You can create hooks for the following models: **user**, **session**, and **account**.

There are two types of hooks you can define:

#### 1\. Before Hook

- **Purpose**: This hook is called before the respective entity (user, session, or account) is created, updated, or deleted.
- **Behavior**: If the hook returns `false`, the operation will be aborted. And If it returns a data object, it'll replace the original payload.

#### 2\. After Hook

- **Purpose**: This hook is called after the respective entity is created or updated.
- **Behavior**: You can perform additional actions or modifications after the entity has been successfully created or updated.

**Example Usage**

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  databaseHooks: {
    user: {
      create: {
        before: async (user, ctx) => {
          // Modify the user object before it is created
          return {
            data: {
              // Ensure to return Better-Auth named fields, not the original field names in your database.
              ...user,
              firstName: user.name.split(" ")[0],
              lastName: user.name.split(" ")[1],
            },
          };
        },
        after: async (user) => {
          //perform additional actions, like creating a stripe customer
        },
      },
      delete: {
        before: async (user, ctx) => {
          console.log(\`User ${user.email} is being deleted\`);
          if (user.email.includes("admin")) {
            return false; // Abort deletion
          }

          return true; // Allow deletion
        },
        after: async (user) => {
          console.log(\`User ${user.email} has been deleted\`);
        },
      },
    },
    session: {
      delete: {
        before: async (session, ctx) => {
          console.log(\`Session ${session.token} is being deleted\`);
          if (session.userId === "admin-user-id") {
            return false; // Abort deletion
          }
          return true; // Allow deletion
        },
        after: async (session) => {
          console.log(\`Session ${session.token} has been deleted\`);
        },
      },
    },
  },
});
```

#### Throwing Errors

If you want to stop the database hook from proceeding, you can throw errors using the `APIError` class imported from `better-auth/api`.

```
import { betterAuth } from "better-auth";
import { APIError } from "better-auth/api";

export const auth = betterAuth({
  databaseHooks: {
    user: {
      create: {
        before: async (user, ctx) => {
          if (user.isAgreedToTerms === false) {
            // Your special condition.
            // Send the API error.
            throw new APIError("BAD_REQUEST", {
              message: "User must agree to the TOS before signing up.",
            });
          }
          return {
            data: user,
          };
        },
      },
    },
  },
});
```

#### Using the Context Object

The context object (`ctx`), passed as the second argument to the hook, contains useful information. For `update` hooks, this includes the current `session`, which you can use to access the logged-in user's details.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  databaseHooks: {
    user: {
      update: {
        before: async (data, ctx) => {
          // You can access the session from the context object.
          if (ctx.context.session) {
            console.log(
              "User update initiated by:",
              ctx.context.session.userId
            );
          }
          return { data };
        },
      },
    },
  },
});
```

Much like standard hooks, database hooks also provide a `ctx` object that offers a variety of useful properties. Learn more in the [Hooks Documentation](https://better-auth.com/docs/concepts/hooks#ctx).

## Plugins Schema

Plugins can define their own tables in the database to store additional data. They can also add columns to the core tables to store additional data. For example, the two factor authentication plugin adds the following columns to the `user` table:

- `twoFactorEnabled`: Whether two factor authentication is enabled for the user.
- `twoFactorSecret`: The secret key used to generate TOTP codes.
- `twoFactorBackupCodes`: Encrypted backup codes for account recovery.

To add new tables and columns to your database, you have two options:

`CLI`: Use the migrate or generate command. These commands will scan your database and guide you through adding any missing tables or columns. `Manual Method`: Follow the instructions in the plugin documentation to manually add tables and columns.

Both methods ensure your database schema stays up to date with your plugins' requirements.

## Joins

Since Better-Auth version `1.4` we've introduced database joins support. This allows Better-Auth to perform multiple database queries in a single request, reducing the number of database roundtrips. Over 50 endpoints support joins, and we're constantly adding more.

Under the hood, our adapter system supports joins natively. When joins are disabled (the default), Better-Auth falls back to making multiple database queries and combining the results.

To enable joins, update your auth config with the following:

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

Make sure your DrizzleORM or PrismaORM schema includes the necessary relationships — run our migrate or generate CLI commands to stay up-to-date.

Read the documentation regarding joins for your given adapter:

- [DrizzleORM](https://better-auth.com/docs/adapters/drizzle#joins)
- [PrismaORM](https://better-auth.com/docs/adapters/prisma#joins)
- [SQLite](https://better-auth.com/docs/adapters/sqlite#joins)
- [MySQL](https://better-auth.com/docs/adapters/mysql#joins)
- [PostgreSQL](https://better-auth.com/docs/adapters/postgresql#joins)
- [MSSQL](https://better-auth.com/docs/adapters/mssql#joins)
- [MongoDB](https://better-auth.com/docs/adapters/mongo#joins)
