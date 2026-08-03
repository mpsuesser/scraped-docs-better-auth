---
url: https://better-auth.com/llms.txt/docs/infrastructure/getting-started
title: "Getting Started"
description: ""
access_date: 2026-08-03T18:54:22.481Z
current_date: 2026-08-03T18:54:22.481Z
---

# Getting Started

This guide will help you integrate Better Auth Infrastructure into your application.



<Steps>
  <Step>
    Prerequisites [#prerequisites]

    Before you begin, make sure you have:

    1. A working [Better Auth](/docs/installation) installation
    2. An account & API Key from the [Better Auth Infrastructure](/dashboard) dashboard
  </Step>

  <Step>
    Installation [#installation]

    Install the `@better-auth/infra` package:

    <CodeBlockTabs defaultValue="npm" groupId="persist-install" persist>
      <CodeBlockTabsList>
        <CodeBlockTabsTrigger value="npm">
          npm
        </CodeBlockTabsTrigger>

        <CodeBlockTabsTrigger value="pnpm">
          pnpm
        </CodeBlockTabsTrigger>

        <CodeBlockTabsTrigger value="yarn">
          yarn
        </CodeBlockTabsTrigger>

        <CodeBlockTabsTrigger value="bun">
          bun
        </CodeBlockTabsTrigger>
      </CodeBlockTabsList>

      <CodeBlockTab value="npm">
        ```bash
        npm install @better-auth/infra
        ```
      </CodeBlockTab>

      <CodeBlockTab value="pnpm">
        ```bash
        pnpm add @better-auth/infra
        ```
      </CodeBlockTab>

      <CodeBlockTab value="yarn">
        ```bash
        yarn add @better-auth/infra
        ```
      </CodeBlockTab>

      <CodeBlockTab value="bun">
        ```bash
        bun add @better-auth/infra
        ```
      </CodeBlockTab>
    </CodeBlockTabs>
  </Step>

  <Step>
    Environment Variables [#environment-variables]

    Once you've gotten the API Key from the infrastructure dashboard,
    go ahead and add the `BETTER_AUTH_API_KEY` environment variable in your production environment variables.

    ```dotenv
    # Required: Your Better Auth Infrastructure API key
    BETTER_AUTH_API_KEY=your_api_key_here
    ```

    <Callout>
      You can get your API key by signing-up and creating a new project in the
      [Better Auth Infrastructure dashboard](/dashboard).
    </Callout>
  </Step>
</Steps>

Server Setup [#server-setup]

Basic Configuration [#basic-configuration]

Add the `dash()` plugin to your Better Auth configuration:

```ts
import { betterAuth } from "better-auth";
import { dash } from "@better-auth/infra"; // [!code highlight]

export const auth = betterAuth({
  // ... your existing Better Auth config
  plugins: [
    dash({ // [!code highlight]
      apiKey: process.env.BETTER_AUTH_API_KEY, // [!code highlight]
    }), // [!code highlight]
  ],
});
```

If you're on **pro** plan or above, you can use the `sentinel()` plugin to enable security checks, abuse protection, and more:

```ts
import { betterAuth } from "better-auth";
import { dash, sentinel } from "@better-auth/infra";  // [!code highlight]

export const auth = betterAuth({
  plugins: [
    dash({ /* ... */ }),
    sentinel({ // [!code highlight]
      apiKey: process.env.BETTER_AUTH_API_KEY, // [!code highlight]
    }) // [!code highlight]
  ],
});
```

Client Setup [#client-setup]

Basic Client Configuration [#basic-client-configuration]

Add the client plugins to your auth client:

```ts
import { createAuthClient } from "better-auth/client";
import { dashClient, sentinelClient } from "@better-auth/infra/client";

export const authClient = createAuthClient({
  plugins: [
    dashClient(),
    sentinelClient({
      autoSolveChallenge: true, // Automatically solve PoW challenges
    }),
  ],
});
```

Expo and React Native [#expo-and-react-native]

For **Expo** or **React Native** clients, import from `@better-auth/infra/native` instead of `@better-auth/infra/client`. The native entry provides `dashClient` (same audit log APIs as the web client) and `sentinelNativeClient`.

```ts
import { createAuthClient } from "better-auth/client";
import { dashClient, sentinelNativeClient } from "@better-auth/infra/native";

export const authClient = createAuthClient({
  baseURL: "https://your-api.example.com",
  plugins: [
    dashClient(),
    sentinelNativeClient({
      autoSolveChallenge: true,
    }),
  ],
});
```

See [Sentinel — Expo and React Native](/docs/infrastructure/plugins/sentinel#expo-and-react-native) for more details.

Plugin Overview [#plugin-overview]

* **`dash()`** - Enables analytics tracking, audit logging, dashboard admin APIs, and more
* **`sentinel()`** - Enables security checks and abuse protection

You can use either plugin independently, but using both together provides the full Better Auth Infrastructure experience.

Next Steps [#next-steps]

Now that you have the basic setup, explore these topics:

* [Dashboard](/docs/infrastructure/plugins/dashboard) - Analytics, activity tracking, and admin APIs
* [Audit Logs](/docs/infrastructure/plugins/audit-logs) - Track and query authentication events
* [Sentinel](/docs/infrastructure/plugins/sentinel) - Configure advanced security
* [Email Service](/docs/infrastructure/services/email) - Set up transactional emails
* [SMS Service](/docs/infrastructure/services/sms) - Enable SMS verification
