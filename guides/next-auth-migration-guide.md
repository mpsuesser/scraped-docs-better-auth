---
url: https://better-auth.com/llms.txt/docs/guides/next-auth-migration-guide
title: "Next Auth Migration Guide"
description: ""
access_date: 2026-08-18T00:08:46.984Z
current_date: 2026-08-18T00:08:46.984Z
---

A step-by-step guide to transitioning from Auth.js to Better Auth.

## Introduction

In this guide, we'll walk through the steps to migrate a project from [Auth.js](https://authjs.dev/) (formerly [NextAuth.js](https://next-auth.js.org/)) to Better Auth. Since these projects have different design philosophies, the migration requires careful planning and work. If your current setup is working well, there's no urgent need to migrate. We continue to handle security patches and critical issues for Auth.js.

However, if you're starting a new project or facing challenges with your current setup, we strongly recommend using Better Auth. Our roadmap includes features previously exclusive to Auth.js, and we hope this will unite the ecosystem more strongly without causing fragmentation.

## Create Better Auth Instance

Before starting the migration process, set up Better Auth in your project. Follow the [installation guide](https://better-auth.com/docs/installation) to get started.

For example, if you use the GitHub OAuth provider, here is a comparison of the `auth.ts` file.

#### Auth.js

```
import NextAuth from "next-auth"
import GitHub from "next-auth/providers/github"
  
export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [GitHub],
})
```

#### Better Auth

## Create Client Instance

This client instance includes a set of functions for interacting with the Better Auth server instance. For more information, see the [client documentation](https://better-auth.com/docs/concepts/client).

```
import { createAuthClient } from "better-auth/react"

export const authClient = createAuthClient()
```

## Update the Route Handler

Rename the `/app/api/auth/[...nextauth]` folder to `/app/api/auth/[...all]` to avoid confusion. Then, update the `route.ts` file as follows:

#### Auth.js

```
import { handlers } from "@/lib/auth"

export const { GET, POST } = handlers
```

#### Better Auth

## Session Management

In this section, we'll look at how to manage sessions in Better Auth compared to Auth.js. For more information, see the [session management documentation](https://better-auth.com/docs/concepts/session-management).

### Client-side

#### Sign In

Here are the differences between Auth.js and Better Auth for signing in users. For example, with the GitHub OAuth provider:

#### Auth.js

```
"use client"

import { signIn } from "next-auth/react"

signIn("github")
```

#### Better Auth

#### Sign Out

Here are the differences between Auth.js and Better Auth when signing out users.

#### Auth.js

```
"use client"

import { signOut } from "next-auth/react"

signOut()
```

#### Better Auth

#### Get Session

Here are the differences between Auth.js and Better Auth for getting the current active session.

#### Auth.js

```
"use client"

import { useSession } from "next-auth/react"

const { data, status, update } = useSession()
```

#### Better Auth

### Server-side

#### Sign In

Here are the differences between Auth.js and Better Auth for signing in users. For example, with the GitHub OAuth provider:

#### Auth.js

```
import { signIn } from "@/lib/auth"

await signIn("github")
```

#### Better Auth

#### Sign Out

Here are the differences between Auth.js and Better Auth when signing out users.

#### Auth.js

```
import { signOut } from "@/lib/auth"

await signOut()
```

#### Better Auth

#### Get Session

Here are the differences between Auth.js and Better Auth for getting the current active session.

#### Auth.js

```
import { auth } from "@/lib/auth";

const session = await auth()
```

#### Better Auth

## Protecting Resources

> Proxy(Middleware) is not intended for slow data fetching. While Proxy can be helpful for optimistic checks such as permission-based redirects, it should not be used as a full session management or authorization solution. - [Next.js docs](https://nextjs.org/docs/app/getting-started/proxy#use-cases)

Auth.js offers approaches using Proxy (Middleware), but we recommend handling auth checks on each page or route rather than in a Proxy or Layout. Here is a basic example of protecting resources with Better Auth.

#### Client-side

```
"use client";

import { authClient } from "@/lib/auth-client";
import { redirect } from "next/navigation";

const DashboardPage = () => {
  const { data, error, isPending } = authClient.useSession();

  if (isPending) {
    return <div>Pending</div>;
  }
  if (!data || error) {
    redirect("/sign-in");
  }

  return (
    <div>
      <h1>Welcome {data.user.name}</h1>
    </div>
  );
};

export default DashboardPage;
```

#### Server-side

## Database models

Both Auth.js and Better Auth provide stateless (JWT) and database session strategies. If you were using the database session strategy in Auth.js and plan to continue using it in Better Auth, you will also need to migrate your database.

Just like Auth.js has database models, Better Auth also has a core schema. In this section, we'll compare the two and explore the differences between them.

### User -> User

#### Auth.js

Table

Field

Type

Key

Description

id

string

PK

Unique identifier for each user

name?

string

\-

User's chosen display name

emailVerified?

Date

\-

When the user's email was verified

image?

string

\-

User's image url

#### Better Auth

### Session -> Session

#### Auth.js

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

sessionToken

string

\-

The unique session token

expires

Date

\-

The time when the session expires

#### Better Auth

### Account -> Account

#### Auth.js

Table

Field

Type

Key

Description

id

string

PK

Unique identifier for each account

userId

string

FK

The ID of the user

type

string

\-

Type of the account (e.g. 'oauth', 'email', 'credentials')

provider

string

\-

Provider identifier

providerAccountId

string

\-

Account ID from the provider

refresh\_token?

string

\-

The refresh token of the account. Returned by the provider

access\_token?

string

\-

The access token of the account. Returned by the provider

expires\_at?

number

\-

The time when the access token expires

token\_type?

string

\-

Type of the token

scope?

string

\-

The scope of the account. Returned by the provider

id\_token?

string

\-

The ID token returned from the provider

session\_state?

string

\-

OAuth session state

#### Better Auth

### VerificationToken -> Verification

#### Auth.js

Table

Field

Type

Key

Description

identifier

string

PK

Identifier for the verification request

token

string

PK

The verification token

expires

Date

\-

The time when the verification token expires

#### Better Auth

### Comparison

Table: **User**

- `name`, `email`, and `emailVerified` are required in Better Auth, while optional in Auth.js
- `emailVerified` uses a boolean in Better Auth, while Auth.js uses a timestamp
- Better Auth includes `createdAt` and `updatedAt` timestamps

Table: **Session**

- Better Auth uses `token` instead of `sessionToken`
- Better Auth uses `expiresAt` instead of `expires`
- Better Auth includes `ipAddress` and `userAgent` fields
- Better Auth includes `createdAt` and `updatedAt` timestamps

Table: **Account**

- Better Auth uses camelCase naming (e.g. `refreshToken` vs `refresh_token`)
- Better Auth uses `id` for the local account record and the unique `issuer` plus `accountId` pair for the external identity
- Better Auth uses `providerId` instead of `provider`
- Better Auth includes `accessTokenExpiresAt` and `refreshTokenExpiresAt` for token management
- Better Auth includes `password` field to support built-in credential authentication
- Better Auth does not have a `type` field as it's determined by the `providerId`
- Better Auth removes `token_type` and `session_state` fields
- Better Auth includes `createdAt` and `updatedAt` timestamps

Table: **VerificationToken -> Verification**

- Better Auth uses `Verification` table instead of `VerificationToken`
- Better Auth uses a single `id` primary key instead of composite primary key
- Better Auth uses `value` instead of `token` to support various verification types
- Better Auth uses `expiresAt` instead of `expires`
- Better Auth includes `createdAt` and `updatedAt` timestamps

### Customization

You may have extended the database models or implemented additional logic in Auth.js. Better Auth allows you to customize the core schema in a type-safe way. You can also define custom logic during the lifecycle of database operations. For more details, see [Concepts - Database](https://better-auth.com/docs/concepts/database).

## Wrapping Up

Now you're ready to migrate from Auth.js to Better Auth. For a complete implementation with multiple authentication methods, check out the [Next.js Demo App](https://github.com/better-auth/better-auth/tree/main/demo/nextjs). Better Auth offers greater flexibility and more features, so be sure to explore the [documentation](https://better-auth.com/docs) to unlock its full potential.

If you need help with migration, join our [community](https://better-auth.com/community) or reach out to [contact@better-auth.com](mailto:contact@better-auth.com).
