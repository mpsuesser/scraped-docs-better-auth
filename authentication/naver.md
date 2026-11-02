---
url: https://better-auth.com/llms.txt/docs/authentication/naver
title: "Naver"
description: ""
access_date: 2026-11-02T21:35:49.252Z
current_date: 2026-11-02T21:35:49.252Z
---

### Configure the provider

To configure the provider, you need to import the provider and pass it to the `socialProviders` option of the auth instance.

```
import { betterAuth } from "better-auth"

export const auth = betterAuth({
    socialProviders: {
        naver: { 
            clientId: process.env.NAVER_CLIENT_ID as string, 
            clientSecret: process.env.NAVER_CLIENT_SECRET as string, 
        }, 
    }
})
```

### Sign In with Naver

To sign in with Naver, you can use the `signIn.social` function provided by the client. The `signIn` function takes an object with the following properties:

- `provider`: The provider to use. It should be set to `naver`.

```
import { createAuthClient } from "better-auth/client"
const authClient =  createAuthClient()

const signIn = async () => {
    const data = await authClient.signIn.social({
        provider: "naver"
    })
}
```
