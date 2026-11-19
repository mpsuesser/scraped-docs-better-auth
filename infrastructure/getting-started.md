---
url: https://better-auth.com/llms.txt/docs/infrastructure/getting-started
title: "Getting Started"
description: ""
access_date: 2026-11-19T21:51:13.312Z
current_date: 2026-11-19T21:51:13.312Z
---

This guide will help you integrate Better Auth Infrastructure into your application.

## Prerequisites

Before you begin, make sure you have:

1. A working [Better Auth](https://better-auth.com/docs/installation) installation
2. An account & API Key from the [Better Auth Infrastructure](https://better-auth.com/dashboard) dashboard

## Installation

Install the `@better-auth/infra` package:

#### npm

```
npm install @better-auth/infra
```

#### pnpm

#### yarn

#### bun

## Environment Variables

Once you've gotten the API Key from the infrastructure dashboard, go ahead and add the `BETTER_AUTH_API_KEY` environment variable in your production environment variables.

```
# Required: Your Better Auth Infrastructure API key
BETTER_AUTH_API_KEY=your_api_key_here
```

## Server Setup

### Basic Configuration

Add the `dash()` plugin to your Better Auth configuration:

```
import { betterAuth } from "better-auth";
import { dash } from "@better-auth/infra"; 

export const auth = betterAuth({
  // ... your existing Better Auth config
  plugins: [
    dash({ 
      apiKey: process.env.BETTER_AUTH_API_KEY, 
    }), 
  ],
});
```

If you're on **pro** plan or above, you can use the `sentinel()` plugin to enable security checks, abuse protection, and more:

```
import { betterAuth } from "better-auth";
import { dash, sentinel } from "@better-auth/infra";  

export const auth = betterAuth({
  plugins: [
    dash({ /* ... */ }),
    sentinel({ 
      apiKey: process.env.BETTER_AUTH_API_KEY, 
    }) 
  ],
});
```

## Client Setup

### Basic Client Configuration

Add the client plugins to your auth client:

```
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

### Expo and React Native

For **Expo** or **React Native** clients, import from `@better-auth/infra/native` instead of `@better-auth/infra/client`. The native entry provides `dashClient` (same audit log APIs as the web client) and `sentinelNativeClient`.

```
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

See [Sentinel — Expo and React Native](https://better-auth.com/docs/infrastructure/plugins/sentinel#expo-and-react-native) for more details.

### Plugin Overview

- **`dash()`** - Enables analytics tracking, audit logging, dashboard admin APIs, and more
- **`sentinel()`** - Enables security checks and abuse protection

You can use either plugin independently, but using both together provides the full Better Auth Infrastructure experience.

## Next Steps

Now that you have the basic setup, explore these topics:

- [Dashboard](https://better-auth.com/docs/infrastructure/plugins/dashboard) - Analytics, activity tracking, and admin APIs
- [Audit Logs](https://better-auth.com/docs/infrastructure/plugins/audit-logs) - Track and query authentication events
- [Sentinel](https://better-auth.com/docs/infrastructure/plugins/sentinel) - Configure advanced security
- [Email Service](https://better-auth.com/docs/infrastructure/services/email) - Set up transactional emails
- [SMS Service](https://better-auth.com/docs/infrastructure/services/sms) - Enable SMS verification
