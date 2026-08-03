---
url: https://better-auth.com/llms.txt/docs/plugins/scim
title: "Scim"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Integrate SCIM with your application.

System for Cross-domain Identity Management ([SCIM](https://simplecloud.info/#Specification)) makes managing identities in multi-domain scenarios easier to support via a standardized protocol. This plugin exposes a [SCIM](https://simplecloud.info/#Specification) server that allows third party identity providers to sync identities to your service.

## Installation

### Install the plugin

#### npm

```
npm install @better-auth/scim
```

#### pnpm

#### yarn

#### bun

### Add Plugin to the server

```
import { betterAuth } from "better-auth"
import { scim } from "@better-auth/scim"; 

const auth = betterAuth({
    plugins: [
        scim() 
    ]
})
```

### Enable HTTP methods

SCIM requires the `POST`, `GET`, `PUT`, `PATCH` and `DELETE` HTTP methods to be supported by your server. For most frameworks, this will work out of the box, but some frameworks may require additional configuration:

#### Next.js

```
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { POST, GET, PUT, PATCH, DELETE } = toNextJsHandler(auth);
```

#### Solid Start

### Migrate the database

Run the migration or generate the schema to add the necessary fields and tables to the database.

#### migrate

#### npm

#### generate

```
npx auth migrate
```

#### pnpm

#### yarn

#### bun

See the [Schema](#schema) section to add the fields manually.

## Usage

Upon registration, this plugin will expose compliant [SCIM 2.0](https://simplecloud.info/#Specification) server. Generally, this server is meant to be consumed by a third-party (your identity provider), and will require a:

- **SCIM base URL**: This should be the fully qualified URL to the SCIM server (e.g `http://your-app.com/api/auth/scim/v2`)
- **SCIM bearer token**: See [generating a SCIM token](#generating-a-scim-token)

### Self-Service Directory Sync

If you're using [Better Auth Infrastructure](https://dash.better-auth.com/sign-in), you get self-service directory sync in the dashboard. Organization admins can create and manage SCIM directory connections and rotate bearer tokens without calling the SCIM APIs directly.

The dashboard is available at:

```
https://dash.better-auth.com/[project]/organization/[orgId]/enterprise
```

From the dashboard you can:

- **Create and remove directory connections** scoped to an organization
- **Regenerate SCIM bearer tokens** when your identity provider requires rotation

This eliminates the back-and-forth typically required when setting up SCIM, reducing onboarding time from days to minutes.

### Generating a SCIM token

Before your identity provider can start syncing information to your SCIM server, you need to generate a SCIM token that your identity provider will use to authenticate against it.

A SCIM token is a simple bearer token that you can generate:

POST/scim/generate-token

```
const { data, error } = await authClient.scim.generateToken({
    providerId: "acme-corp", // required
    organizationId: "the-org",
});
```

Parameters

`providerId` stringrequired

The provider id

`organizationId` string

Optional organization id. When specified, the organizations plugin must also be enabled

A `SCIM` token is always restricted to a provider, thus you are required to specify a `providerId`. This can be any provider your instance supports (e.g one of the built-in providers such as `credentials` or an external provider registered through an external plugin such as `@better-auth/sso`). Additionally, when the `organization` plugin is registered, you can optionally restrict the token to an organization via the `organizationId`.

#### Organization-scoped authorization

When `organizationId` is provided, Better Auth requires the current user to be a member of that organization and to have at least one of the configured `requiredRole` values.

By default, `requiredRole` resolves to:

- `admin`
- `organization.creatorRole` or `owner`

The same role requirement is also used by the SCIM management endpoints for organization-scoped connections:

- `GET /scim/list-provider-connections`
- `GET /scim/get-provider-connection`
- `POST /scim/delete-provider-connection`

```
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

#### Default SCIM token

We also provide a way for you to specify a `SCIM` token to use by default. This allows you to test a SCIM connection without setting up providers in the database:

```
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

### SCIM provider connection ownership

SCIM provider connection ownership applies to personal (non-organization) SCIM connections. It lets your application track who generated a connection and restricts later management operations for that connection to the same user.

```
import { betterAuth } from "better-auth";
import { scim } from "@better-auth/scim";

const auth = betterAuth({
    plugins: [
        scim({ 
            providerOwnership: { 
                enabled: true
            } 
        }) 
    ]
});
```

When enabled:

- Personal connections store the creating user's `userId`
- Only the owner can regenerate, list, inspect, or delete those personal connections later
- Organization-scoped connections continue to use the organization role checks configured by `requiredRole`

Once enabled, make sure you migrate the database schema (again).

#### migrate

```
npx auth migrate
```

#### generate

See the [Schema](#if-you-have-provider-ownership-enabled-via-providerownershipenabled) section to add the fields manually.

### Managing SCIM provider connections

You can manage SCIM provider connections from your application using the following endpoints:

#### List SCIM provider connections

List existing connections the current user can manage. For organization-scoped connections, the user must have one of the configured `requiredRole` roles for that organization. For personal connections, access is based on ownership when `providerOwnership.enabled` is turned on.

GET/scim/list-provider-connections

```
const { data, error } = await authClient.scim.listProviderConnections();
```

#### Get SCIM provider connection details

Get a single connection by provider id. Access is allowed only if the user can manage that connection: either because they satisfy the configured organization role requirement, or because they own the personal connection.

GET/scim/get-provider-connection

```
const { data, error } = await authClient.scim.getProviderConnection({
    query: {
        providerId: "acme-corp", // required
    },
});
```

Parameters

`providerId` stringrequired

Unique provider identifier

#### Delete SCIM provider connection

Delete an existing connection. This will immediately invalidate the connection's associated token.

POST/scim/delete-provider-connection

```
const { data, error } = await authClient.scim.deleteProviderConnection({
    providerId: "acme-corp", // required
});
```

Parameters

`providerId` stringrequired

Unique provider identifier

### SCIM endpoints

The following subset of the specification is currently supported:

#### List users

Get a list of available users in the database. This is restricted to list only users associated to the same provider and organization than your SCIM token.

GET/scim/v2/Users

Notes

Returns the provisioned SCIM user details. See https://datatracker.ietf.org/doc/html/rfc7644#section-3.4.1

```
const data = await auth.api.listSCIMUsers({
    query: {
        filter: 'userName eq "user-a"',
    },
    // This endpoint requires a bearer authentication token.
    headers: { authorization: 'Bearer <token>' },
});
```

Parameters

`filter` string

SCIM compliant filter expression

#### Get user

Get an user from the database. The user will be only returned if it belongs to the same provider and organization than the SCIM token.

GET/scim/v2/Users/:userId

Notes

Returns the provisioned SCIM user details. See https://datatracker.ietf.org/doc/html/rfc7644#section-3.4.1

```
const data = await auth.api.getSCIMUser({
    params: {
        userId: "user id", // required
    },
    // This endpoint requires a bearer authentication token.
    headers: { authorization: 'Bearer <token>' },
});
```

Parameters

`userId` stringrequired

Unique user identifier

#### Create new user

Provisions a new user to the database. The user will have an account associated to the same provider and will be member of the same org than the SCIM token.

POST/scim/v2/Users

Notes

Provision a new user via SCIM. See https://datatracker.ietf.org/doc/html/rfc7644#section-3.3

```
const data = await auth.api.createSCIMUser({
    body: {
        externalId: "third party id",
        name: {
            formatted: "Daniel Perez",
            givenName: "Daniel",
            familyName: "Perez",
        },
        emails: [{ value: "daniel@email.com", primary: true }],
    },
    // This endpoint requires a bearer authentication token.
    headers: { authorization: 'Bearer <token>' },
});
```

Parameters

`externalId` string

Unique external (third party) identifier

`name` Object

User name details

`formatted` string

Formatted name (takes priority over given and family name)

`givenName` string

Given name

`familyName` string

Family name

`emails` Array<{ value: string, primary?: boolean }>

List of emails associated to the user, only a single email can be primary

#### Update an existing user

Replaces an existing user details in the database. This operation can only update users that belong to the same provider and organization than the SCIM token.

PUT/scim/v2/Users/:userId

Notes

Updates an existing user via SCIM. See https://datatracker.ietf.org/doc/html/rfc7644#section-3.3

```
const data = await auth.api.updateSCIMUser({
    body: {
        externalId: "third party id",
        name: {
            formatted: "Daniel Perez",
            givenName: "Daniel",
            familyName: "Perez",
        },
        emails: [{ value: "daniel@email.com", primary: true }],
    },
    // This endpoint requires a bearer authentication token.
    headers: { authorization: 'Bearer <token>' },
});
```

Parameters

`externalId` string

Unique external (third party) identifier

`name` Object

User name details

`formatted` string

Formatted name (takes priority over given and family name)

`givenName` string

Given name

`familyName` string

Family name

`emails` Array<{ value: string, primary?: boolean }>

List of emails associated to the user, only a single email can be primary

#### Partial update an existing user

Allows to apply a partial update to the user details. This operation can only update users that belong to the same provider and organization than the SCIM token.

PATCH/scim/v2/Users/:userId

Notes

Partially updates a user resource. See https://datatracker.ietf.org/doc/html/rfc7644#section-3.5.2

```
const data = await auth.api.patchSCIMUser({
    body: {
        schemas: ["urn:ietf:params:scim:api:messages:2.0:PatchOp"], // required
        Operations: [{ op: "replace", path: "/userName", value: "any value" }], // required
    },
    // This endpoint requires a bearer authentication token.
    headers: { authorization: 'Bearer <token>' },
});
```

Parameters

`schemas` string\[\]required

Mandatory schema declaration

`Operations` Array<{ op: "replace" | "add" | "remove", path: string, value: any }>required

List of JSON patch operations

#### Deletes a user resource

Removes a user resource. This operation only affects users that belong to the same provider (and organization) as the SCIM token.

For an organization-scoped token, the user is deprovisioned from the organization: their membership and the provider account are removed, while the global user record is kept. For a non-organization token, the global user is deleted only when this provider's account is their sole identity; otherwise just that account is unlinked.

DELETE/scim/v2/Users/:userId

Notes

Deletes an existing user resource. See https://datatracker.ietf.org/doc/html/rfc7644#section-3.6

```
const data = await auth.api.deleteSCIMUser({
    params: {
        userId, // required
    },
    // This endpoint requires a bearer authentication token.
    headers: { authorization: 'Bearer <token>' },
});
```

Parameters

`userId` stringrequired

#### Get service provider config

Get SCIM metadata describing supported features of this server.

GET/scim/v2/ServiceProviderConfig

Notes

Standard SCIM metadata endpoint used by identity providers. See https://datatracker.ietf.org/doc/html/rfc7644#section-4

```
const data = await auth.api.getSCIMServiceProviderConfig();
```

#### Get SCIM schemas

Get the list of supported SCIM schemas.

GET/scim/v2/Schemas

Notes

Standard SCIM metadata endpoint used by identity providers to acquire information about supported schemas. See https://datatracker.ietf.org/doc/html/rfc7644#section-4

```
const data = await auth.api.getSCIMSchemas();
```

#### Get SCIM schema

Get the details of a supported SCIM schema.

GET/scim/v2/Schemas/:schemaId

Notes

Standard SCIM metadata endpoint used by identity providers to acquire information about a given schema. See https://datatracker.ietf.org/doc/html/rfc7644#section-4

```
const data = await auth.api.getSCIMSchema();
```

#### Get SCIM resource types

Get the list of supported SCIM types.

GET/scim/v2/ResourceTypes

Notes

Standard SCIM metadata endpoint used by identity providers to get a list of server supported types. See https://datatracker.ietf.org/doc/html/rfc7644#section-4

```
const data = await auth.api.getSCIMResourceTypes();
```

#### Get SCIM resource type

Get the details of a supported SCIM resource type.

GET/scim/v2/ResourceTypes/:resourceTypeId

Notes

Standard SCIM metadata endpoint used by identity providers to get a server supported type. See https://datatracker.ietf.org/doc/html/rfc7644#section-4

```
const data = await auth.api.getSCIMResourceType();
```

#### SCIM attribute mapping

By default, the SCIM provisioning will automatically map the following fields:

- `user.email`: User primary email or the first available email if there is not a primary one
- `user.name`: Derived from `name` (`name.formatted` or `name.givenName` + `name.familyName`) and fallbacks to the user primary email
- `account.providerId`: Provider associated to the `SCIM` token
- `account.accountId`: Defaults to `externalId` and fallbacks to `userName`
- `member.organizationId`: Organization associated to the provider

The SCIM `active` attribute maps to the user's disabled state. `active: false` deactivates the user (via the [admin](https://better-auth.com/docs/plugins/admin) plugin's `banned` state) and revokes their sessions; `active: true` reactivates. Honoring `active` requires the admin plugin. Changing a user's email through SCIM also resets their verified status.

## Schema

The plugin requires additional fields in the `scimProvider` table to store the provider's configuration.

Table

Field

Type

Key

Description

id

string

PK

A database identifier

providerId

string

\-

The provider ID. Used to identify a provider and to generate a redirect URL.

scimToken

string

\-

The SCIM bearer token. Used by your identity provider to authenticate against your server

organizationId?

string

\-

The organization Id. If provider is linked to an organization.

### If you have provider ownership enabled via providerOwnership.enabled:

The `scimProvider` schema is extended as follows:

Table

Field

Type

Key

Description

userId?

string

\-

The user id of the connection owner. Set automatically when generating a token via the API.

## Options

### Server

- `requiredRole`: `string[]` — Minimum organization role(s) allowed to generate organization-scoped tokens and manage organization-scoped connections. Defaults to `["admin", organization.creatorRole ?? "owner"]`.

Allow only owners to manage organization-scoped SCIM connections

```
scim({
    requiredRole: ["owner"],
})
```

- `providerOwnership`: `{ enabled: boolean }` — When enabled, links each personal provider connection to the user who generated its token. See [Connection ownership](#scim-provider-connection-ownership) for details. Default is `{ enabled: false }`.

```
scim({
    providerOwnership: { enabled: true },
})
```

- `defaultSCIM`: Default list of SCIM tokens for testing.
- `storeSCIMToken`: The method to store the SCIM token in your database, whether `encrypted`, `hashed` or `plain` text. Default is `plain` text.

Alternatively, you can pass a custom encryptor or hasher to store the SCIM token in your database.

**Custom encryptor**

```
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

```
scim({
    storeSCIMToken: {
        hash: async (scimToken) => {
            return myCustomHasher(scimToken);
        },
    }
})
```

### Hooks

The following hooks allow to intercept the lifecycle of the `SCIM` token generation:

```
const approvedScimOperators = new Set(["some-admin-user-id"]);

scim({
    beforeSCIMTokenGenerated: async ({ user, member, scimToken }) => {
        // \`member\` is null for personal connections.
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
