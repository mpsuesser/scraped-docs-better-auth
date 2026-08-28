---
url: https://better-auth.com/llms.txt/docs/authentication/twitter
title: "Twitter"
description: ""
access_date: 2026-08-28T22:16:12.077Z
current_date: 2026-08-28T22:16:12.077Z
---

# Twitter (X) (/docs/authentication/twitter)

Twitter provider setup and usage.




### ### Get your Twitter Credentials
Get your Twitter credentials from the [Twitter Developer Portal](https://developer.twitter.com/en/portal/dashboard).

Make sure to set the redirect URL to `http://localhost:3000/api/auth/callback/twitter` for local development. For production, you should set it to the URL of your application. If you change the base path of the auth routes, you should update the redirect URL accordingly.

> Twitter API v2 now supports email address retrieval. Make sure to request the `user.email` scope when configuring your Twitter app to enable this feature.

### ### Configure the provider
To configure the provider, you need to import the provider and pass it to the `socialProviders` option of the auth instance.

```ts title="auth.ts"  
import { betterAuth } from "better-auth" 

export const auth = betterAuth({
    socialProviders: {
        twitter: { // [!code highlight]
            clientId: process.env.TWITTER_CLIENT_ID as string, // [!code highlight]
            clientSecret: process.env.TWITTER_CLIENT_SECRET as string, // [!code highlight]
        }, // [!code highlight]
    },
})
```

### ### Sign In with Twitter
To sign in with Twitter, you can use the `signIn.social` function provided by the client. The `signIn` function takes an object with the following properties:

* `provider`: The provider to use. It should be set to `twitter`.

```ts title="auth-client.ts"  
import { createAuthClient } from "better-auth/client"
const authClient =  createAuthClient()

const signIn = async () => {
    const data = await authClient.signIn.social({
        provider: "twitter"
    })
}
```
