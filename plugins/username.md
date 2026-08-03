---
url: https://better-auth.com/llms.txt/docs/plugins/username
title: "Username"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Username plugin

The username plugin is a lightweight plugin that adds username support to the email and password authenticator. This allows users to sign in with their username instead of their email.

## Installation

### Add Plugin to the server

```
import { betterAuth } from "better-auth"
import { username } from "better-auth/plugins"

export const auth = betterAuth({
    emailAndPassword: { 
        enabled: true, 
    }, 
    plugins: [ 
        username() 
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

```
import { createAuthClient } from "better-auth/client"
import { usernameClient } from "better-auth/client/plugins"

export const authClient = createAuthClient({
    plugins: [ 
        usernameClient() 
    ] 
})
```
