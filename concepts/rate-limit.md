---
url: https://better-auth.com/llms.txt/docs/concepts/rate-limit
title: "Rate Limit"
description: ""
access_date: 2026-08-28T22:16:12.077Z
current_date: 2026-08-28T22:16:12.077Z
---

Learn how to configure rate limiting in Better Auth, including IP address detection, IPv6 support, custom rate limit windows, storage backends, error handling, and per-endpoint rules.

Better Auth includes a built-in rate limiter to help manage traffic and prevent abuse. By default, in production mode, the rate limiter is set to:

- Window: 60 seconds
- Max Requests: 100 requests

You can easily customize these settings by passing the rateLimit object to the betterAuth function.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    rateLimit: {
        window: 10, // time window in seconds
        max: 100, // max requests in the window
    },
})
```

Rate limiting is disabled in development mode by default. In order to enable it, set `enabled` to `true`:

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    rateLimit: {
        enabled: true,
        //...other options
    },
})
```

In addition to the default settings, Better Auth provides custom rules for specific paths. For example:

- `/sign-in/email`: Is limited to 3 requests within 10 seconds.

In addition, plugins also define custom rules for specific paths. For example, `twoFactor` plugin has custom rules:

- `/two-factor/verify`: Is limited to 3 requests within 10 seconds.

These custom rules ensure that sensitive operations are protected with stricter limits.

## Configuring Rate Limit

### Connecting IP Address

Rate limiting uses the connecting IP address to track the number of requests made by a user. The default header checked is `x-forwarded-for`, which is commonly used in production environments. If you are using a different header to track the user's IP address, you'll need to specify it.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    //...other options
    advanced: {
        ipAddress: {
          ipAddressHeaders: ["cf-connecting-ip"], // Cloudflare specific header example
      },
    },
    rateLimit: {
        enabled: true,
        window: 60, // time window in seconds
        max: 100, // max requests in the window
    },
})
```

By default Better Auth does not trust comma-separated forwarded IP chains, since the leftmost `X-Forwarded-For` token is client-controlled behind an appending proxy. Behind a reverse proxy or load balancer, do one of the following.

Point `ipAddressHeaders` at a single trusted header your proxy sets (for example `x-real-ip` or `cf-connecting-ip`):

```
export const auth = betterAuth({
    advanced: {
        ipAddress: {
            ipAddressHeaders: ["x-real-ip"],
        },
    },
})
```

Or list your proxies in `trustedProxies`. Better Auth walks the chain right to left, skips the trusted hops, and takes the first untrusted address as the client:

```
export const auth = betterAuth({
    advanced: {
        ipAddress: {
            // your proxies' addresses, not a broad private range that also covers clients
            trustedProxies: ["192.0.2.10", "10.0.0.0/24"],
        },
    },
})
```

#### IPv6 Address Support

Better Auth automatically normalizes IPv6 addresses to prevent bypass attacks where attackers use different representations of the same IPv6 address (e.g., `2001:db8::1` vs `2001:0db8:0000:0000:0000:0000:0000:0001`). This ensures that all representations of the same IPv6 address are treated as the same for rate limiting purposes.

Additionally, IPv4-mapped IPv6 addresses (e.g., `::ffff:192.0.2.1`) are automatically converted to their IPv4 form (`192.0.2.1`) to prevent attackers from bypassing rate limits by switching between IPv4 and IPv6 representations.

#### IPv6 Subnet Rate Limiting

By default, IPv6 addresses are rate limited per `/64` subnet, not per individual address. ISPs and cloud providers assign IPv6 prefixes (typically `/64` for residential users per [RFC 6177](https://datatracker.ietf.org/doc/html/rfc6177)) rather than single addresses, so any per-address counter would let one client rotate through 2^64 source addresses without exhausting the limit.

You can override the prefix length via the `ipv6Subnet` option if your deployment needs a different allocation boundary:

```
export const auth = betterAuth({
    //...other options
    advanced: {
        ipAddress: {
            ipv6Subnet: 56, // Rate limit by /56 subnet (residential ISP allocation)
        },
    },
    rateLimit: {
        enabled: true,
        window: 60,
        max: 100,
    },
})
```

Common IPv6 prefix lengths:

- `128`: Individual IPv6 address. Most restrictive. Only safe when you control the network and trust that each address maps to a distinct client.
- `64` (default): /64 subnet. Typical home or business allocation.
- `56`: /56 subnet. Residential ISP allocation per [RFC 6177](https://datatracker.ietf.org/doc/html/rfc6177).
- `48`: /48 subnet. Larger network allocation.
- `32`: /32 subnet. ISP-level allocation.

Any integer prefix length from `0` to `128` is accepted; the values above are the most common.

### Rate Limit Window

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    //...other options
    rateLimit: {
        window: 60, // time window in seconds
        max: 100, // max requests in the window
    },
})
```

