---
url: https://better-auth.com/llms.txt/docs/integrations/convex
title: "Convex"
description: ""
access_date: 2026-10-27T05:40:43.490Z
current_date: 2026-10-27T05:40:43.490Z
---

Integrate Better Auth with Convex.

In this guide, we'll walk through the steps to integrate Better Auth with [Convex](https://www.convex.dev/).

## Prerequisites

### Create a Convex project

To use Better Auth with Convex, you need a Convex project. If you don’t have one, run the following command to create a new project. For more details, see the [Convex documentation](https://docs.convex.dev/home).

#### npm

```
npm create convex@latest

# Choose user authentication:
# > none
```

#### pnpm

#### yarn

#### bun

### Run convex dev

Running the CLI during setup will initialize your Convex deployment if it doesn't already exist, and keeps generated types current through the process. Keep it running.

#### npm

```
npx convex dev
```

#### pnpm

#### yarn

#### bun

## Install packages

Install Better Auth and the Convex component for Better Auth, and make sure you are using the latest version of Convex.

#### npm

```
npm install better-auth @convex-dev/better-auth
```

#### pnpm

#### yarn

#### bun

## Set environment variables

Generate a secret for encryption and generating hashes. Use the command below if you have openssl installed, or use `npx auth secret` to generate one.

#### npm

```
npx convex env set BETTER_AUTH_SECRET=$(openssl rand -base64 32)
```

#### pnpm

#### yarn

#### bun

Add your site URL to your Convex deployment.

#### npm

```
npx convex env set SITE_URL http://localhost:3000
```

#### pnpm

#### yarn

#### bun

Add environment variables to the `.env.local` file created by `npx convex dev`. It will be picked up by your framework dev server.

#### Cloud

```
# Deployment used by \`npx convex dev\`
CONVEX_DEPLOYMENT=dev:adjective-animal-123 # team: team-name, project: project-name

NEXT_PUBLIC_CONVEX_URL=https://adjective-animal-123.convex.cloud

# Same as NEXT_PUBLIC_CONVEX_URL but ends in .site
NEXT_PUBLIC_CONVEX_SITE_URL=https://adjective-animal-123.convex.site

# Your local site URL
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

#### Self hosted

## Add Convex auth config

Add a `convex/auth.config.ts` file to configure Better Auth as an authentication provider.

```
import { getAuthConfigProvider } from "@convex-dev/better-auth/auth-config";
import type { AuthConfig } from "convex/server";

export default {
  providers: [getAuthConfigProvider()],
} satisfies AuthConfig;
```

## Create the Better Auth Convex Component

Convex components can be installed from NPM or a local folder. While the [NPM version](https://labs.convex.dev/better-auth) is available, this guide uses a local folder setup to unlock the full potential of Better Auth.

### Create the component definition

Create a `convex/betterAuth/convex.config.ts` file to define the component. This will signal to Convex that the `convex/betterAuth` directory is a locally installed component.

```
import { defineComponent } from "convex/server";

const component = defineComponent("betterAuth");

export default component;
```

### Register the component

Register the Better Auth component in your Convex project.

```
import { defineApp } from "convex/server";
import betterAuth from "./betterAuth/convex.config";

const app = defineApp();

app.use(betterAuth);

export default app;
```

### Create a Better Auth instance

Create a Better Auth instance and initialize the component.

```
import { createClient } from "@convex-dev/better-auth";
import { convex } from "@convex-dev/better-auth/plugins";
import type { GenericCtx } from "@convex-dev/better-auth/utils";
import type { BetterAuthOptions } from "better-auth";
import { betterAuth } from "better-auth";
import { components } from "../_generated/api";
import type { DataModel } from "../_generated/dataModel";
import authConfig from "../auth.config";
import schema from "./schema";

// Better Auth Component
export const authComponent = createClient<DataModel, typeof schema>(
  components.betterAuth,
  {
    local: { schema },
    verbose: false,
  },
);

// Better Auth Options
export const createAuthOptions = (ctx: GenericCtx<DataModel>) => {
  return {
    appName: "My App",
    baseURL: process.env.SITE_URL,
    secret: process.env.BETTER_AUTH_SECRET,
    database: authComponent.adapter(ctx),
    emailAndPassword: {
      enabled: true,
    },
    plugins: [convex({ authConfig })],
  } satisfies BetterAuthOptions;
};

// For \`auth\` CLI
export const options = createAuthOptions({} as GenericCtx<DataModel>);

// Better Auth Instance
export const createAuth = (ctx: GenericCtx<DataModel>) => {
  return betterAuth(createAuthOptions(ctx));
};
```

### Generate the schema

After configuring your Better Auth instance, run the command below to generate `convex/betterAuth/schema.ts`.

#### npm

```
npx auth generate --config ./convex/betterAuth/auth.ts --output ./convex/betterAuth/schema.ts
```

#### pnpm

#### yarn

#### bun

### Export adapter functions

Export adapter functions for the Better Auth component.

```
import { createApi } from "@convex-dev/better-auth";
import { createAuthOptions } from "./auth";
import schema from "./schema";

export const {
  create,
  findOne,
  findMany,
  updateOne,
  updateMany,
  deleteOne,
  deleteMany,
} = createApi(schema, createAuthOptions);
```

## Create the Better Auth client instance

Create the Better Auth client instance for interacting with the Better Auth server from your client.

```
import { convexClient } from "@convex-dev/better-auth/client/plugins";
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  plugins: [convexClient()],
});
```

## Configure Next.js server helpers

Configure a set of helper functions for authenticated SSR, server functions, and route handlers.

```
import { convexBetterAuthNextJs } from "@convex-dev/better-auth/nextjs";

