---
url: https://better-auth.com/llms.txt/docs/plugins/admin
title: "Admin"
description: ""
access_date: 2026-08-18T00:08:46.984Z
current_date: 2026-08-18T00:08:46.984Z
---

Admin plugin for Better Auth

The Admin plugin provides a set of administrative functions for user management in your application. It allows administrators to perform various operations such as creating users, managing user roles, banning/unbanning users, impersonating users, and more.

## Installation

### Add the plugin to your auth config

To use the Admin plugin, add it to your auth config.

```
import { betterAuth } from "better-auth"
import { admin } from "better-auth/plugins"

export const auth = betterAuth({
    // ... other config options
    plugins: [
        admin() 
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

See the [Schema](#schema) section to add the fields manually.

### Add the client plugin

Next, include the admin client plugin in your authentication client instance.

```
import { createAuthClient } from "better-auth/client"
import { adminClient } from "better-auth/client/plugins"

export const authClient = createAuthClient({
    plugins: [
        adminClient()  
    ]
})
```

## Usage

Before performing any admin operations, the user must be authenticated with an admin account. An admin is any user assigned the `admin` role or any user whose ID is included in the `adminUserIds` option.

To create the first admin user, run the CLI after adding the Admin plugin and applying the schema:

#### npm

```
npx auth@latest create-admin --email admin@example.com --name "Admin" --role admin
```

#### pnpm

#### yarn

#### bun

If users already exist, the CLI asks for confirmation before creating another admin. Use `--force` or `--yes` to skip that prompt. The CLI marks the admin email as verified by default; pass `--no-email-verified` to disable that.

### Create User

Allows an admin to create a new user.

POST/admin/create-user

```
const { data: newUser, error } = await authClient.admin.createUser({
    email: "user@example.com", // required
    password: "some-secure-password", // required
    name: "James Smith", // required
    role: "user",
    data: { customField: "customValue" },
});
```

Parameters

`email` stringrequired

The email of the user.

`password` stringrequired

The password of the user.

`name` stringrequired

The name of the user.

`role` string | string\[\]

A string or array of strings representing the roles to apply to the new user.

`data` Record<string, any>

Extra fields for the user. Including custom additional fields.

### List Users

Allows an admin to list all users in the database.

GET/admin/list-users

Notes

All properties are optional to configure. By default, 100 rows are returned, you can configure this by the `limit` property.

```
const { data: users, error } = await authClient.admin.listUsers({
    query: {
        searchValue: "some name",
        searchField: "name",
        searchOperator: "contains",
        limit: 100,
        offset: 100,
        sortBy: "name",
        sortDirection: "desc",
        filterField: "email",
        filterValue: "hello@example.com",
        filterOperator: "eq",
    },
});
```

Parameters

`searchValue` string

The value to search for.

`searchField` "email" | "name"

The field to search in, defaults to email. Can be `email` or `name`.

`searchOperator` "contains" | "starts\_with" | "ends\_with"

The operator to use for the search. Can be `contains`, `starts_with` or `ends_with`.

`limit` string | number

The number of users to return. Defaults to 100.

`offset` string | number

The offset to start from.

`sortBy` string

The field to sort by.

`sortDirection` "asc" | "desc"

The direction to sort by.

`filterField` string

The field to filter by.

`filterValue` string | number | boolean | string\[\] | number\[\]

The value to filter by.

`filterOperator` "eq" | "ne" | "lt" | "lte" | "gt" | "gte" | "in" | "not\_in" | "contains" | "starts\_with" | "ends\_with"

The operator to use for the filter.

#### Query Filtering

The `listUsers` function supports various filter operators including `eq`, `contains`, `starts_with`, and `ends_with`.

#### Pagination

The `listUsers` function supports pagination by returning metadata alongside the user list. The response includes the following fields:

```
{
  users: User[],   // Array of returned users
  total: number,   // Total number of users after filters and search queries
  limit: number | undefined,   // The limit provided in the query
  offset: number | undefined   // The offset provided in the query
}
```

##### How to Implement Pagination

To paginate results, use the `total`, `limit`, and `offset` values to calculate:

- **Total pages:** `Math.ceil(total / limit)`
- **Current page:** `(offset / limit) + 1`
- **Next page offset:** `Math.min(offset + limit, (total - 1))` – The value to use as `offset` for the next page, ensuring it does not exceed the total number of pages.
- **Previous page offset:** `Math.max(0, offset - limit)` – The value to use as `offset` for the previous page (ensuring it doesn’t go below zero).

##### Example Usage

Fetching the second page with 10 users per page:

```
import { authClient } from "@/lib/auth-client";