You can also pass custom rules for specific paths.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    //...other options
    rateLimit: {
        window: 60, // time window in seconds
        max: 100, // max requests in the window
        customRules: {
            "/sign-in/email": {
                window: 10,
                max: 3,
            },
            "/two-factor/*": async (request)=> {
                // custom function to return rate limit window and max
                return {
                    window: 10,
                    max: 3,
                }
            }
        },
    },
})
```

If you like to disable rate limiting for a specific path, you can set it to `false` or return `false` from the custom rule function.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    //...other options
    rateLimit: {
        customRules: {
            "/get-session": false,
        },
    },
})
```

### Storage

By default, rate limit data is stored in memory, which may not be suitable for many use cases, particularly in serverless environments. To address this, you can use a database, secondary storage, or custom storage for storing rate limit data.

**Using Database**

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    //...other options
    rateLimit: {
        storage: "database",
        modelName: "rateLimit", //optional by default "rateLimit" is used
    },
})
```

Make sure to run `migrate` to create the rate limit table in your database:

#### npm

```
npx auth@latest migrate
```

#### pnpm

#### yarn

#### bun

**Using Secondary Storage**

If a [Secondary Storage](https://better-auth.com/docs/concepts/database#secondary-storage) has been configured you can use that to store rate limit data.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    //...other options
    rateLimit: {
        storage: "secondary-storage"
    },
})
```

**Custom Storage**

If none of the above solutions suits your use case you can implement a `customStorage`.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    //...other options
    rateLimit: {
        customStorage: {
            consume: async (key, rule) => {
                // atomically record one request for \`key\`
                // rule.window is in seconds, rule.max is the request limit
                return {
                    allowed: true,
                    retryAfter: null,
                };
            },
        },
    },
})
```

`customStorage.consume` must check and increment in one operation. Better Auth no longer accepts separate `get` and `set` methods for rate limiting because that shape can allow concurrent requests to pass the same stale counter.

## Handling Rate Limit Errors

When a request exceeds the rate limit, Better Auth returns the following header:

- `X-Retry-After`: The number of seconds until the user can make another request.

To handle rate limit errors on the client side, you can manage them either globally or on a per-request basis. Since Better Auth clients wrap over Better Fetch, you can pass `fetchOptions` to handle rate limit errors

**Global Handling**

```
import { createAuthClient } from "better-auth/client";

export const authClient = createAuthClient({
    fetchOptions: {
        onError: async (context) => {
            const { response } = context;
            if (response.status === 429) {
                const retryAfter = response.headers.get("X-Retry-After");
                console.log(\`Rate limit exceeded. Retry after ${retryAfter} seconds\`);
            }
        },
    }
})
```

**Per Request Handling**

```
import { authClient } from "./auth-client";

await authClient.signIn.email({
    fetchOptions: {
        onError: async (context) => {
            const { response } = context;
            if (response.status === 429) {
                const retryAfter = response.headers.get("X-Retry-After");
                console.log(\`Rate limit exceeded. Retry after ${retryAfter} seconds\`);
            }
        },
    }
})
```

### Schema

If you are using a database to store rate limit data you need this schema:

Table Name: `rateLimit`

Table

Field

Type

Attributes

Description

id

string

PK

Database ID

key

string

UQ

Unique identifier for each rate limit key

count

integer

\-

Number of requests made in the current window

lastRequest

bigint

\-

Timestamp of the last request (epoch ms)[OAuth](https://better-auth.com/docs/concepts/oauth)

[

Learn how to configure social OAuth providers, sign in and link accounts, request scopes, pass additional data, refresh access tokens, map profiles, and customize provider options.

](https://better-auth.com/docs/concepts/oauth)
