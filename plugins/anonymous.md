---
url: https://better-auth.com/llms.txt/docs/plugins/anonymous
title: "Anonymous"
description: ""
access_date: 2026-08-25T05:48:23.554Z
current_date: 2026-08-25T05:48:23.554Z
---

Anonymous plugin for Better Auth.

The Anonymous plugin allows users to have an authenticated experience without requiring them to provide an email address, password, OAuth provider, or any other Personally Identifiable Information (PII). Users can later link an authentication method to their account when ready.

## Installation

### Add the plugin to your auth config

To enable anonymous authentication, add the anonymous plugin to your authentication configuration.

```
import { betterAuth } from "better-auth"
import { anonymous } from "better-auth/plugins"

export const auth = betterAuth({
    // ... other config options
    plugins: [
        anonymous() 
    ]
})
```

### Migrate the database

Run the migration or generate the schema to add the necessary fields and tables to the database.

#### migrate

#### npm

#### generate

```
npx auth migrate
```

#### pnpm

#### yarn

#### bun

See the [Schema](#schema) section to add the fields manually.

### Add the client plugin

Next, include the anonymous client plugin in your authentication client instance.

```
import { createAuthClient } from "better-auth/client"
import { anonymousClient } from "better-auth/client/plugins"

export const authClient = createAuthClient({
    plugins: [
        anonymousClient() 
    ]
})
```

## Usage

### Sign In Anonymously

To sign in a user anonymously, use the `signIn.anonymous()` method. This creates a new user with a generated email and the configured or default name, then establishes a session.

POST/sign-in/anonymous

```
const { data, error } = await authClient.signIn.anonymous();
```

### Link Account

If a user is already signed in anonymously and tries to `signIn` or `signUp` with another method, their anonymous activities can be linked to the new account.

To do that you first need to provide `onLinkAccount` callback to the plugin.

```
import { betterAuth } from "better-auth"
import { anonymous } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        anonymous({
            onLinkAccount: async ({ anonymousUser, newUser }) => {
               // perform actions like moving the cart items from anonymous user to the new user
            }
        })
    ]
```

Then when you call `signIn` or `signUp` with another method, the `onLinkAccount` callback will be called. And the `anonymousUser` will be deleted by default.

```
import { authClient } from "@/lib/auth-client";

const user = await authClient.signIn.email({
    email,
})
```

### Delete Anonymous User

To delete an anonymous user, you can call the `/delete-anonymous-user` endpoint.

POST/delete-anonymous-user

```
await authClient.deleteAnonymousUser();
```

## Options

### emailDomainName

The domain name to use when generating an email address for anonymous users. If not provided, the default format `temp@{id}.com` is used.

```
import { betterAuth } from "better-auth"
import { anonymous } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        anonymous({
            emailDomainName: "example.com" // -> temp-{id}@example.com
        })
    ]
})
```

### generateRandomEmail

A custom function to generate email addresses for anonymous users. This allows you to define your own email format. The function can be synchronous or asynchronous.

```
import { betterAuth } from "better-auth"
import { anonymous } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        anonymous({
            generateRandomEmail: () => { 
                const id = crypto.randomUUID() 
                return \`guest-${id}@example.com\`
            } 
        })
    ]
})
```

### onLinkAccount

A callback function that is called when an anonymous user links their account to a new authentication method. The callback receives an object with the `anonymousUser` and the `newUser`.

### disableDeleteAnonymousUser

By default, when an anonymous user links their account to a new authentication method, the anonymous user record is automatically deleted. If you set this option to `true`, this automatic deletion will be disabled, and the `/delete-anonymous-user` endpoint will no longer be accessible to anonymous users.

### generateName

A callback function that is called to generate a name for the anonymous user. Useful if you want to have random names for anonymous users, or if `name` is unique in your database.

## Schema

The anonymous plugin requires an additional field in the user table:

Table

Field

Type

Attributes

Description

isAnonymous?

boolean

\-

Indicates whether the user is anonymous.
