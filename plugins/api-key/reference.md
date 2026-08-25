---
url: https://better-auth.com/llms.txt/docs/plugins/api-key/reference
title: "Reference"
description: ""
access_date: 2026-08-25T05:48:23.554Z
current_date: 2026-08-25T05:48:23.554Z
---

API Key plugin options, permissions, and schema reference.

## API Key plugin options

`configId` `string`

A unique identifier for this configuration. Required when using multiple configurations. Default is `"default"`.

`references` `"user" | "organization"`

What the API key references. This determines ownership over the API key. Default is `"user"`.

- `"user"`: API keys are owned by users (requires `userId` on creation)
- `"organization"`: API keys are owned by organizations (requires `organizationId` on creation)

`apiKeyHeaders` `string | string[];`

The header name to check for API key. Default is `x-api-key`.

`customAPIKeyGetter` `(ctx: GenericEndpointContext) => string | null`

A custom function to get the API key from the context.

`customAPIKeyValidator` `(options: { ctx: GenericEndpointContext; key: string; }) => boolean | Promise<boolean>`

A custom function to validate the API key.

`customKeyGenerator` `(options: { length: number; prefix: string | undefined; }) => string | Promise<string>`

A custom function to generate the API key.

`startingCharactersConfig` `{ shouldStore?: boolean; charactersLength?: number; }`

Customize the starting characters configuration.

`defaultKeyLength` `number`

