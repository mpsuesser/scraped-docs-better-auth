---
url: https://better-auth.com/llms.txt/docs/plugins/scim
title: "Scim"
description: ""
access_date: 2026-08-03T19:00:28.246Z
current_date: 2026-08-03T19:00:28.246Z
---

# System for Cross-domain Identity Management (SCIM)

Integrate SCIM with your application.



System for Cross-domain Identity Management ([SCIM](https://simplecloud.info/#Specification)) makes managing identities in multi-domain scenarios easier to support via a standardized protocol.
This plugin exposes a [SCIM](https://simplecloud.info/#Specification) server that allows third party identity providers to sync identities to your service.

<Callout>
  Need a self-service SCIM setup where your customers can configure their own identity provider sync? [Contact us for enterprise](/enterprise).
</Callout>

Installation [#installation]

<Steps>
  <Step>
    Install the plugin [#install-the-plugin]

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
        npm install @better-auth/scim
        ```
      </CodeBlockTab>

      <CodeBlockTab value="pnpm">
        ```bash
        pnpm add @better-auth/scim
        ```
      </CodeBlockTab>

      <CodeBlockTab value="yarn">
        ```bash
        yarn add @better-auth/scim
        ```
      </CodeBlockTab>

      <CodeBlockTab value="bun">
        ```bash
        bun add @better-auth/scim
        ```
      </CodeBlockTab>
    </CodeBlockTabs>
  </Step>

  <Step>
    Add Plugin to the server [#add-plugin-to-the-server]

    ```ts title="auth.ts"
    import { betterAuth } from "better-auth"
    import { scim } from "@better-auth/scim"; // [!code highlight]

    const auth = betterAuth({
        plugins: [
            scim() // [!code highlight]
        ]
    })
    ```
  </Step>

  <Step>
    Enable HTTP methods [#enable-http-methods]

    SCIM requires the `POST`, `GET`, `PUT`, `PATCH` and `DELETE` HTTP methods to be supported by your server.
    For most frameworks, this will work out of the box, but some frameworks may require additional configuration:

    <Tabs items={["Next.js", "Solid Start"]}>
      <Tab value="Next.js">
        ```ts title="api/auth/[...all]/route.ts"
        import { auth } from "@/lib/auth";
        import { toNextJsHandler } from "better-auth/next-js";

        export const { POST, GET, PUT, PATCH, DELETE } = toNextJsHandler(auth); // [!code highlight]
        ```
      </Tab>

      <Tab value="Solid Start">
        ```ts title="routes/api/auth/*auth.ts"
        import { auth } from "~/lib/auth";
        import { toSolidStartHandler } from "better-auth/solid-start";

        export const { GET, POST, PUT, PATCH, DELETE } = toSolidStartHandler(auth); // [!code highlight]
        ```
      </Tab>
    </Tabs>
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
</Steps>

Usage [#usage]

Upon registration, this plugin will expose compliant [SCIM 2.0](https://simplecloud.info/#Specification) server. Generally, this server is meant to be consumed by a third-party (your identity provider), and will require a:

* **SCIM base URL**: This should be the fully qualified URL to the SCIM server (e.g `http://your-app.com/api/auth/scim/v2`)
* **SCIM bearer token**: See [generating a SCIM token](#generating-a-scim-token)

Self-Service Directory Sync [#self-service-directory-sync]

If you're using [Better Auth Infrastructure](https://dash.better-auth.com/sign-in), you get self-service directory sync in the dashboard. Organization admins can create and manage SCIM directory connections and rotate bearer tokens without calling the SCIM APIs directly.

The dashboard is available at:

```
https://dash.better-auth.com/[project]/organization/[orgId]/enterprise
```

From the dashboard you can:

* **Create and remove directory connections** scoped to an organization
* **Regenerate SCIM bearer tokens** when your identity provider requires rotation

This eliminates the back-and-forth typically required when setting up SCIM, reducing onboarding time from days to minutes.

Generating a SCIM token [#generating-a-scim-token]

Before your identity provider can start syncing information to your SCIM server,
you need to generate a SCIM token that your identity provider will use to authenticate against it.

A SCIM token is a simple bearer token that you can generate:


### Client Side

```ts
const { data, error } = await authClient.scim.generateToken({
    providerId: acme-corp,
    organizationId: the-org, // optional
});
```

### Server Side

```ts
const data = await auth.api.generateSCIMToken({
    body: {
        providerId: acme-corp,
        organizationId: the-org, // optional
    },
    // This endpoint requires session cookies.
    headers: await headers()
});
```

### Type Definition

```ts
type generateSCIMToken = {
    /**
    * The provider id
    */
    providerId: string = "acme-corp"
    /**
     * Optional organization id. When specified, the organizations plugin must also be enabled
    */
    organizationId?: string = "the-org"
  
}
```


A `SCIM` token is always restricted to a provider, thus you are required to specify a `providerId`. This can be any provider your instance supports (e.g one of the built-in providers such as `credentials` or an external provider registered through an external plugin such as `@better-auth/sso`).
Additionally, when the `organization` plugin is registered, you can optionally restrict the token to an organization via the `organizationId`.

<Callout>
  **Important:** Personal SCIM connections can still be generated by any authenticated user. Organization-scoped connections are restricted by default to users with the `admin` role or the organization creator role (`organization.creatorRole`, which defaults to `owner`). If you need a different policy, configure [`requiredRole`](#options) and/or add stricter checks in [hooks](#hooks).
</Callout>

Organization-scoped authorization [#organization-scoped-authorization]

When `organizationId` is provided, Better Auth requires the current user to be a member of that organization and to have at least one of the configured `requiredRole` values.

By default, `requiredRole` resolves to:

* `admin`
* `organization.creatorRole` or `owner`

The same role requirement is also used by the SCIM management endpoints for organization-scoped connections:

* `GET /scim/list-provider-connections`
* `GET /scim/get-provider-connection`
* `POST /scim/delete-provider-connection`

```ts
const approvedScimOperators = new Set(["some-admin-user-id"]);

scim({
    beforeSCIMTokenGenerated: async ({ user }) => {
        // Add stricter rules on top of the built-in organization role checks.
        if (!approvedScimOperators.has(user.id)) {
            throw new APIError("FORBIDDEN", { message: "User does not have enough permissions" });
        }
    },
})
```

See the [hooks](#hooks) documentation for more details about supported hooks.

Default SCIM token [#default-scim-token]

We also provide a way for you to specify a `SCIM` token to use by default. This allows you to test a SCIM connection without setting up providers in the database:

```ts title="auth.ts"
import { betterAuth } from "better-auth"
import { scim } from "@better-auth/scim"; 

const auth = betterAuth({
    plugins: [
        scim({
            defaultSCIM: [
                {
                    providerId: "default-scim", // ID of the existing provider you want to provision
                    scimToken: "some-scim-token", // SCIM plain token
                    organizationId: "the-org" // Optional organization id
                }
            ]
        })
    ]
});
```

<Callout type="info">
  **Important**: Please note that you must base64 encode your `scimToken` before you try to use as follows: `base64(scimToken:providerId[:organizationId])`.

  In our example above, you would need to encode the `some-scim-token:default-scim:the-org` text to base64, resulting in the following scimToken: `c29tZS1zY2ltLXRva2VuOmRlZmF1bHQtc2NpbTp0aGUtb3Jn`
</Callout>

SCIM provider connection ownership [#scim-provider-connection-ownership]

SCIM provider connection ownership applies to personal (non-organization) SCIM connections. It lets your application track who generated a connection and restricts later management operations for that connection to the same user.

```ts title="auth.ts"
import { betterAuth } from "better-auth";
import { scim } from "@better-auth/scim";

const auth = betterAuth({
    plugins: [
        scim({ // [!code highlight]
            providerOwnership: { // [!code highlight]
                enabled: true // [!code highlight]
            } // [!code highlight]
        }) // [!code highlight]
    ]
});
```

When enabled:

* Personal connections store the creating user's `userId`
* Only the owner can regenerate, list, inspect, or delete those personal connections later
* Organization-scoped connections continue to use the organization role checks configured by `requiredRole`

Once enabled, make sure you migrate the database schema (again).

<Tabs items={["migrate", "generate"]}>
  <Tab value="migrate">
    ```bash
    npx auth migrate
    ```
  </Tab>

  <Tab value="generate">
    ```bash
    npx auth generate
    ```
  </Tab>
</Tabs>

See the [Schema](#if-you-have-provider-ownership-enabled-via-providerownershipenabled) section to add the fields manually.

Managing SCIM provider connections [#managing-scim-provider-connections]

You can manage SCIM provider connections from your application using the following endpoints:

List SCIM provider connections [#list-scim-provider-connections]

List existing connections the current user can manage. For organization-scoped connections, the user must have one of the configured `requiredRole` roles for that organization. For personal connections, access is based on ownership when `providerOwnership.enabled` is turned on.


### Client Side

```ts
const { data, error } = await authClient.scim.listProviderConnections({});
```

### Server Side

```ts
const data = await auth.api.listSCIMProviderConnections({

    // This endpoint requires session cookies.
    headers: await headers()
});
```

### Type Definition

```ts
type listSCIMProviderConnections = {
  
}
```


Get SCIM provider connection details [#get-scim-provider-connection-details]

Get a single connection by provider id. Access is allowed only if the user can manage that connection: either because they satisfy the configured organization role requirement, or because they own the personal connection.


### Client Side

```ts
const { data, error } = await authClient.scim.getProviderConnection({
    providerId: acme-corp,
});
```

### Server Side

```ts
const data = await auth.api.getSCIMProviderConnection({
    query: {
        providerId: acme-corp,
    },
    // This endpoint requires session cookies.
    headers: await headers()
});
```

### Type Definition

```ts
type getSCIMProviderConnection = {
    /**
     * Unique provider identifier
     */
    providerId: string = "acme-corp"
  
}
```


Delete SCIM provider connection [#delete-scim-provider-connection]

Delete an existing connection. This will immediately invalidate the connection's associated token.


### Client Side

```ts
const { data, error } = await authClient.scim.deleteProviderConnection({
    providerId: acme-corp,
});
```

### Server Side

```ts
const data = await auth.api.deleteSCIMProviderConnection({
    body: {
        providerId: acme-corp,
    },
    // This endpoint requires session cookies.
    headers: await headers()
});
```

### Type Definition

```ts
type deleteSCIMProviderConnection = {
    /**
     * Unique provider identifier
     */
    providerId: string = "acme-corp"
  
}
```


SCIM endpoints [#scim-endpoints]

The following subset of the specification is currently supported:

List users [#list-users]

Get a list of available users in the database. This is restricted to list only users associated to the same provider and organization than your SCIM token.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.users({
    filter, // optional
});
```

### Server Side

```ts
const data = await auth.api.listSCIMUsers({
    query: {
        filter, // optional
    }
});
```

### Type Definition

```ts
type listSCIMUsers = {
      /**
       * SCIM compliant filter expression
      */
      filter?: string = 'userName eq "user-a"'
  
}
```


Get user [#get-user]

Get an user from the database. The user will be only returned if it belongs to the same provider and organization than the SCIM token.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.users.:userid({
    userId: user id,
});
```

### Server Side

```ts
const data = await auth.api.getSCIMUser({
    query: {
        userId: user id,
    }
});
```

### Type Definition

```ts
type getSCIMUser = {
      /**
       * Unique user identifier
      */
      userId: string = "user id"
  
}
```


Create new user [#create-new-user]

Provisions a new user to the database. The user will have an account associated to the same provider and will be member of the same org than the SCIM token.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.users({
    externalId: third party id, // optional
    name, // optional
    formatted: Daniel Perez, // optional
    givenName: Daniel, // optional
    familyName: Perez, // optional
});
```

### Server Side

```ts
const data = await auth.api.createSCIMUser({
    body: {
        externalId: third party id, // optional
        name, // optional
        formatted: Daniel Perez, // optional
        givenName: Daniel, // optional
        familyName: Perez, // optional
    }
});
```

### Type Definition

```ts
type createSCIMUser = {
      /*
       * Unique external (third party) identifier
      */
      externalId?: string = "third party id"
      /**
       * User name details
      */
      name?: {
          /**
           * Formatted name (takes priority over given and family name)
          */
          formatted?: string = "Daniel Perez"
          /**
           * Given name
          */
          givenName?: string = "Daniel"
          /**
           * Family name
          */
          familyName?: string = "Perez"
      
}
```


Update an existing user [#update-an-existing-user]

Replaces an existing user details in the database. This operation can only update users that belong to the same provider and organization than the SCIM token.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.users.:userid({
    externalId: third party id, // optional
    name, // optional
    formatted: Daniel Perez, // optional
    givenName: Daniel, // optional
    familyName: Perez, // optional
});
```

### Server Side

```ts
const data = await auth.api.updateSCIMUser({
    query: {
        externalId: third party id, // optional
        name, // optional
        formatted: Daniel Perez, // optional
        givenName: Daniel, // optional
        familyName: Perez, // optional
    }
});
```

### Type Definition

```ts
type updateSCIMUser = {
      /*
       * Unique external (third party) identifier
      */
      externalId?: string = "third party id"
      /**
       * User name details
      */
      name?: {
          /**
           * Formatted name (takes priority over given and family name)
          */
          formatted?: string = "Daniel Perez"
          /**
           * Given name
          */
          givenName?: string = "Daniel"
          /**
           * Family name
          */
          familyName?: string = "Perez"
      
}
```


Partial update an existing user [#partial-update-an-existing-user]

Allows to apply a partial update to the user details. This operation can only update users that belong to the same provider and organization than the SCIM token.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.users.:userid({
    schemas,
    Operations,
});
```

### Server Side

```ts
const data = await auth.api.patchSCIMUser({
    query: {
        schemas,
        Operations,
    }
});
```

### Type Definition

```ts
type patchSCIMUser = {
      /**
       * Mandatory schema declaration
      */
      schemas: string[] = ["urn:ietf:params:scim:api:messages:2.0:PatchOp"]
      /**
       * List of JSON patch operations
      */
      Operations: Array<{ op: "replace" | "add" | "remove", path?: string, value: any 
}
```


Deletes a user resource [#deletes-a-user-resource]

Removes a user resource. This operation only affects users that belong to the same provider (and organization) as the SCIM token.

For an organization-scoped token, the user is deprovisioned from the organization: their membership and the provider account are removed, while the global user record is kept. For a non-organization token, the global user is deleted only when this provider's account is their sole identity; otherwise just that account is unlinked.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.users.:userid({
    userId,
});
```

### Server Side

```ts
const data = await auth.api.deleteSCIMUser({
    query: {
        userId,
    }
});
```

### Type Definition

```ts
type deleteSCIMUser = {
      userId: string
  
}
```


Get service provider config [#get-service-provider-config]

Get SCIM metadata describing supported features of this server.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.serviceproviderconfig({});
```

### Server Side

```ts
const data = await auth.api.getSCIMServiceProviderConfig({});
```

### Type Definition

```ts
type getSCIMServiceProviderConfig = {
  
}
```


Get SCIM schemas [#get-scim-schemas]

Get the list of supported SCIM schemas.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.schemas({});
```

### Server Side

```ts
const data = await auth.api.getSCIMSchemas({});
```

### Type Definition

```ts
type getSCIMSchemas = {
  
}
```


Get SCIM schema [#get-scim-schema]

Get the details of a supported SCIM schema.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.schemas.:schemaid({});
```

### Server Side

```ts
const data = await auth.api.getSCIMSchema({});
```

### Type Definition

```ts
type getSCIMSchema = {
  
}
```


Get SCIM resource types [#get-scim-resource-types]

Get the list of supported SCIM types.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.resourcetypes({});
```

### Server Side

```ts
const data = await auth.api.getSCIMResourceTypes({});
```

### Type Definition

```ts
type getSCIMResourceTypes = {
  
}
```


Get SCIM resource type [#get-scim-resource-type]

Get the details of a supported SCIM resource type.


### Client Side

```ts
const { data, error } = await authClient.scim.v2.resourcetypes.:resourcetypeid({});
```

### Server Side

```ts
const data = await auth.api.getSCIMResourceType({});
```

### Type Definition

```ts
type getSCIMResourceType = {
  
}
```


SCIM attribute mapping [#scim-attribute-mapping]

By default, the SCIM provisioning will automatically map the following fields:

* `user.email`: User primary email or the first available email if there is not a primary one
* `user.name`: Derived from `name` (`name.formatted` or `name.givenName` + `name.familyName`) and fallbacks to the user primary email
* `account.providerId`: Provider associated to the `SCIM` token
* `account.accountId`: Defaults to `externalId` and fallbacks to `userName`
* `member.organizationId`: Organization associated to the provider

The SCIM `active` attribute maps to the user's disabled state. `active: false` deactivates the user (via the [admin](/docs/plugins/admin) plugin's `banned` state) and revokes their sessions; `active: true` reactivates. Honoring `active` requires the admin plugin. Changing a user's email through SCIM also resets their verified status.

Schema [#schema]

The plugin requires additional fields in the `scimProvider` table to store the provider's configuration.

export const scimProviderTableFields = [
	{
		name: "id",
		type: "string",
		description: "A database identifier",
		isPrimaryKey: true,
	},
	{
		name: "providerId",
		type: "string",
		description:
			"The provider ID. Used to identify a provider and to generate a redirect URL.",
		isUnique: true,
	},
	{
		name: "scimToken",
		type: "string",
		description:
			"The SCIM bearer token. Used by your identity provider to authenticate against your server",
		isUnique: true,
	},
	{
		name: "organizationId",
		type: "string",
		description:
			"The organization Id. If provider is linked to an organization.",
		isOptional: true,
	},
];

<DatabaseTable name="scimProvider" fields={scimProviderTableFields} />

If you have provider ownership enabled via providerOwnership.enabled: [#if-you-have-provider-ownership-enabled-via-providerownershipenabled]

The `scimProvider` schema is extended as follows:

export const scimProviderOwnershipFields = [
	{
		name: "userId",
		type: "string",
		description:
			"The user id of the connection owner. Set automatically when generating a token via the API.",
		isOptional: true,
	},
];

<DatabaseTable name="scimProvider" fields={scimProviderOwnershipFields} />

Options [#options]

Server [#server]

* `requiredRole`: `string[]` — Minimum organization role(s) allowed to generate organization-scoped tokens and manage organization-scoped connections. Defaults to `["admin", organization.creatorRole ?? "owner"]`.

```ts title="Allow only owners to manage organization-scoped SCIM connections"
scim({
    requiredRole: ["owner"],
})
```

* `providerOwnership`: `{ enabled: boolean }` — When enabled, links each personal provider connection to the user who generated its token. See [Connection ownership](#scim-provider-connection-ownership) for details. Default is `{ enabled: false }`.

```ts title="Enable connection ownership (requires migration)"
scim({
    providerOwnership: { enabled: true },
})
```

* `defaultSCIM`: Default list of SCIM tokens for testing.
* `storeSCIMToken`: The method to store the SCIM token in your database, whether `encrypted`, `hashed` or `plain` text. Default is `plain` text.

Alternatively, you can pass a custom encryptor or hasher to store the SCIM token in your database.

**Custom encryptor**

```ts title="auth.ts"
scim({
    storeSCIMToken: { 
        encrypt: async (scimToken) => {
            return myCustomEncryptor(scimToken);
        },
        decrypt: async (scimToken) => {
            return myCustomDecryptor(scimToken);
        },
    }
})
```

**Custom hasher**

```ts title="auth.ts"
scim({
    storeSCIMToken: {
        hash: async (scimToken) => {
            return myCustomHasher(scimToken);
        },
    }
})
```

Hooks [#hooks]

The following hooks allow to intercept the lifecycle of the `SCIM` token generation:

<Callout>
  **Note:** The built-in organization role check runs before these hooks. Use hooks to add stricter rules, not to bypass `requiredRole`.
</Callout>

```ts
const approvedScimOperators = new Set(["some-admin-user-id"]);

scim({
    beforeSCIMTokenGenerated: async ({ user, member, scimToken }) => {
        // `member` is null for personal connections.
        // Add any extra restrictions you need before the token is persisted.
        if (!approvedScimOperators.has(user.id)) {
            throw new APIError("FORBIDDEN", { message: "User does not have enough permissions" });
        }
    },
    afterSCIMTokenGenerated: async ({ user, member, scimToken, scimProvider }) => {
        // Callback called after the scim token has been persisted
        // can be useful to send a notification or otherwise share the token
        await shareSCIMTokenWithInterestedParty(scimToken);
    },
})
```

<Callout>
  **Note**: All hooks support error handling. Throwing an error in a before hook will prevent the operation from proceeding.
</Callout>
