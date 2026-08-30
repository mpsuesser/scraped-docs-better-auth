---
url: https://better-auth.com/llms.txt/docs/plugins/multi-session
title: "Multi Session"
description: ""
access_date: 2026-08-30T04:55:24.662Z
current_date: 2026-08-30T04:55:24.662Z
---

Learn how to use multi-session plugin in Better Auth.

The multi-session plugin allows users to maintain multiple active sessions across different accounts in the same browser. This plugin is useful for applications that require users to switch between multiple accounts without logging out.

## Installation

### Add the plugin to your auth config

```
import { betterAuth } from "better-auth"
import { multiSession } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        multiSession(), 
    ]
})
```

### Add the client Plugin

Add the multi-session client plugin to enable managing multiple active sessions.

```
import { createAuthClient } from "better-auth/client"
import { multiSessionClient } from "better-auth/client/plugins"

export const authClient = createAuthClient({
    plugins: [
        multiSessionClient()  
    ]
})
```

## Usage

Whenever a user logs in, the plugin will add additional cookie to the browser. This cookie will be used to maintain multiple sessions across different accounts.

### List all device sessions

To list all active sessions for the current user, you can call the `listDeviceSessions` method.

GET/multi-session/list-device-sessions

```
const { data, error } = await authClient.multiSession.listDeviceSessions();
```

### Set active session

To set the active session, you can call the `setActive` method.

POST/multi-session/set-active

```
const { data, error } = await authClient.multiSession.setActive({
    sessionToken: "some-session-token", // required, The session token to set as active.
});
```

Parameters

`sessionToken` stringrequired

The session token to set as active.

### Revoke a session

To revoke a session, you can call the `revoke` method.

POST/multi-session/revoke

```
const { data, error } = await authClient.multiSession.revoke({
    sessionToken: "some-session-token", // required, The session token to revoke.
});
```

Parameters

`sessionToken` stringrequired

The session token to revoke.

### Signout and Revoke all sessions

When a user logs out, the plugin will revoke all active sessions for the user. You can do this by calling the existing `signOut` method, which handles revoking all sessions automatically.

### Max Sessions

You can specify the maximum number of sessions a user can have by passing the `maximumSessions` option to the plugin. By default, the plugin allows 5 sessions per device.

```
import { betterAuth } from "better-auth"
import { multiSession } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        multiSession({
            maximumSessions: 3
        })
    ]
})
```
