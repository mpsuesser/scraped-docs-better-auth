---
url: https://better-auth.com/llms.txt/docs/infrastructure/introduction
title: "Introduction"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Enterprise-grade dashboard, security, and managed services for Better Auth.

Better Auth Infrastructure is a paid service that extends your authentication system with a management dashboard, abuse protection, transactional messaging, and enterprise features - all without managing additional infrastructure.

## Key Features

### Dashboard

Manage users, organizations, sessions, and view analytics from a single interface.

### Security

Protect against credential stuffing, bots, disposable emails, and impossible travel.

### Email & SMS

Send verification emails, password resets, and OTP codes with pre-built templates.

### Enterprise

SSO/SAML, directory sync (SCIM), log drains, and role-based dashboard access.

You're one plugin away from getting started:

```
import { betterAuth } from "better-auth";
import { dash } from "@better-auth/infra";
import { sentinel } from "@better-auth/infra";

export const auth = betterAuth({
  plugins: [dash()],
});
```

Check out our [Getting Started](https://better-auth.com/docs/infrastructure/getting-started) guide for more details.
