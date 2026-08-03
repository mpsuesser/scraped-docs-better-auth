---
url: https://better-auth.com/llms.txt/docs/installation
title: "Installation"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Learn how to configure Better Auth in your project.

### Install the Package

Let's start by adding Better Auth to your project:

#### npm

```
npm install better-auth
```

#### pnpm

#### yarn

#### bun

### Set Environment Variables

Create a `.env` file in the root of your project and add the following environment variables:

1. **Secret Key**

A secret value used for encryption and hashing. It must be at least 32 characters and generated with high entropy. Click the button below to generate one. You can also use `openssl rand -base64 32` to generate one.

```
BETTER_AUTH_SECRET=
```

2. **Set Base URL**

```
BETTER_AUTH_URL=http://localhost:3000 # Base URL of your app
```

### Create A Better Auth Instance

Create a file named `auth.ts` in one of these locations:

- Project root
- `lib/` folder
- `utils/` folder

You can also nest any of these folders under `src/`, `app/` or `server/` folder. (e.g. `src/lib/auth.ts`, `app/lib/auth.ts`).

And in this file, import Better Auth and create your auth instance. Make sure to export the auth instance with the variable name `auth` or as a `default` export.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  //...
});
```

### Configure Database

Better Auth requires a database to store user data. You can easily configure Better Auth to use SQLite, PostgreSQL, or MySQL, and more!

#### sqlite

```
import { betterAuth } from "better-auth";
import Database from "better-sqlite3";

export const auth = betterAuth({
    database: new Database("./sqlite.db"),
})
```

#### postgres

#### mysql

Alternatively, if you prefer to use an ORM, you can use one of the built-in adapters.

#### drizzle

```
import { betterAuth } from "better-auth";
import { drizzleAdapter } from "better-auth/adapters/drizzle";
import { db } from "@/db"; // your drizzle instance

export const auth = betterAuth({
    database: drizzleAdapter(db, {
        provider: "pg", // or "mysql", "sqlite"
    }),
});
```

#### prisma

#### mongodb

If your database is not listed above, check out our other supported [databases](https://better-auth.com/docs/adapters/other-relational-databases) for more information, or use one of the supported ORMs.

### Create Database Tables

Better Auth includes a CLI tool to help manage the schema required by the library.

- **Generate**: This command generates an ORM schema or SQL migration file.

#### npm

```
npx auth@latest generate
```

#### pnpm

#### yarn

#### bun

- **Migrate**: This command creates the required tables directly in the database. (Available only for the built-in Kysely adapter)
	#### npm
	```
	npx auth@latest migrate
	```
	#### pnpm
	#### yarn
	#### bun

see the [CLI documentation](https://better-auth.com/docs/concepts/cli) for more information.

### Authentication Methods

Configure the authentication methods you want to use. Better Auth comes with built-in support for email/password, and social sign-on providers.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  //...other options
  emailAndPassword: { 
    enabled: true, 
  }, 
  socialProviders: { 
    github: { 
      clientId: process.env.GITHUB_CLIENT_ID as string, 
      clientSecret: process.env.GITHUB_CLIENT_SECRET as string, 
    }, 
  }, 
});
```

### Mount Handler

To handle API requests, you need to set up a route handler on your server.

Create a new file or route in your framework's designated catch-all route handler. This route should handle requests for the path `/api/auth/*` (unless you've configured a different base path).

#### next-js-app-router

```
import { auth } from "@/lib/auth"; // path to your auth file
import { toNextJsHandler } from "better-auth/next-js";

export const { POST, GET } = toNextJsHandler(auth);
```

#### nuxt

#### svelte-kit

#### react-router

#### solid-start

#### hono

#### cloudflare-workers

#### express

#### elysia

#### tanstack-start

#### expo

### Create Client Instance

The client-side library helps you interact with the auth server. Better Auth comes with a client for all the popular web frameworks, including vanilla JavaScript.

1. Import `createAuthClient` from the package for your framework (e.g., "better-auth/react" for React).
2. Call the function to create your client.
3. Pass the base URL of your auth server. (If the auth server is running on the same domain as your client, you can skip this step.)

#### react

#### vue

```
import { createAuthClient } from "better-auth/react"
export const authClient = createAuthClient({
    /** The base URL of the server (optional if you're using the same domain) */
    baseURL: "http://localhost:3000"
})
```

#### svelte

#### solid

#### vanilla

```
export const { signIn, signUp, useSession } = createAuthClient()
```

### 🎉 That's it!

That's it! You're now ready to use better-auth in your application. Continue to [basic usage](https://better-auth.com/docs/basic-usage) to learn how to use the auth instance to sign in users.
