---
url: https://better-auth.com/llms.txt/docs/plugins/api-key
title: "Api Key"
description: ""
access_date: 2026-08-28T22:27:55.614Z
current_date: 2026-08-28T22:27:55.614Z
---

API Key plugin for Better Auth.

The API Key plugin allows you to create and manage API keys for your application. It provides a way to authenticate and authorize API requests by verifying API keys.

## Features

- Create, manage, and verify API keys
- [Built-in rate limiting](https://better-auth.com/docs/plugins/api-key/advanced#rate-limiting)
- [Custom expiration times, remaining count, and refill systems](https://better-auth.com/docs/plugins/api-key/advanced#remaining-refill-and-expiration)
- [Metadata for API keys](https://better-auth.com/docs/plugins/api-key/advanced#metadata)
- Custom prefix
- [Sessions from API keys](https://better-auth.com/docs/plugins/api-key/advanced#sessions-from-api-keys)
- [Secondary storage support](https://better-auth.com/docs/plugins/api-key/advanced#storage-modes) for high-performance API key lookups
- [Multiple configurations](https://better-auth.com/docs/plugins/api-key/advanced#multiple-configurations) for different API key types
- [Organization-owned API keys](https://better-auth.com/docs/plugins/api-key/advanced#organization-owned-api-keys) in addition to user-owned keys

## Installation

### Install the plugin

#### npm

```
npm install @better-auth/api-key
```

#### pnpm

#### yarn

#### bun

### Add Plugin to the server

```
import { betterAuth } from "better-auth"
import { apiKey } from "@better-auth/api-key"

export const auth = betterAuth({
    plugins: [
        apiKey() 
    ]
})
```

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

See the [Schema](https://better-auth.com/docs/plugins/api-key/reference#schema) section to add the fields manually.

### Add the client plugin

```
import { createAuthClient } from "better-auth/client"
import { apiKeyClient } from "@better-auth/api-key/client"

export const authClient = createAuthClient({
    plugins: [
        apiKeyClient() 
    ]
})
```

## Usage

You can view the [API Key plugin options](https://better-auth.com/docs/plugins/api-key/reference#api-key-plugin-options).

### Create an API key

POST/api-key/create

Notes

You can adjust more specific API key configurations by using the server method instead.

```
const { data, error } = await authClient.apiKey.create({
    configId, // The configuration ID to use. If not provided, the default configuration is used.
    name: 'project-api-key', // Name of the Api Key.
    expiresIn: 60 * 60 * 24 * 7, // Expiration time of the Api Key in seconds.
    organizationId: "org-id", // Organization Id that the Api Key belongs to. Required for organization-owned keys (when config has \`references: "organization"\`).
    prefix: 'project-api-key', // Prefix of the Api Key.
    metadata: { someKey: 'someValue' }, // Metadata of the Api Key.
});
```

Parameters

`configId` string

The configuration ID to use. If not provided, the default configuration is used.

`name` string

Name of the Api Key.

`expiresIn` number

Expiration time of the Api Key in seconds.

`organizationId` string

Organization Id that the Api Key belongs to. Required for organization-owned keys (when config has `references: "organization"`).

`prefix` string

Prefix of the Api Key.

`metadata` any | null

Metadata of the Api Key.

#### Result

It'll return the `ApiKey` object which includes the `key` value for you to use. Otherwise if it throws, it will throw an `APIError`.

---

### Verify an API key

```
const permissions = { // Permissions to check are optional.
  projects: ["read", "read-write"],
}
const data = await auth.api.verifyApiKey({
    body: {
        configId, // Configuration ID to scope verification to. When omitted, the key is validated against its own configuration.
        key: "your_api_key_here", // required, The key to verify.
        permissions, // The permissions to verify. Optional.
    },
});
```

Parameters

`configId` string

Configuration ID to scope verification to. When omitted, the key is validated against its own configuration.

`key` stringrequired

The key to verify.

`permissions` Record<string, string\[\]>

The permissions to verify. Optional.

#### Result

```
type Result = {
  valid: boolean;
  error: { message: string; code: string } | null;
  key: Omit<ApiKey, "key"> | null;
};
```

---

### Get an API key

GET/api-key/get

```
const { data, error } = await authClient.apiKey.get({
    query: {
        configId, // The configuration ID to use for the API key lookup. If not provided, the default configuration is used.
        id: "some-api-key-id", // required, The id of the Api Key.
    },
});
```

Parameters

`configId` string

The configuration ID to use for the API key lookup. If not provided, the default configuration is used.

`id` stringrequired

The id of the Api Key.

#### Result

You'll receive everything about the API key details, except for the `key` value itself. If it fails, it will throw an `APIError`.

```
type Result = Omit<ApiKey, "key">;
```

---

### Update an API key

POST/api-key/update

```
const { data, error } = await authClient.apiKey.update({
    configId, // The configuration ID to use for the API key lookup. If not provided, the default configuration is used.
    keyId: "some-api-key-id", // required, The id of the Api Key to update.
    name: "some-api-key-name", // The name of the key.
});
```

Parameters

`configId` string

The configuration ID to use for the API key lookup. If not provided, the default configuration is used.

`keyId` stringrequired

The id of the Api Key to update.

`name` string

The name of the key.

#### Result

If fails, throws `APIError`. Otherwise, you'll receive the API Key details, except for the `key` value itself.

---

### Delete an API Key

POST/api-key/delete

Notes

This endpoint is attempting to delete the API key from the perspective of the user. It will check if the user's ID matches the key owner to be able to delete it. If you want to delete a key without these checks, we recommend you use an ORM to directly mutate your DB instead.

```
const { data, error } = await authClient.apiKey.delete({
    configId, // The configuration ID to use for the API key lookup. If not provided, the default configuration is used.
    keyId: "some-api-key-id", // required, The id of the Api Key to delete.
});
```

Parameters

`configId` string

The configuration ID to use for the API key lookup. If not provided, the default configuration is used.

`keyId` stringrequired

The id of the Api Key to delete.

#### Result

If fails, throws `APIError`. Otherwise, you'll receive:

```
type Result = {
  success: boolean;
};
```

---

### List API keys

GET/api-key/list

```
const { data, error } = await authClient.apiKey.list({
    query: {
        configId, // Filter by configuration ID. If not provided, returns keys from all configurations.
        organizationId, // Organization ID to list keys for. If provided, returns organization-owned keys. If not provided, returns user-owned keys for the current session user.
        limit, // The number of API keys to return.
        offset, // The offset to start from (for pagination).
        sortBy, // The field to sort by (e.g., "createdAt", "name", "expiresAt").
        sortDirection, // The direction to sort by.
    },
});
```

Parameters

`configId` string

Filter by configuration ID. If not provided, returns keys from all configurations.

`organizationId` string

Organization ID to list keys for. If provided, returns organization-owned keys. If not provided, returns user-owned keys for the current session user.

`limit` number

The number of API keys to return.

`offset` number

The offset to start from (for pagination).

`sortBy` string

The field to sort by (e.g., "createdAt", "name", "expiresAt").

`sortDirection` "asc" | "desc"

The direction to sort by.

#### Result

If fails, throws `APIError`. Otherwise, you'll receive a paginated response:

```
type Result = {
  apiKeys: Omit<ApiKey, "key">[];
  total: number;
  limit?: number;
  offset?: number;
};
```

#### Pagination Examples

```
// Get first 10 API keys for the current user
const result = await authClient.apiKey.list({
  query: { limit: 10 }
});

// Get second page (10 items per page)
const page2 = await authClient.apiKey.list({
  query: { limit: 10, offset: 10 }
});

// Sort by creation date (newest first)
const sorted = await authClient.apiKey.list({
  query: { sortBy: "createdAt", sortDirection: "desc" }
});

// Combined pagination and sorting
const combined = await authClient.apiKey.list({
  query: { 
    limit: 20, 
    offset: 0, 
    sortBy: "name", 
    sortDirection: "asc" 
  }
});

// List organization-owned keys
const orgKeys = await authClient.apiKey.list({
  query: { organizationId: "org_123" }
});

// List organization keys with specific config
const orgPublicKeys = await authClient.apiKey.list({
  query: { 
    organizationId: "org_123",
    configId: "public" 
  }
});
```

---

### Delete all expired API keys

This function will delete all API keys that have an expired expiration date.

```
const data = await auth.api.deleteAllExpiredApiKeys();
```
