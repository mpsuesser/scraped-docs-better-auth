---
url: https://better-auth.com/llms.txt/docs/integrations/react-router
title: "React Router"
description: ""
access_date: 2026-08-29T09:54:12.340Z
current_date: 2026-08-29T09:54:12.340Z
---

Integrate Better Auth with React Router v7 (formerly Remix).

Better Auth can be easily integrated with React Router v7. This guide will show you how to integrate Better Auth with React Router v7.

You can follow the steps from [installation](https://better-auth.com/docs/installation) to get started or you can follow this guide to make it the React Router way.

If you have followed the installation steps, you can skip the first step.

## Create auth instance

Create a file named `auth.server.ts` in one of these locations:

- Project root
- `lib/` folder
- `utils/` folder

You can also nest any of these folders under `app/` folder. (e.g. `app/lib/auth.server.ts`)

And in this file, import Better Auth and create your instance.

```
import { betterAuth } from "better-auth"

export const auth = betterAuth({
    database: {
        provider: "postgres", //change this to your database provider
        url: process.env.DATABASE_URL, // path to your database or connection string
    }
})
```

## Create API Route

We need to mount the handler to a API route. Create a resource route file `api.auth.$.ts` inside `app/routes/` directory. And add the following code:

### React Router v7

```
import { auth } from '~/lib/auth.server' // Adjust the path as necessary
import type { LoaderFunctionArgs, ActionFunctionArgs } from "react-router"

export async function loader({ request }: LoaderFunctionArgs) {
    return auth.handler(request)
}

export async function action({ request }: ActionFunctionArgs) {
    return auth.handler(request)
}
```

### Remix v2 (Legacy)

If you're still using Remix v2, the only difference is the import:

```
import { auth } from '~/lib/auth.server' // Adjust the path as necessary
import type { LoaderFunctionArgs, ActionFunctionArgs } from "@remix-run/node"

export async function loader({ request }: LoaderFunctionArgs) {
    return auth.handler(request)
}

export async function action({ request }: ActionFunctionArgs) {
    return auth.handler(request)
}
```

## Create a client

Create a client instance. Here we are creating `auth-client.ts` file inside the `lib/` directory.

```
import { createAuthClient } from "better-auth/react" // make sure to import from better-auth/react

export const authClient = createAuthClient({
    //you can pass client configuration here
})
```

Once you have created the client, you can use it to sign up, sign in, and perform other actions.
