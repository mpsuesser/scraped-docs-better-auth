---
url: https://better-auth.com/llms.txt/docs/authentication/linkedin
title: "Linkedin"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

# LinkedIn

LinkedIn Provider




### Get your LinkedIn credentials
To use LinkedIn sign in, you need a client ID and client secret. You can get them from the [LinkedIn Developer Portal](https://www.linkedin.com/developers/).

Make sure to set the redirect URL to `http://localhost:3000/api/auth/callback/linkedin` for local development. For production, you should set it to the URL of your application. If you change the base path of the auth routes, you should update the redirect URL accordingly.

> In the LinkedIn portal under products you need the **Sign In with LinkedIn using OpenID Connect** product.

  There are some different Guides here:
  [Authorization Code Flow (3-legged OAuth) (Outdated)](https://learn.microsoft.com/en-us/linkedin/shared/authentication/authorization-code-flow)
  [Sign In with LinkedIn using OpenID Connect](https://learn.microsoft.com/en-us/linkedin/consumer/integrations/self-serve/sign-in-with-linkedin-v2?context=linkedin%2Fconsumer%2Fcontext)

### Configure the provider
To configure the provider, you need to import the provider and pass it to the `socialProviders` option of the auth instance.

```ts title="auth.ts"
import { betterAuth } from "better-auth"

export const auth = betterAuth({
    socialProviders: {
        linkedin: { // [!code highlight]
            clientId: process.env.LINKEDIN_CLIENT_ID as string, // [!code highlight]
            clientSecret: process.env.LINKEDIN_CLIENT_SECRET as string, // [!code highlight]
        }, // [!code highlight]
    },
})
```

> LinkedIn's `email` and `email_verified` claims are [documented as optional](https://learn.microsoft.com/en-us/linkedin/consumer/integrations/self-serve/sign-in-with-linkedin-v2) and may be absent when no confirmed email is associated with the member. See [Handling Providers Without Email](/docs/concepts/oauth#handling-providers-without-email) for the recommended `mapProfileToUser` fallback.

### Sign In with LinkedIn
To sign in with LinkedIn, you can use the `signIn.social` function provided by the client. The `signIn` function takes an object with the following properties:

* `provider`: The provider to use. It should be set to `linkedin`.

```ts title="auth-client.ts"
import { createAuthClient } from "better-auth/client"
const authClient =  createAuthClient()

const signIn = async () => {
    const data = await authClient.signIn.social({
        provider: "linkedin"
    })
}
```
