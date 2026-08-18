---
url: https://better-auth.com/llms.txt/docs/authentication/roblox
title: "Roblox"
description: ""
access_date: 2026-08-18T00:08:46.984Z
current_date: 2026-08-18T00:08:46.984Z
---

# Roblox

Roblox provider setup and usage.




### ### Get your Roblox Credentials
Get your Roblox credentials from the [Roblox Creator Hub](https://create.roblox.com/dashboard/credentials?activeTab=OAuthTab).

Make sure to set the redirect URL to `http://localhost:3000/api/auth/callback/roblox` for local development. For production, you should set it to the URL of your application. If you change the base path of the auth routes, you should update the redirect URL accordingly.

> The Roblox API does not provide email addresses. As a workaround, the user's `email` field uses the `preferred_username` value instead.

### ### Configure the provider
To configure the provider, you need to import the provider and pass it to the `socialProviders` option of the auth instance.

```ts title="auth.ts"  
import { betterAuth } from "better-auth" 

export const auth = betterAuth({
    socialProviders: {
        roblox: { // [!code highlight]
            clientId: process.env.ROBLOX_CLIENT_ID as string, // [!code highlight]
            clientSecret: process.env.ROBLOX_CLIENT_SECRET as string, // [!code highlight]
        }, // [!code highlight]
    },
})
```

### ### Sign In with Roblox
To sign in with Roblox, you can use the `signIn.social` function provided by the client. The `signIn` function takes an object with the following properties:

* `provider`: The provider to use. It should be set to `roblox`.

```ts title="auth-client.ts"  
import { createAuthClient } from "better-auth/client"
const authClient =  createAuthClient()

const signIn = async () => {
    const data = await authClient.signIn.social({
        provider: "roblox"
    })
}
```