The length of the API key. Longer is better. Default is 64. (Doesn't include the prefix length)

`defaultPrefix` `string`

The prefix of the API key.

Note: We recommend you append an underscore to the prefix to make the prefix more identifiable. (eg `hello_`)

`maximumPrefixLength` `number`

The maximum length of the prefix.

`minimumPrefixLength` `number`

The minimum length of the prefix.

`requireName` `boolean`

Whether to require a name for the API key. Default is `false`.

`maximumNameLength` `number`

The maximum length of the name.

`minimumNameLength` `number`

The minimum length of the name.

`enableMetadata` `boolean`

Whether to enable metadata for an API key.

`keyExpiration` `{ defaultExpiresIn?: number | null; disableCustomExpiresTime?: boolean; minExpiresIn?: number; maxExpiresIn?: number; }`

Customize the key expiration.

`rateLimit` `{ enabled?: boolean; timeWindow?: number; maxRequests?: number; }`

Customize the rate-limiting.

`schema` `InferOptionSchema<ReturnType<typeof apiKeySchema>>`

Custom schema for the API key plugin.

`enableSessionForAPIKeys` `boolean`

An API Key can represent a valid session, so we can mock a session for the user if we find a valid API key in the request headers. Default is `false`.

`storage` `"database" | "secondary-storage"`

Storage backend for API keys. Default is `"database"`.

- `"database"`: Store API keys in the database adapter (default)
- `"secondary-storage"`: Store API keys in the configured secondary storage (e.g., Redis)

`fallbackToDatabase` `boolean`

When `storage` is `"secondary-storage"`, enable fallback to database if key is not found in secondary storage. Default is `false`.

```
import { betterAuth } from "better-auth";
import { apiKey } from "@better-auth/api-key"

export const auth = betterAuth({
  secondaryStorage: {
    get: async (key) => {
      return await redis.get(key);
    },
    set: async (key, value, ttl) => {
      if (ttl) await redis.set(key, value, { EX: ttl });
      else await redis.set(key, value);
    },
    delete: async (key) => {
      await redis.del(key);
    },
  },
  plugins: [
    apiKey({
      storage: "secondary-storage",
    }),
  ],
});
```

`customStorage` `SecondaryStorage`

Custom secondary storage for API keys. If provided, it is used instead of `ctx.context.secondaryStorage`. Custom storage takes precedence over global secondary storage.

Useful when you want to use a different storage backend specifically for API keys, or when you need custom logic for storage operations. It must implement the full secondary-storage contract, including `getAndDelete` and `increment`.

```
import { betterAuth } from "better-auth";
import { apiKey } from "@better-auth/api-key"

export const auth = betterAuth({
  plugins: [
    apiKey({
      storage: "secondary-storage",
      customStorage: {
        get: async (key) => await customStorage.get(key),
        getAndDelete: async (key) => await customStorage.getAndDelete(key),
        increment: async (key, ttl) => await customStorage.increment(key, ttl),
        set: async (key, value, ttl) => await customStorage.set(key, value, ttl),
        delete: async (key) => await customStorage.delete(key), 
      },
    }),
  ],
});
```

`deferUpdates` `boolean`

Defer non-critical updates (rate limiting counters, timestamps, remaining count) to run after the response is sent using the global `backgroundTasks` handler. This can significantly improve response times on serverless platforms. Default is `false`.

Requires `backgroundTasks.handler` to be configured in the main auth options.

#### Vercel

```
import { waitUntil } from "@vercel/functions";

export const auth = betterAuth({
  advanced: { 
      backgroundTasks: {
         handler: waitUntil,
      },
  }
  plugins: [
    apiKey({
      deferUpdates: true,
    }),
  ],
});
```

#### Cloudflare Workers

`permissions` `{ defaultPermissions?: Statements | ((referenceId: string, ctx: GenericEndpointContext) => Statements | Promise<Statements>) }`

Permissions for the API key.

Read more about permissions [below](#permissions).

`disableKeyHashing` `boolean`

Disable hashing of the API key.

---

## Permissions

API keys can have permissions associated with them, allowing you to control access at a granular level. Permissions are structured as a record of resource types to arrays of allowed actions.

### Setting Default Permissions

You can configure default permissions that will be applied to all newly created API keys:

```
import { betterAuth } from "better-auth"
import { apiKey } from "@better-auth/api-key"

export const auth = betterAuth({
  plugins: [
    apiKey({
      permissions: {
        defaultPermissions: {
          files: ["read"],
          users: ["read"],
        },
      },
    }),
  ],
});
```

You can also provide a function that returns permissions dynamically:

```
import { betterAuth } from "better-auth"
import { apiKey } from "@better-auth/api-key"

export const auth = betterAuth({
  plugins: [
    apiKey({
      permissions: {
        defaultPermissions: async (referenceId, ctx) => {
          // referenceId is either userId or orgId depending on config
          // Fetch user/org role or other data to determine permissions
          return {
            files: ["read"],
            users: ["read"],
          };
        },
      },
    }),
  ],
});
```

### Creating API Keys with Permissions

When creating an API key, you can specify custom permissions:

```
import { auth } from "@/lib/auth"

const apiKey = await auth.api.createApiKey({
  body: {
    name: "My API Key",
    permissions: {
      files: ["read", "write"],
      users: ["read"],
    },
    userId: "userId",
  },
});
```

### Verifying API Keys with Required Permissions

When verifying an API key, you can check if it has the required permissions:

```
import { auth } from "@/lib/auth"

const result = await auth.api.verifyApiKey({
  body: {
    key: "your_api_key_here",
    permissions: {
      files: ["read"],
    },
  },
});

if (result.valid) {
  // API key is valid and has the required permissions
} else {
  // API key is invalid or doesn't have the required permissions
}
```

### Updating API Key Permissions

You can update the permissions of an existing API key:

```
import { auth } from "@/lib/auth"

const apiKey = await auth.api.updateApiKey({
  body: {
    keyId: existingApiKeyId,
    permissions: {
      files: ["read", "write", "delete"],
      users: ["read", "write"],
    },
  },
  headers: user_headers,
});
```

### Permissions Structure

Permissions follow a resource-based structure:

```
type Permissions = {
  [resourceType: string]: string[];
};

// Example:
const permissions = {
  files: ["read", "write", "delete"],
  users: ["read"],
  projects: ["read", "write"],
};
```

When verifying an API key, all required permissions must be present in the API key's permissions for validation to succeed.

## Schema

Table: `apikey`

Table

Field

Type

Attributes

Description

id

string

PKUQ

The ID of the API key.

configId

string

\-

The configuration ID this key belongs to. Default is 'default'.

name?

string

\-

The name of the API key.

start?

string

\-

The starting characters of the API key. Useful for showing the first few characters of the API key in the UI for the users to easily identify.

prefix?

string

\-

The API Key prefix. Stored as plain text.

key

string

\-

The hashed API key itself.

referenceId

string

\-

The ID of the owner (user ID or organization ID based on the config's \`references\` setting).

refillInterval?

number

\-

The interval to refill the key in milliseconds.

refillAmount?

number

\-

The amount to refill the remaining count of the key.

lastRefillAt?

Date

\-

The date and time when the key was last refilled.

enabled?

boolean

\-

Whether the API key is enabled.

rateLimitEnabled?

boolean

\-

Whether the API key has rate limiting enabled.

rateLimitTimeWindow?

number

\-

The time window in milliseconds for the rate limit.

rateLimitMax?

number

\-

The maximum number of requests allowed within the \`rateLimitTimeWindow\`.

requestCount?

number

\-

The number of requests made within the rate limit time window.

remaining?

number

\-

The number of requests remaining.

lastRequest?

Date

\-

The date and time of the last request made to the key.

expiresAt?

Date

\-

The date and time when the key will expire.

createdAt

Date

\-

The date and time the API key was created.

updatedAt

Date

\-

The date and time the API key was updated.

permissions?

string

\-

The permissions of the key.

metadata?

string

\-

Any additional metadata you want to store with the key.

### Migration from Previous Versions

If you're upgrading from a previous version, you'll need to migrate the `userId` field to the new reference system:

```
-- Add new columns
ALTER TABLE apikey ADD COLUMN config_id VARCHAR(255) NOT NULL DEFAULT 'default';
ALTER TABLE apikey ADD COLUMN reference_id VARCHAR(255);

-- Migrate existing data (copy userId to referenceId)
UPDATE apikey SET reference_id = user_id WHERE reference_id IS NULL;

-- Make reference_id required and add indexes
ALTER TABLE apikey ALTER COLUMN reference_id SET NOT NULL;
CREATE INDEX idx_apikey_reference_id ON apikey(reference_id);
CREATE INDEX idx_apikey_config_id ON apikey(config_id);

-- Optionally drop the old column after verifying migration
-- ALTER TABLE apikey DROP COLUMN user_id;
```
