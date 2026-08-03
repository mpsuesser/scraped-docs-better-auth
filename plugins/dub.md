---
url: https://better-auth.com/llms.txt/docs/plugins/dub
title: "Dub"
description: ""
access_date: 2026-08-03T19:38:28.543Z
current_date: 2026-08-03T19:38:28.543Z
---

# Dub

Better Auth Plugin for Lead Tracking using Dub links and OAuth Linking



[Dub](https://dub.co/) is an open source modern link management platform for entrepreneurs, creators, and growth teams.

This plugins allows you to track leads when a user signs up using a Dub link. It also adds OAuth linking support to allow you to build integrations extending Dub's linking management infrastructure.

Installation [#installation]

<Steps>
  <Step>
    Install the plugin [#install-the-plugin]

    First, install the plugin:

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
        npm install @dub/better-auth
        ```
      </CodeBlockTab>

      <CodeBlockTab value="pnpm">
        ```bash
        pnpm add @dub/better-auth
        ```
      </CodeBlockTab>

      <CodeBlockTab value="yarn">
        ```bash
        yarn add @dub/better-auth
        ```
      </CodeBlockTab>

      <CodeBlockTab value="bun">
        ```bash
        bun add @dub/better-auth
        ```
      </CodeBlockTab>
    </CodeBlockTabs>
  </Step>

  <Step>
    Install the Dub SDK [#install-the-dub-sdk]

    Next, install the Dub SDK on your server:

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
        npm install dub
        ```
      </CodeBlockTab>

      <CodeBlockTab value="pnpm">
        ```bash
        pnpm add dub
        ```
      </CodeBlockTab>

      <CodeBlockTab value="yarn">
        ```bash
        yarn add dub
        ```
      </CodeBlockTab>

      <CodeBlockTab value="bun">
        ```bash
        bun add dub
        ```
      </CodeBlockTab>
    </CodeBlockTabs>
  </Step>

  <Step>
    Configure the plugin [#configure-the-plugin]

    Add the plugin to your auth config:

    ```ts title="auth.ts"
    import { betterAuth } from "better-auth"
    import { dubAnalytics } from "@dub/better-auth"
    import { dub } from "dub"

    export const auth = betterAuth({
        plugins: [
            dubAnalytics({
                dubClient: new Dub()
            })
        ]
    })
    ```
  </Step>
</Steps>

Usage [#usage]

Lead Tracking [#lead-tracking]

By default, the plugin will track sign up events as leads. You can disable this by setting `disableLeadTracking` to `true`.

```ts
import { dubAnalytics } from "@dub/better-auth";
import { betterAuth } from "better-auth";
import { Dub } from "dub";

const dub = new Dub();

const betterAuth = betterAuth({
  plugins: [
    dubAnalytics({
      dubClient: dub,
      disableLeadTracking: true, // Disable lead tracking
    }),
  ],
});
```

OAuth Linking [#oauth-linking]

The plugin supports OAuth for account linking.

First, you need to setup OAuth app in Dub. Dub supports OAuth 2.0 authentication, which is recommended if you build integrations extending Dub’s functionality [Learn more about OAuth](https://dub.co/docs/integrations/quickstart#integrating-via-oauth-2-0-recommended).

Once you get the client ID and client secret, you can configure the plugin.

```ts
dubAnalytics({
  dubClient: dub,
  oauth: {
    clientId: "your-client-id",
    clientSecret: "your-client-secret",
  },
});
```

And in the client, you need to use the `dubAnalyticsClient` plugin.

```ts
import { createAuthClient } from "better-auth/client";
import { dubAnalyticsClient } from "@dub/better-auth/client";

const authClient = createAuthClient({
  plugins: [dubAnalyticsClient()],
});
```

To link account with Dub, you need to use the `dub.link`.


### Client Side

```ts
const { data, error } = await authClient.dub.link({
    callbackURL: /dashboard,
});
```

### Server Side

```ts
const data = await auth.api.dubLink({
    body: {
        callbackURL: /dashboard,
    },
    // This endpoint requires session cookies.
    headers: await headers()
});
```

### Type Definition

```ts
type dubLink = {
    /**
     * URL to redirect to after linking
     * @clientOnly
    */
    callbackURL: string = "/dashboard"
  
}
```


Options [#options]

You can pass the following options to the plugin:

dubClient [#dubclient]

The Dub client instance.

disableLeadTracking [#disableleadtracking]

Disable lead tracking for sign up events.

leadEventName [#leadeventname]

Event name for sign up leads.

customLeadTrack [#customleadtrack]

Custom lead track function.

oauth [#oauth]

Dub OAuth configuration.

oauth.clientId [#oauthclientid]

Client ID for Dub OAuth.

oauth.clientSecret [#oauthclientsecret]

Client secret for Dub OAuth.

oauth.pkce [#oauthpkce]

Enable PKCE for Dub OAuth.
