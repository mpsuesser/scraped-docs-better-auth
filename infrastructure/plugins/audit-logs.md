---
url: https://better-auth.com/llms.txt/docs/infrastructure/plugins/audit-logs
title: "Audit Logs"
description: ""
access_date: 2026-09-16T23:35:19.451Z
current_date: 2026-09-16T23:35:19.451Z
---

Track and query authentication events across your application with automatic audit logging.

Better Auth Infrastructure automatically collects audit logs for authentication events in your application. Once you've added the `dash()` plugin, events are tracked without any additional configuration.

## How Audit Logs Are Collected

The `dash()` plugin hooks into your Better Auth instance and automatically records events as they happen. No manual instrumentation is needed — sign-ups, sign-ins, password changes, and more are all captured.

```
import { betterAuth } from "better-auth";
import { dash } from "@better-auth/infra";

export const auth = betterAuth({
  plugins: [
    dash(),
  ],
});
```

That's it. Once `dash()` is active, audit logs are collected automatically.

## Tracked Events

#### User

| Event | Trigger |
| --- | --- |
| `user_signed_up` | New user registration |
| `user_profile_updated` | User updates their profile |
| `user_profile_image_updated` | User changes their avatar |
| `user_email_verified` | Email verification completed |
| `user_banned` | User is banned |
| `user_unbanned` | User is unbanned |
| `user_deleted` | User account deleted |

#### Session

#### Account

#### Verification

#### Organization

#### Security

## Fetching Audit Logs

### Client setup

Add `dashClient()` to your auth client:

```
import { createAuthClient } from "better-auth/client";
import { dashClient } from "@better-auth/infra/client";

export const authClient = createAuthClient({
  plugins: [dashClient()],
});
```

### Get current user's audit logs

Returns audit events for the **current user**. Use this for end-user activity and org views where the caller is a normal member.

#### Query parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `limit` | `number` | Results per page (max 100, default 50) |
| `offset` | `number` | Pagination offset (default 0) |
| `organizationId` | `string` | Filter by organization |
| `identifier` | `string` | Filter by identifier |
| `eventType` | `string` | Filter by event type |
| `userId` | `string` | Filter by user ID |
| `user` | `object` | User object with ID |
| `session` | `object` | Session object with user |

#### Basic query

```
const session = await authClient.getSession();

const logs = await authClient.dash.getAuditLogs({
  session: session.data,
  limit: 50,
  offset: 0,
});

logs.data?.events;  // Array of audit log events
logs.data?.total;   // Total count
logs.data?.limit;   // Page size
logs.data?.offset;  // Current offset
```

#### Filter by event type

Returns the current user's signed-in audit logs.

```
const signIns = await authClient.dash.getAuditLogs({
  session: session.data,
  eventType: "user_signed_in",
});
```

#### Filter by organization

Returns the current user's audit logs restricted to the given organization.

```
const orgLogs = await authClient.dash.getAuditLogs({
  session: session.data,
  organizationId: "org_123",
});
```

#### Combined filters

You can combine filters to get more specific results (e.g. all current user's signed-in events for a specific organization).

```
const orgSignIns = await authClient.dash.getAuditLogs({
  session: session.data,
  organizationId: "org_123",
  eventType: "user_signed_in",
});
```

You can also limit results to a specific identifier (e.g all current user's events for a specific org and email).

```
const orgSignIns = await authClient.dash.getAuditLogs({
  session: session.data,
  organizationId: "org_123",
  identifier: "user@example.com",
});
```

> **Note**: An identifier is a unique value for an event. For example, an email address, username, etc.

#### Pagination

Results are paginated. If you need to consume more than one page, you can use the `limit` and `offset` parameters to paginate the results:

```
async function fetchAllUserAuditLogEvents(session: unknown) {
  const limit = 100;
  let offset = 0;
  const allEvents = [];

  while (true) {
    const result = await authClient.dash.getAuditLogs({
      session: session.data,
      limit,
      offset,
    });
    
    const events = result.data?.events ?? [];
    allEvents.push(...events);

    if (events.length < limit) break;
    offset += limit;
  }

  return allEvents;
}
```

### Get all audit logs

Returns all audit events for organizations the current user has **admin** or **owner** access to. Use it for admin-style dashboards that need activity across organizations you manage. You must use the [organization plugin](https://better-auth.com/docs/plugins/organization) so membership roles can be evaluated.

#### Query parameters

| Parameter | Type | Description |
| --- | --- | --- |
| `limit` | `number` | Results per page (max `100`, default `50`) |
| `offset` | `number` | Pagination offset (default `0`) |
| `organizationId` | `string` | Limit results to one organization (you must be owner or admin) |
| `userId` | `string` | Limit to one user’s activity across organizations you administer as owner or admin |
| `eventType` | `string` | Filter by event type |
| `identifier` | `string` | Match `eventData.identifier` (organization-scoped actor identity) |
| `session` | `object` | Session object with user |

#### Basic query

```
const session = await authClient.getSession();

const activity = await authClient.dash.getAllAuditLogs({
  session: session.data,
  limit: 50,
  offset: 0,
});

activity.data?.events;
activity.data?.total;
activity.data?.limit;
activity.data?.offset;
```

#### Filter by organization

Returns all audit events for the given organization:

```
const orgActivity = await authClient.dash.getAllAuditLogs({
  session: session.data,
  organizationId: "org_123",
});
```

#### Filter by user

Returns all audit events for the given user across organizations:

```
const userActivity = await authClient.dash.getAllAuditLogs({
  session: session.data,
  userId: "user_456",
});
```

#### Filter by event type

Returns all audit events across organizations for the given event type:

```
const events = await authClient.dash.getAllAuditLogs({
  session: session.data,
  eventType: "member_invited",
});
```

#### Combined filters

You can combine filters to get more specific results (e.g. all audit events for a specific organization and event type).

```
const orgEvents = await authClient.dash.getAllAuditLogs({
  session: session.data,
  organizationId: "org_123",
  eventType: "member_invited",
});
```

#### Pagination

Results are paginated. If you need to consume more than one page, you can use the `limit` and `offset` parameters to paginate the results:

```
async function fetchAllAuditLogs(session: unknown) {
  const limit = 100;
  let offset = 0;
  const allEvents = [];

  while (true) {
    const result = await authClient.dash.getAllAuditLogs({
      session: session.data,
      limit,
      offset,
    });
    
    const events = result.data?.events ?? [];
    allEvents.push(...events);

    if (events.length < limit) break;
    offset += limit;
  }

  return allEvents;
}
```[Security Plugin (sentinel)](https://better-auth.com/docs/infrastructure/plugins/sentinel)

[

The \`sentinel()\` plugin provides comprehensive security and abuse protection for your authentication system. It detects and prevents various attack vectors including credential stuffing, impossible travel, free trial abuse, and more.

](https://better-auth.com/docs/infrastructure/plugins/sentinel)
