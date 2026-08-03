---
url: https://better-auth.com/llms.txt/docs/authentication/vk
title: "Vk"
description: ""
access_date: 2026-08-03T18:54:22.481Z
current_date: 2026-08-03T18:54:22.481Z
---

# VK

VK ID Provider



<Steps>
  <Step>
    Get your VK ID credentials [#get-your-vk-id-credentials]

    To use VK ID sign in, you need a client ID and client secret. You can get them from the [VK ID Developer Portal](https://id.vk.com/about/business/go/docs).

    Make sure to set the redirect URL to `http://localhost:3000/api/auth/callback/vk` for local development. For production, you should set it to the URL of your application. If you change the base path of the auth routes, you should update the redirect URL accordingly.
  </Step>

  <Step>
    Configure the provider [#configure-the-provider]

    To configure the provider, you need to import the provider and pass it to the `socialProviders` option of the auth instance.

    ```ts title="auth.ts"
    import { betterAuth } from "better-auth";

    export const auth = betterAuth({
      socialProviders: {
        vk: { // [!code highlight]
          clientId: process.env.VK_CLIENT_ID as string, // [!code highlight]
          clientSecret: process.env.VK_CLIENT_SECRET as string, // [!code highlight]
        },
      },
    });
    ```
  </Step>

  <Step>
    Sign In with VK [#sign-in-with-vk]

    To sign in with VK, you can use the `signIn.social` function provided by the client. The `signIn` function takes an object with the following properties:

    * `provider`: The provider to use. It should be set to `vk`.

    ```ts title="auth-client.ts"
    import { createAuthClient } from "better-auth/client";
    const authClient = createAuthClient();

    const signIn = async () => {
      const data = await authClient.signIn.social({
        provider: "vk",
      });
    };
    ```
  </Step>
</Steps>
