---
url: https://better-auth.com/llms.txt/docs/integrations/nitro
title: "Nitro"
description: ""
access_date: 2026-09-11T23:52:38.819Z
current_date: 2026-09-11T23:52:38.819Z
---

Integrate Better Auth with Nitro.

Better Auth can be integrated with your [Nitro Application](https://nitro.build/) (an open source framework to build web servers).

This guide aims to help you integrate Better Auth with your Nitro application in a few simple steps.

## Create a new Nitro Application

Start by scaffolding a new Nitro application using the following command:

```
npx create-nitro-app
```

This will create the `nitro-app` directory and install all the dependencies. You can now open the `nitro-app` directory in your code editor.

### Prisma Adapter Setup

For this guide, we will be using the Prisma adapter. You can install prisma client by running the following command:

#### npm

```
npm install @prisma/client
```

#### pnpm

#### yarn

#### bun

`prisma` can be installed as a dev dependency using the following command:

#### npm

```
npm install -D prisma
```

#### pnpm

#### yarn

#### bun

Generate a `schema.prisma` file in the `prisma` directory by running the following command:

```
npx prisma init
```

You can now replace the contents of the `schema.prisma` file with the following:

```
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

// Will be deleted. Just need it to generate the prisma client
model Test {
  id   Int    @id @default(autoincrement())
  name String
}
```

Ensure that you update the `DATABASE_URL` in your `.env` file to point to the location of your database.

```
DATABASE_URL="file:./dev.db"
```

Run the following command to generate the Prisma client & sync the database:

```
npx prisma db push
```

### Install & Configure Better Auth

Follow steps 1 & 2 from the [installation guide](https://better-auth.com/docs/installation) to install Better Auth in your Nitro application & set up the environment variables.

Once that is done, create your Better Auth instance within the `server/utils/auth.ts` file.

```
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export default betterAuth({
  database: prismaAdapter(prisma, { provider: "sqlite" }),
  emailAndPassword: { enabled: true },
});
```

### Update Prisma Schema

Use the Better Auth CLI to update your Prisma schema with the required models by running the following command:

```
npx auth generate --config server/utils/auth.ts
```

Head over to the `prisma/schema.prisma` file & save the file to trigger the format on save.

After saving the file, you can run the `npx prisma db push` command to update the database schema.

## Mount The Handler

You can now mount the Better Auth handler in your Nitro application. You can do this by adding the following code to your `server/api/auth/[...all].ts` file:

```
import auth from "~/server/utils/auth";

export default auth;
```

### CORS

You can configure CORS for your Nitro app using a [route rule](https://nitro.build/docs/routing#cors) in your `nitro.config.ts`:

```
import { defineConfig } from "nitro";

export default defineConfig({
  routeRules: {
    "/api/auth/**": {
      cors: {
        origin: ["http://localhost:3000"],
        credentials: true,
      },
    },
  },
});
```

Learn more about CORS on the [Nitro documentation](https://nitro.build/docs/routing#cors).

### Auth Guard/Middleware

You can add an auth guard to your Nitro application to protect routes that require authentication. You can do this by creating a new file `server/utils/require-auth.ts` and adding the following code:

```
import { defineHandler, HTTPError } from "nitro";
import auth from "~/server/utils/auth.ts";

/**
 * Middleware used to require authentication for a route.
 *
 * Can be extended to check for specific roles or permissions.
 */
export default defineHandler(async (event) => {
  const session = await auth.api.getSession({
    headers: event.req.headers,
  });

  if (!session) {
    throw HTTPError.status(401, "Unauthorized");
  }

  // You can save the session to the event context for later use
  event.context.auth = session;
});
```

You can now use the [Object Syntax Event Handler](https://h3.dev/guide/basics/handler#object-syntax) to apply middleware to specific routes:

```
import { defineHandler } from "nitro";
import requireAuth from "~/server/utils/require-auth.ts";

export default defineHandler({
  middleware: [requireAuth],
  handler: () =>
    Response.json(
      { message: "Secret data" },
      { status: 201, statusText: "Secret data" },
    ),
});
```

### Example

See an [example Nitro application integrated with Better Auth & Prisma](https://github.com/BayBreezy/nitrojs-better-auth-prisma).
