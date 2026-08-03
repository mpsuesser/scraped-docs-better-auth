---
url: https://better-auth.com/llms.txt/docs/authentication/wechat
title: "Wechat"
description: ""
access_date: 2026-08-03T19:38:28.543Z
current_date: 2026-08-03T19:38:28.543Z
---

# WeChat

WeChat provider setup and usage.



<Steps>
  <Step>
    Get your WeChat Credentials [#get-your-wechat-credentials]

    To use WeChat sign in, you need to register a website application on the [WeChat Open Platform](https://open.weixin.qq.com/), set the `Authorization Callback Domain` to the better auth domain and get your App ID and App Secret.
  </Step>

  <Step>
    Configure the provider [#configure-the-provider]

    To configure the provider, you need to import the provider and pass it to the `socialProviders` option of the auth instance.

    ```ts title="auth.ts"
    import { betterAuth } from "better-auth"

    export const auth = betterAuth({
        socialProviders: {
            wechat: { // [!code highlight]
                clientId: process.env.WECHAT_CLIENT_ID, // [!code highlight]
                clientSecret: process.env.WECHAT_CLIENT_SECRET, // [!code highlight]
            }, // [!code highlight]
        },
    })
    ```

    Optional Configuration [#optional-configuration]

    You can customize the WeChat provider with additional options:

    ```ts title="auth.ts"
    export const auth = betterAuth({
        socialProviders: {
            wechat: {
                clientId: process.env.WECHAT_CLIENT_ID,
                clientSecret: process.env.WECHAT_CLIENT_SECRET,
                // Optional: Set UI language for the WeChat login page
                lang: "cn", // or "en" for English
                // Optional: Use custom scopes
                scope: [], // "snsapi_login" for web QR code login.
            },
        },
    })
    ```
  </Step>

  <Step>
    Sign In with WeChat [#sign-in-with-wechat]

    To sign in with WeChat, you can use the `signIn.social` function provided by the client. The `signIn` function takes an object with the following properties:

    * `provider`: The provider to use. It should be set to `wechat`.

    ```ts title="auth-client.ts"
    import { createAuthClient } from "better-auth/client"
    const authClient = createAuthClient()

    const signIn = async () => {
        const data = await authClient.signIn.social({
            provider: "wechat"
        })
    }
    ```
  </Step>
</Steps>

Usage [#usage]

Platform Support [#platform-support]

WeChat provider currently supports the Website Application platform type, which enables WeChat QR code login for web applications.

Development Notes [#development-notes]

* The redirect URL domain must match the domain configured in the WeChat Open Platform
