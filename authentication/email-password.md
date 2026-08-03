---
url: https://better-auth.com/llms.txt/docs/authentication/email-password
title: "Email Password"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Implementing email and password authentication with Better Auth.

Email and password authentication is a common method used by many applications. Better Auth provides a built-in email and password authenticator that you can easily integrate into your project.

## Enable Email and Password

To enable email and password authentication, you need to set the `emailAndPassword.enabled` option to `true` in the `auth` configuration.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  emailAndPassword: { 
    enabled: true, 
  }, 
});
```

Additionally, to use the following server methods like `signUpEmail`, cookies must also be passed back to the client. This may require additional configuration. Plugins are provided for [Next](https://better-auth.com/docs/integrations/next#server-action-cookies) and [SvelteKit](https://better-auth.com/docs/integrations/svelte-kit#server-action-cookies).