export const {
  handler,
  preloadAuthQuery,
  isAuthenticated,
  getToken,
  fetchAuthQuery,
  fetchAuthMutation,
  fetchAuthAction,
} = convexBetterAuthNextJs({
  convexUrl: process.env.NEXT_PUBLIC_CONVEX_URL!,
  convexSiteUrl: process.env.NEXT_PUBLIC_CONVEX_SITE_URL!,
});
```

## Mount handlers

Register Better Auth route handlers on your Convex deployment.

```
import { httpRouter } from "convex/server";
import { authComponent, createAuth } from "./betterAuth/auth";

const http = httpRouter();

authComponent.registerRoutes(http, createAuth);

export default http;
```

Set up route handlers to proxy auth requests from Next.js to your Convex deployment.

```
import { handler } from "@/lib/auth-server";

export const { GET, POST } = handler;
```

## Set up Convex client provider

Create a `ConvexClientProvider` component.

```
"use client";

import { ConvexBetterAuthProvider } from "@convex-dev/better-auth/react";
import { ConvexReactClient } from "convex/react";
import { authClient } from "@/lib/auth-client";

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

export function ConvexClientProvider({
  children,
  initialToken,
}: {
  children: React.ReactNode;
  initialToken?: string | null;
}) {
  return (
    <ConvexBetterAuthProvider
      client={convex}
      authClient={authClient}
      initialToken={initialToken}
    >
      {children}
    </ConvexBetterAuthProvider>
  );
}
```

Wrap your app with the `ConvexClientProvider` component.

```
import { ConvexClientProvider } from "@/components/ConvexClientProvider";
import { getToken } from "@/lib/auth-server";

export default async function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  const token = await getToken();
  return (
    <html>
      <body>
        <ConvexClientProvider initialToken={token}>
          {children}
        </ConvexClientProvider>
      </body>
    </html>
  );
}
```

## Done 🎉

You're now ready to start using Better Auth with Convex.

---

## Usage

### Create your functions

Better Auth's `auth.api` methods would normally run in your server, but with Convex being your backend, these methods need to run in a Convex function. The Convex function can then be called from the client via hooks like `useMutation` or in server functions and other server code using one of the auth-server utilities like `fetchAuthMutation`.

Convex has a convention of creating functions in the `convex/` directory. For example, you can create auth-related functions in the `convex/auth.ts` file like this:

```
import { query } from "./_generated/server";

export const getCurrentUser = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    return identity;
  },
});

// ... other functions
```

### Using the Better Auth client

You can use the `authClient` like the Better Auth client normally.

```
"use client";

import { Button } from "@/components/ui/button";
import { authClient } from "@/lib/auth-client";

const Page = () => {
  return (
    <div className="flex flex-col items-center justify-center h-screen">
      <Button
        onClick={async () => {
          await authClient.signIn.social({
            provider: "github",
            callbackURL: "/dashboard",
          });
        }}
      >
        Sign in with GitHub
      </Button>
    </div>
  );
};

export default Page;
```

### Using the Convex React hooks

If you've created Convex functions, you can use the Convex React hooks to query or mutate data in the client side like the usual way.

```
"use client";

import { useQuery } from "convex/react";
import { api } from "@/convex/_generated/api";

const Page = () => {
  const user = useQuery(api.auth.getCurrentUser);

  if (user === undefined) {
    return <div>Loading...</div>;
  }

  if (user === null) {
    return <div>Unauthorized</div>;
  }

  return (
    <div>
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </div>
  );
};

export default Page;
```

### Using the server helpers

You can use the server helpers to perform authentication related operations in the server side.

```
import { isAuthenticated } from "@/lib/auth-server";

const Page = async () => {
  const hasToken = await isAuthenticated();
  if (!hasToken) {
    return <div>Unauthorized</div>;
  }

  return (
    <div>
      <p>Hello</p>
    </div>
  );
};

export default Page;
```

### SSR with server components

Convex queries can be preloaded in server components and rendered in client components via `preloadAuthQuery` and `usePreloadedAuthQuery`.

*Preloading in a server component:*

```
import { api } from "@/convex/_generated/api";
import { preloadAuthQuery } from "@/lib/auth-server";
import Header from "./header";

const Page = async () => {
  const preloadedUserQuery = await preloadAuthQuery(api.auth.getCurrentUser);

  return (
    <div>
      <Header preloadedUserQuery={preloadedUserQuery} />
    </div>
  );
};

export default Page;
```

*Rendering preloaded data in a client component:*

```
"use client";

import { usePreloadedAuthQuery } from "@convex-dev/better-auth/nextjs/client";
import type { Preloaded } from "convex/react";
import Link from "next/link";
import { Button } from "@/components/ui/button";
import type { api } from "@/convex/_generated/api";

export const Header = ({
  preloadedUserQuery,
}: {
  preloadedUserQuery: Preloaded<typeof api.auth.getCurrentUser>;
}) => {
  const user = usePreloadedAuthQuery(preloadedUserQuery);
  return (
    <div>
      {user ? (
        <Button asChild>
          <Link href="/dashboard">Dashboard</Link>
        </Button>
      ) : (
        <Button asChild>
          <Link href="/sign-in">Sign in</Link>
        </Button>
      )}
    </div>
  );
};

export default Header;
```
