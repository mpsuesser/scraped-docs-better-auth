---
url: https://better-auth.com/llms.txt/docs/plugins/anonymous
title: "Anonymous"
description: ""
access_date: 2026-08-03T19:08:19.489Z
current_date: 2026-08-03T19:08:19.489Z
---

# Anonymous

Anonymous plugin for Better Auth.



The Anonymous plugin allows users to have an authenticated experience without requiring them to provide an email address, password, OAuth provider, or any other Personally Identifiable Information (PII). Users can later link an authentication method to their account when ready.

Installation [#installation]

<Steps>
  <Step>
    Add the plugin to your auth config [#add-the-plugin-to-your-auth-config]

    To enable anonymous authentication, add the anonymous plugin to your authentication configuration.

    ```ts title="auth.ts"
    import { betterAuth } from "better-auth"
    import { anonymous } from "better-auth/plugins" // [!code highlight]

    export const auth = betterAuth({
        // ... other config options
        plugins: [
            anonymous() // [!code highlight]
        ]
    })
    ```
  </Step>

  <Step>
    Migrate the database [#migrate-the-database]

    Run the migration or generate the schema to add the necessary fields and tables to the database.

    <Tabs items={["migrate", "generate"]}>
      <Tab value="migrate">
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
            npx auth migrate
            ```
          </CodeBlockTab>

          <CodeBlockTab value="pnpm">
            ```bash
            pnpm dlx auth migrate
            ```
          </CodeBlockTab>

          <CodeBlockTab value="yarn">
            ```bash
            yarn dlx auth migrate
            ```
          </CodeBlockTab>

          <CodeBlockTab value="bun">
            ```bash
            bun x auth migrate
            ```
          </CodeBlockTab>
        </CodeBlockTabs>
      </Tab>

      <Tab value="generate">
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
            npx auth generate
            ```
          </CodeBlockTab>

          <CodeBlockTab value="pnpm">
            ```bash
            pnpm dlx auth generate
            ```
          </CodeBlockTab>

          <CodeBlockTab value="yarn">
            ```bash
            yarn dlx auth generate
            ```
          </CodeBlockTab>

          <CodeBlockTab value="bun">
            ```bash
            bun x auth generate
            ```
          </CodeBlockTab>
        </CodeBlockTabs>
      </Tab>
    </Tabs>

    See the [Schema](#schema) section to add the fields manually.
  </Step>

  <Step>
    Add the client plugin [#add-the-client-plugin]

    Next, include the anonymous client plugin in your authentication client instance.

    ```ts title="auth-client.ts"
    import { createAuthClient } from "better-auth/client"
    import { anonymousClient } from "better-auth/client/plugins" // [!code highlight]

    export const authClient = createAuthClient({
        plugins: [
            anonymousClient() // [!code highlight]
        ]
    })
    ```
  </Step>
</Steps>

Usage [#usage]

Sign In [#sign-in]

To sign in a user anonymously, use the `signIn.anonymous()` method.

```ts
import { authClient } from "@/lib/auth-client";

const user = await authClient.signIn.anonymous()
```

Link Account [#link-account]

If a user is already signed in anonymously and tries to `signIn` or `signUp` with another method,
their anonymous activities can be linked to the new account.

To do that you first need to provide `onLinkAccount` callback to the plugin.

```ts title="auth.ts"
import { betterAuth } from "better-auth"
import { anonymous } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        anonymous({
            onLinkAccount: async ({ anonymousUser, newUser }) => {
               // perform actions like moving the cart items from anonymous user to the new user
            }
        })
    ]
```

Then when you call `signIn` or `signUp` with another method, the `onLinkAccount` callback will be called. And the `anonymousUser` will be deleted by default.

```ts
import { authClient } from "@/lib/auth-client";

const user = await authClient.signIn.email({
    email,
})
```

Delete Anonymous User [#delete-anonymous-user]

To delete an anonymous user, you can call the `/delete-anonymous-user` endpoint.


### Client Side

```ts
const { data, error } = await authClient.deleteAnonymousUser({});
```

### Server Side

```ts
await auth.api.deleteAnonymousUser({});
```

### Type Definition

```ts
type deleteAnonymousUser = {
  
}
```


<Callout type="info">
  **Notes:**

  * The anonymous user is deleted by default when the account is linked to a new authentication method.
  * Setting `disableDeleteAnonymousUser` to `true` will prevent the anonymous user from being able to call the `/delete-anonymous-user` endpoint.
</Callout>

Options [#options]

emailDomainName [#emaildomainname]

The domain name to use when generating an email address for anonymous users. If not provided, the default format `temp@{id}.com` is used.

```ts title="auth.ts"
import { betterAuth } from "better-auth"
import { anonymous } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        anonymous({
            emailDomainName: "example.com" // [!code highlight] -> temp-{id}@example.com
        })
    ]
})
```

generateRandomEmail [#generaterandomemail]

A custom function to generate email addresses for anonymous users. This allows you to define your own email format. The function can be synchronous or asynchronous.

```ts title="auth.ts"
import { betterAuth } from "better-auth"
import { anonymous } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        anonymous({
            generateRandomEmail: () => { // [!code highlight]
                const id = crypto.randomUUID() // [!code highlight]
                return `guest-${id}@example.com` // [!code highlight]
            } // [!code highlight]
        })
    ]
})
```

<Callout type="info">
  **Notes:**

  * If `generateRandomEmail` is provided, `emailDomainName` is ignored.
  * You are responsible for ensuring the email is unique to avoid conflicts. The returned email must be in a valid format.
</Callout>

onLinkAccount [#onlinkaccount]

A callback function that is called when an anonymous user links their account to a new authentication method. The callback receives an object with the `anonymousUser` and the `newUser`.

disableDeleteAnonymousUser [#disabledeleteanonymoususer]

By default, when an anonymous user links their account to a new authentication method,
the anonymous user record is automatically deleted.
If you set this option to `true`, this automatic deletion will be disabled,
and the `/delete-anonymous-user` endpoint will no longer be accessible to anonymous users.

generateName [#generatename]

A callback function that is called to generate a name for the anonymous user. Useful if you want to have random names for anonymous users, or if `name` is unique in your database.

Schema [#schema]

The anonymous plugin requires an additional field in the user table:

export const anonymousUserTableFields = [
	{
		name: "isAnonymous",
		type: "boolean",
		description: "Indicates whether the user is anonymous.",
		isOptional: true,
	},
];

<DatabaseTable name="user" fields={anonymousUserTableFields} />