const pageSize = 10;
const currentPage = 2;

const users = await authClient.admin.listUsers({
    query: {
        limit: pageSize,
        offset: (currentPage - 1) * pageSize
    }
});

const totalUsers = users.total;
const totalPages = Math.ceil(totalUsers / pageSize)
```

### Get User

Fetches a user's information using an id.

GET/admin/get-user

```
const { data, error } = await authClient.admin.getUser({
    query: {
        id: "user-id", // required
    },
});
```

Parameters

`id` stringrequired

The id of the user you want to fetch.

#### Returns

On success, `data` contains the user object. On failure, `error` is populated by `code`, `message`, `status`, and `statusText`.

```
type GetUserResponse = {
  data: User | null;
  error: null | {
    message: string;
    status: number; //HTTP status code
    statusText: string;
    code: string;
}
```

### Set User Role

Changes the role of a user.

POST/admin/set-role

```
const { data, error } = await authClient.admin.setRole({
    userId: "user-id",
    role: "admin", // required
});
```

Parameters

`userId` string

The user id which you want to set the role for.

`role` string | string\[\]required

The role to set, this can be a string or an array of strings.

### Set User Password

Sets the password for a user. A credential account is created if the user doesn't already have one.

POST/admin/set-user-password

```
const { data, error } = await authClient.admin.setUserPassword({
    newPassword: 'new-password', // required
    userId: 'user-id', // required
});
```

Parameters

`newPassword` stringrequired

The new password.

`userId` stringrequired

The user id which you want to set the password for.

### Update user

Update a user's details.

POST/admin/update-user

```
const { data, error } = await authClient.admin.updateUser({
    userId: "user-id", // required
    data: { name: "John Doe" }, // required
});
```

Parameters

`userId` stringrequired

The user id which you want to update.

`data` Record<string, any>required

The data to update.

### Ban User

Bans a user, preventing them from signing in and revokes all of their existing sessions.

POST/admin/ban-user

```
await authClient.admin.banUser({
    userId: "user-id", // required
    banReason: "Spamming",
    banExpiresIn: 60 * 60 * 24 * 7,
});
```

Parameters

`userId` stringrequired

The user id which you want to ban.

`banReason` string

The reason for the ban.

`banExpiresIn` number

The number of seconds until the ban expires. If not provided, the ban will never expire.

### Unban User

Removes the ban from a user, allowing them to sign in again.

POST/admin/unban-user

```
await authClient.admin.unbanUser({
    userId: "user-id", // required
});
```

Parameters

`userId` stringrequired

The user id which you want to unban.

### List User Sessions

Lists all sessions for a user.

POST/admin/list-user-sessions

```
const { data, error } = await authClient.admin.listUserSessions({
    userId: "user-id", // required
});
```

Parameters

`userId` stringrequired

The user id.

### Revoke User Session

Revokes a specific session for a user.

POST/admin/revoke-user-session

```
const { data, error } = await authClient.admin.revokeUserSession({
    sessionToken: "session_token_here", // required
});
```

Parameters

`sessionToken` stringrequired

The session token which you want to revoke.

### Revoke All Sessions for a User

Revokes all sessions for a user.

POST/admin/revoke-user-sessions

```
const { data, error } = await authClient.admin.revokeUserSessions({
    userId: "user-id", // required
});
```

Parameters

`userId` stringrequired

The user id which you want to revoke all sessions for.

### Impersonate User

This feature allows an admin to create a session that mimics the specified user. The session will remain active until either the browser session ends or it reaches 1 hour. You can change this duration by setting the `impersonationSessionDuration` option.

POST/admin/impersonate-user

```
const { data, error } = await authClient.admin.impersonateUser({
    userId: "user-id", // required
});
```

Parameters

`userId` stringrequired

The user id which you want to impersonate.

By default, admins cannot impersonate other admin users. To allow this, grant the `impersonate-admins` permission to a role:

```
const superAdmin = ac.newRole({
  ...adminAc.statements,
  user: ["impersonate-admins", ...adminAc.statements.user],
});
```

### Stop Impersonating User

To stop impersonating a user and continue with the admin account, you can use `stopImpersonating`

POST/admin/stop-impersonating

```
await authClient.admin.stopImpersonating();
```

### Remove User

Hard deletes a user from the database.

POST/admin/remove-user

```
const { data: deletedUser, error } = await authClient.admin.removeUser({
    userId: "user-id", // required
});
```

Parameters

`userId` stringrequired

The user id which you want to remove.

## Access Control

The admin plugin offers a highly flexible access control system, allowing you to manage user permissions based on their role. You can define custom permission sets to fit your needs.

### Roles

By default, there are two roles:

`admin`: Users with the admin role have full control over other users.

`user`: Users with the user role have no control over other users.

### Permissions

By default, there are two resources with the following permissions.

**user**: `create` `list` `set-role` `ban` `impersonate` `impersonate-admins` `delete` `set-password` `set-email` `get` `update`

**session**: `list` `revoke` `delete`

Users with the admin role have full control over all the resources and actions. Users with the user role have no control over any of those actions.

### Custom Permissions

The plugin provides an easy way to define your own set of permissions for each role.

#### Create Access Control

You first need to create an access controller by calling the `createAccessControl` function and passing the statement object. The statement object should have the resource name as the key and the array of actions as the value.

```
import { createAccessControl } from "better-auth/plugins/access";

/**
 * make sure to use \`as const\` so typescript can infer the type correctly
 */
const statement = { 
    project: ["create", "share", "update", "delete"], 
} as const; 

const ac = createAccessControl(statement);
```

#### Create Roles

Once you have created the access controller you can create roles with the permissions you have defined.

```
import { createAccessControl } from "better-auth/plugins/access";

export const statement = {
    project: ["create", "share", "update", "delete"], // <-- Permissions available for created roles
} as const;

export const ac = createAccessControl(statement);

export const user = ac.newRole({ 
    project: ["create"], 
}); 

export const admin = ac.newRole({ 
    project: ["create", "update"], 
}); 

export const myCustomRole = ac.newRole({ 
    project: ["create", "update", "delete"], 
    user: ["ban"], 
});
```

When you create custom roles for existing roles, the predefined permissions for those roles will be overridden. To add the existing permissions to the custom role, you need to import `defaultStatements` and merge it with your new statement, plus merge the roles' permissions set with the default roles.

```
import { createAccessControl } from "better-auth/plugins/access";
import { defaultStatements, adminAc } from "better-auth/plugins/admin/access";

const statement = {
    ...defaultStatements, 
    project: ["create", "share", "update", "delete"],
} as const;

const ac = createAccessControl(statement);

const admin = ac.newRole({
    project: ["create", "update"],
    ...adminAc.statements, 
});
```

#### Pass Roles to the Plugin

Once you have created the roles you can pass them to the admin plugin both on the client and the server.

```
import { betterAuth } from "better-auth"
import { admin as adminPlugin } from "better-auth/plugins"
import { ac, admin, user } from "@/auth/permissions"

export const auth = betterAuth({
    plugins: [
        adminPlugin({
            ac,
            roles: {
                admin,
                user,
                myCustomRole
            }
        }),
    ],
});
```

You also need to pass the access controller and the roles to the client plugin.

```
import { createAuthClient } from "better-auth/client"
import { adminClient } from "better-auth/client/plugins"
import { ac, admin, user, myCustomRole } from "@/auth/permissions"

export const client = createAuthClient({
    plugins: [
        adminClient({
            ac,
            roles: {
                admin,
                user,
                myCustomRole
            }
        })
    ]
})
```

### Access Control Usage

**Has Permission**:

To check a user's permissions, you can use the `hasPermission` function provided by the client.

POST/admin/has-permission

```
const { data, error } = await authClient.admin.hasPermission({
    userId: "user-id",
    permission: { "project": ["create", "update"] } /* Must use this, or permissions */,
    permissions,
});
```

Parameters

`userId` string

The user id which you want to check the permissions for.

`permission` Record<string, string\[\]>

Optionally check if a single permission is granted. Must use this, or permissions.

`permissions` Record<string, string\[\]>

Optionally check if multiple permissions are granted. Must use this, or permission.

Example usage:

```
import { authClient } from "@/lib/auth-client";

const canCreateProject = await authClient.admin.hasPermission({
  permissions: {
    project: ["create"],
  },
});

// You can also check multiple resource permissions at the same time
const canCreateProjectAndCreateSale = await authClient.admin.hasPermission({
  permissions: {
    project: ["create"],
    sale: ["create"]
  },
});
```

If you want to check a user's permissions server-side, you can use the `userHasPermission` action provided by the `api` to check the user's permissions.

```
import { auth } from "@/lib/auth"

await auth.api.userHasPermission({
  body: {
    userId: 'id', //the user id
    permissions: {
      project: ["create"], // This must match the structure in your access control
    },
  },
});

// You can also just pass the role directly
await auth.api.userHasPermission({
  body: {
   role: "admin",
    permissions: {
      project: ["create"], // This must match the structure in your access control
    },
  },
});

// You can also check multiple resource permissions at the same time
await auth.api.userHasPermission({
  body: {
   role: "admin",
    permissions: {
      project: ["create"], // This must match the structure in your access control
      sale: ["create"]
    },
  },
});
```

**Check Role Permission**:

Use the `checkRolePermission` function on the client side to verify whether a given **role** has a specific **permission**. This is helpful after defining roles and their permissions, as it allows you to perform permission checks without needing to contact the server.

Note that this function does **not** check the permissions of the currently logged-in user directly. Instead, it checks what permissions are assigned to a specified role. The function is synchronous, so you don't need to use `await` when calling it.

```
import { authClient } from "@/lib/auth-client";

const canCreateProject = authClient.admin.checkRolePermission({
  permissions: {
    user: ["delete"],
  },
  role: "admin",
});

// You can also check multiple resource permissions at the same time
const canDeleteUserAndRevokeSession = authClient.admin.checkRolePermission({
  permissions: {
    user: ["delete"],
    session: ["revoke"]
  },
  role: "admin",
});
```

## Schema

This plugin adds the following fields to the `user` table:

Table

Field

Type

Key

Description

role?

string

\-

The user's role. Defaults to \`user\`. Admins will have the \`admin\` role.

banned?

boolean

\-

Indicates whether the user is banned.

banReason?

string

\-

The reason for the user's ban.

banExpires?

date

\-

The date when the user's ban will expire.

And adds one field in the `session` table:

Table

Field

Type

Key

Description

impersonatedBy?

string

\-

The ID of the admin that is impersonating this session.

### Email Enumeration Protection

If you use [email enumeration protection](https://better-auth.com/docs/authentication/email-password#email-enumeration-protection) (`requireEmailVerification` or `autoSignIn: false`), you need to configure `customSyntheticUser` to include the admin plugin fields in the fake sign-up response:

```
export const auth = betterAuth({
  emailAndPassword: {
    enabled: true,
    requireEmailVerification: true,
    customSyntheticUser: ({ coreFields, additionalFields, id }) => ({
      ...coreFields,
      // Admin plugin fields (in schema order)
      role: "user", // or your configured defaultRole
      banned: false,
      banReason: null,
      banExpires: null,
      ...additionalFields,
      id,
    }),
  },
  plugins: [admin()],
});
```

## Options

### Default Role

The default role for a user. Defaults to `user`.

```
admin({
  defaultRole: "regular",
});
```

### Admin Roles

Specifies which roles are considered admin roles. Defaults to `["admin"]`. Custom roles (for example, `superadmin`) must be defined in custom access control.

```
admin({
  // Requires custom access control with \`superadmin\` defined in \`roles\`
  adminRoles: ["admin", "superadmin"],
});
```

### Admin userIds

You can pass an array of userIds that should be considered as admin. Default to `[]`

```
admin({
    adminUserIds: ["user_id_1", "user_id_2"]
})
```

If a user is in the `adminUserIds` list, they will be able to perform any admin operation.

### impersonationSessionDuration

The duration of the impersonation session in seconds. Defaults to 1 hour.

```
admin({
  impersonationSessionDuration: 60 * 60 * 24, // 1 day
});
```

### Default Ban Reason

The default ban reason for a user created by the admin. Defaults to `No reason`.

```
admin({
  defaultBanReason: "Spamming",
});
```

### Default Ban Expires In

The default ban expires in for a user created by the admin in seconds. Defaults to `undefined` (meaning the ban never expires).

```
admin({
  defaultBanExpiresIn: 60 * 60 * 24, // 1 day
});
```

### bannedUserMessage

The message to show when a banned user tries to sign in. Defaults to "You have been banned from this application. Please contact support if you believe this is an error."

```
admin({
  bannedUserMessage: "Custom banned user message",
});
```
