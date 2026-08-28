---
url: https://better-auth.com/llms.txt/docs/plugins/organization
title: "Organization"
description: ""
access_date: 2026-08-28T22:27:55.614Z
current_date: 2026-08-28T22:27:55.614Z
---

The organization plugin allows you to manage your organization's members and teams.

Organizations simplifies user access and permissions management. Assign roles and permissions to streamline project management, team coordination, and partnerships.

## Installation

### Add the plugin to your auth config

```
import { betterAuth } from "better-auth"
import { organization } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        organization() 
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

```
import { createAuthClient } from "better-auth/client"
import { organizationClient } from "better-auth/client/plugins"

export const authClient = createAuthClient({
    plugins: [
        organizationClient() 
    ]
})
```

## Usage

Once you've installed the plugin, you can start using the organization plugin to manage your organization's members and teams. The client plugin will provide you with methods under the `organization` namespace, and the server `api` will provide you with the necessary endpoints to manage your organization and give you an easier way to call the functions on your own backend.

## Organization

### Create an organization

POST/organization/create

```
const metadata = { someKey: "someValue" };
const { data, error } = await authClient.organization.create({
    name: "My Organization", // required, The organization name.
    slug: "my-org", // required, The organization slug.
    logo: "https://example.com/logo.png", // The organization logo.
    metadata, // The metadata of the organization.
    keepCurrentActiveOrganization: false, // Whether to keep the current active organization active after creating a new one.
});
```

Parameters

`name` stringrequired

The organization name.

`slug` stringrequired

The organization slug.

`logo` string | null

The organization logo.

`metadata` Record<string, any>

The metadata of the organization.

`keepCurrentActiveOrganization` boolean

Whether to keep the current active organization active after creating a new one.

#### Restrict who can create an organization

By default, any user can create an organization. To restrict this, set the `allowUserToCreateOrganization` option to a function that returns a boolean, or directly to `true` or `false`.

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

const auth = betterAuth({
  //...
  plugins: [
    organization({
      allowUserToCreateOrganization: async (user) => { 
        const subscription = await getSubscription(user.id); 
        return subscription.plan === "pro"; 
      }, 
    }),
  ],
});
```

#### Check if organization slug is taken

To check if an organization slug is taken or not you can use the `checkSlug` function provided by the client. The function takes an object with the following properties:

POST/organization/check-slug

```
const { data, error } = await authClient.organization.checkSlug({
    slug: "my-org", // required, The organization slug to check.
});
```

Parameters

`slug` stringrequired

The organization slug to check.

### Organization Hooks

You can customize organization operations using hooks that run before and after various organization-related activities. Better Auth provides two ways to configure hooks:

1. **Legacy organizationCreation hooks** (deprecated, use `organizationHooks` instead)
2. **Modern organizationHooks** (recommended) - provides comprehensive control over all organization-related activities

#### Organization Creation and Management Hooks

Control organization lifecycle operations:

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    organization({
      organizationHooks: {
        // Organization creation hooks
        beforeCreateOrganization: async ({ organization, user }) => {
          // Run custom logic before organization is created
          // Optionally modify the organization data
          return {
            data: {
              ...organization,
              metadata: {
                customField: "value",
              },
            },
          };
        },

        afterCreateOrganization: async ({ organization, member, user }) => {
          // Run custom logic after organization is created
          // e.g., create default resources, send notifications
          await setupDefaultResources(organization.id);
        },

        // Organization update hooks
        beforeUpdateOrganization: async ({ organization, user, member }) => {
          // Validate updates, apply business rules
          return {
            data: {
              ...organization,
              name: organization.name?.toLowerCase(),
            },
          };
        },

        afterUpdateOrganization: async ({ organization, user, member }) => {
          // Sync changes to external systems
          await syncOrganizationToExternalSystems(organization);
        },
      },
    }),
  ],
});
```

#### Member Hooks

Control member operations within organizations:

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    organization({
      organizationHooks: {
        // Before a member is added to an organization
        beforeAddMember: async ({ member, user, organization }) => {
          // Custom validation or modification
          console.log(\`Adding ${user.email} to ${organization.name}\`);

          // Optionally modify member data
          return {
            data: {
              ...member,
              role: "custom-role", // Override the role
            },
          };
        },

        // After a member is added
        afterAddMember: async ({ member, user, organization }) => {
          // Send welcome email, create default resources, etc.
          await sendWelcomeEmail(user.email, organization.name);
        },

        // Before a member is removed
        beforeRemoveMember: async ({ member, user, organization }) => {
          // Cleanup user's resources, send notification, etc.
          await cleanupUserResources(user.id, organization.id);
        },

        // After a member is removed
        afterRemoveMember: async ({ member, user, organization }) => {
          await logMemberRemoval(user.id, organization.id);
        },

        // Before updating a member's role
        beforeUpdateMemberRole: async ({
          member,
          newRole,
          user,
          organization,
        }) => {
          // Validate role change permissions
          if (newRole === "owner" && !hasOwnerUpgradePermission(user)) {
            throw new Error("Cannot upgrade to owner role");
          }

          // Optionally modify the role
          return {
            data: {
              role: newRole,
            },
          };
        },

        // After updating a member's role
        afterUpdateMemberRole: async ({
          member,
          previousRole,
          user,
          organization,
        }) => {
          await logRoleChange(user.id, previousRole, member.role);
        },
      },
    }),
  ],
});
```

#### Invitation Hooks

Control invitation lifecycle:

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    organization({
      organizationHooks: {
        // Before creating an invitation
        beforeCreateInvitation: async ({
          invitation,
          inviter,
          organization,
        }) => {
          // Custom validation or expiration logic
          const customExpiration = new Date(
            Date.now() + 1000 * 60 * 60 * 24 * 7
          ); // 7 days

          return {
            data: {
              ...invitation,
              expiresAt: customExpiration,
            },
          };
        },

        // After creating an invitation
        afterCreateInvitation: async ({
          invitation,
          inviter,
          organization,
        }) => {
          // Send custom invitation email, track metrics, etc.
          await sendCustomInvitationEmail(invitation, organization);
        },

        // Before accepting an invitation
        beforeAcceptInvitation: async ({ invitation, user, organization }) => {
          // Additional validation before acceptance
          await validateUserEligibility(user, organization);
        },

        // After accepting an invitation
        afterAcceptInvitation: async ({
          invitation,
          member,
          user,
          organization,
        }) => {
          // Setup user account, assign default resources
          await setupNewMemberResources(user, organization);
        },

        // Before/after rejecting invitations
        beforeRejectInvitation: async ({ invitation, user, organization }) => {
          // Log rejection reason, send notification to inviter
        },

        afterRejectInvitation: async ({ invitation, user, organization }) => {
          await notifyInviterOfRejection(invitation.inviterId, user.email);
        },

        // Before/after cancelling invitations
        beforeCancelInvitation: async ({
          invitation,
          cancelledBy,
          organization,
        }) => {
          // Verify cancellation permissions
        },

        afterCancelInvitation: async ({
          invitation,
          cancelledBy,
          organization,
        }) => {
          await logInvitationCancellation(invitation.id, cancelledBy.id);
        },
      },
    }),
  ],
});
```

#### Team Hooks

Control team operations (when teams are enabled):

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    organization({
      teams: { enabled: true },
      organizationHooks: {
        // Before creating a team
        beforeCreateTeam: async ({ team, user, organization }) => {
          // Validate team name, apply naming conventions
          return {
            data: {
              ...team,
              name: team.name.toLowerCase().replace(/\s+/g, "-"),
            },
          };
        },

        // After creating a team
        afterCreateTeam: async ({ team, user, organization }) => {
          // Create default team resources, channels, etc.
          await createDefaultTeamResources(team.id);
        },

        // Before updating a team
        beforeUpdateTeam: async ({ team, updates, user, organization }) => {
          // Validate updates, apply business rules
          return {
            data: {
              ...updates,
              name: updates.name?.toLowerCase(),
            },
          };
        },

        // After updating a team
        afterUpdateTeam: async ({ team, user, organization }) => {
          await syncTeamChangesToExternalSystems(team);
        },

        // Before deleting a team
        beforeDeleteTeam: async ({ team, user, organization }) => {
          // Backup team data, notify members
          await backupTeamData(team.id);
        },

        // After deleting a team
        afterDeleteTeam: async ({ team, user, organization }) => {
          await cleanupTeamResources(team.id);
        },

        // Team member operations
        beforeAddTeamMember: async ({
          teamMember,
          team,
          user,
          organization,
        }) => {
          // Validate team membership limits, permissions
          const memberCount = await getTeamMemberCount(team.id);
          if (memberCount >= 10) {
            throw new Error("Team is full");
          }
        },

        afterAddTeamMember: async ({
          teamMember,
          team,
          user,
          organization,
        }) => {
          await grantTeamAccess(user.id, team.id);
        },

        beforeRemoveTeamMember: async ({
          teamMember,
          team,
          user,
          organization,
        }) => {
          // Backup user's team-specific data
          await backupTeamMemberData(user.id, team.id);
        },

        afterRemoveTeamMember: async ({
          teamMember,
          team,
          user,
          organization,
        }) => {
          await revokeTeamAccess(user.id, team.id);
        },
      },
    }),
  ],
});
```

#### Hook Error Handling

All hooks support error handling. Throwing an error in a `before` hook will prevent the operation from proceeding:

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";
import { APIError } from "better-auth/api";

export const auth = betterAuth({
  plugins: [
    organization({
      organizationHooks: {
        beforeAddMember: async ({ member, user, organization }) => {
          // Check if user has pending violations
          const violations = await checkUserViolations(user.id);
          if (violations.length > 0) {
            throw new APIError("BAD_REQUEST", {
              message:
                "User has pending violations and cannot join organizations",
            });
          }
        },

        beforeCreateTeam: async ({ team, user, organization }) => {
          // Validate team name uniqueness
          const existingTeam = await findTeamByName(team.name, organization.id);
          if (existingTeam) {
            throw new APIError("BAD_REQUEST", {
              message: "Team name already exists in this organization",
            });
          }
        },
      },
    }),
  ],
});
```

### List User's Organizations

To list the organizations that a user is a member of, you can use `useListOrganizations` hook. It implements a reactive way to get the organizations that the user is a member of.

#### React

```
import { authClient } from "@/lib/auth-client"

function App(){
const { data: organizations } = authClient.useListOrganizations()
return (
  <div>
    {organizations.map((org) => (
      <p>{org.name}</p>
    ))}
  </div>)
}
```

#### Vue

#### Svelte

Or alternatively, you can call `organization.list` if you don't want to use a hook.

GET/organization/list

```
const { data, error } = await authClient.organization.list();
```

### Active Organization

Active organization is the workspace the user is currently working on. By default when the user is signed in the active organization is set to `null`. You can set the active organization to the user session.

#### Set Active Organization

You can set the active organization by calling the `organization.setActive` function. It'll set the active organization for the user session.

POST/organization/set-active

```
const { data, error } = await authClient.organization.setActive({
    organizationId: "org-id", // The organization ID to set as active. It can be null to unset the active organization.
    organizationSlug: "org-slug", // The organization slug to set as active. It can be null to unset the active organization if organizationId is not provided.
});
```

Parameters

`organizationId` string | null

The organization ID to set as active. It can be null to unset the active organization.

`organizationSlug` string

The organization slug to set as active. It can be null to unset the active organization if organizationId is not provided.

To automatically set an active organization when a session is created, you can use [database hooks](https://better-auth.com/docs/concepts/database#database-hooks). You'll need to implement logic to determine which organization to set as the initial active organization.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  databaseHooks: {
    session: {
      create: {
        before: async (session) => {
          // Implement your custom logic to set initial active organization
          const organization = await getInitialOrganization(session.userId);
          return {
            data: {
              ...session,
              activeOrganizationId: organization?.id,
            },
          };
        },
      },
    },
  },
});
```

#### Use Active Organization

To retrieve the active organization for the user, you can call the `useActiveOrganization` hook. It returns the active organization for the user. Whenever the active organization changes, the hook will re-evaluate and return the new active organization.

#### React

```
import { authClient } from "@/lib/auth-client"

function App(){
    const { data: activeOrganization } = authClient.useActiveOrganization()
    return (
        <div>
            {activeOrganization ? <p>{activeOrganization.name}</p> : null}
        </div>
    )
}
```

#### Vue

#### Svelte

### Get Organization

To get organization metadata without members or invitations, use `getOrganization`. By default, if you don't pass any properties, it will use the active organization.

Prefer this over `getFullOrganization` when you only need fields like `id`, `name`, `slug`, `logo`, and `metadata`.

GET/organization/get-organization

```
const { data, error } = await authClient.organization.getOrganization({
    query: {
        organizationId: "org-id", // The organization ID to get. By default, it will use the active organization.
        organizationSlug: "org-slug", // The organization slug to get.
    },
});
```

Parameters

`organizationId` string

The organization ID to get. By default, it will use the active organization.

`organizationSlug` string

The organization slug to get.

### Get Full Organization

To get the full details of an organization, you can use the `getFullOrganization` function. By default, if you don't pass any properties, it will use the active organization.

GET/organization/get-full-organization

```
const { data, error } = await authClient.organization.getFullOrganization({
    query: {
        organizationId: "org-id", // The organization ID to get. By default, it will use the active organization.
        organizationSlug: "org-slug", // The organization slug to get.
        membersLimit: 100, // The limit of members to get. By default, it uses the membershipLimit option which defaults to 100.
    },
});
```

Parameters

`organizationId` string

The organization ID to get. By default, it will use the active organization.

`organizationSlug` string

The organization slug to get.

`membersLimit` number

The limit of members to get. By default, it uses the membershipLimit option which defaults to 100.

### Update Organization

To update organization info, you can use `organization.update`

POST/organization/update

```
const { data, error } = await authClient.organization.update({
    data: { // required, A partial list of data to update the organization.
        name: "updated-name", // The name of the organization.
        slug: "updated-slug", // The slug of the organization.
        logo: "new-logo.url", // The logo of the organization.
        metadata: { customerId: "test" }, // The metadata of the organization.
    },
    organizationId: "org-id", // The organization ID. to update.
});
```

Parameters

`data` Objectrequired

A partial list of data to update the organization.

`name` string

The name of the organization.

`slug` string

The slug of the organization.

`logo` string | null

The logo of the organization.

`metadata` Record<string, any> | null

The metadata of the organization.

`organizationId` string

The organization ID. to update.

### Delete Organization

To remove user owned organization, you can use `organization.delete`

POST/organization/delete

```
const { data, error } = await authClient.organization.delete({
    organizationId: "org-id", // required, The organization ID to delete.
});
```

Parameters

`organizationId` stringrequired

The organization ID to delete.

If the user has the necessary permissions (by default: role is owner) in the specified organization, all members, invitations and organization information will be removed.

You can configure how organization deletion is handled through `organizationDeletion` option:

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

const auth = betterAuth({
  plugins: [
    organization({
      disableOrganizationDeletion: true, //to disable it altogether
      organizationHooks: {
        beforeDeleteOrganization: async (data, ctx) => {
          // a callback to run before deleting org
          // \`ctx\` is the endpoint context (e.g. \`ctx?.request\`)
        },
        afterDeleteOrganization: async (data, ctx) => {
          // a callback to run after deleting org
        },
      },
    }),
  ],
});
```

## Invitations

To add a member to an organization, we first need to send an invitation to the user. The user will receive an email/sms with the invitation link. Once the user accepts the invitation, they will be added to the organization.

### Setup Invitation Email

For member invitation to work we first need to provide `sendInvitationEmail` to the `better-auth` instance. This function is responsible for sending the invitation email to the user.

You'll need to construct and send the invitation link to the user. The link should include the invitation ID, which will be used with the acceptInvitation function when the user clicks on it.

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";
import { sendOrganizationInvitation } from "./email";

export const auth = betterAuth({
  plugins: [
    organization({
      async sendInvitationEmail(data) {
        const inviteLink = \`https://example.com/accept-invitation/${data.id}\`;
        sendOrganizationInvitation({
          email: data.email,
          invitedByUsername: data.inviter.user.name,
          invitedByEmail: data.inviter.user.email,
          teamName: data.organization.name,
          inviteLink,
        });
      },
    }),
  ],
});
```

### Send Invitation

To invite users to an organization, you can use the `invite` function provided by the client. The `invite` function takes an object with the following properties:

POST/organization/invite-member

```
const { data, error } = await authClient.organization.inviteMember({
    email: "example@gmail.com", // required, The email address of the user to invite.
    role: "member", // required, The role(s) to assign to the user. It can be \`admin\`, \`member\`, \`owner\`
    organizationId: "org-id", // The organization ID to invite the user to. Defaults to the active organization.
    resend: true, // Resend the invitation email, if the user is already invited.
    teamId: "team-id", // The team ID to invite the user to.
});
```

Parameters

`email` stringrequired

The email address of the user to invite.

`role` string | string\[\]required

The role(s) to assign to the user. It can be `admin`, `member`, `owner`

`organizationId` string

The organization ID to invite the user to. Defaults to the active organization.

`resend` boolean

Resend the invitation email, if the user is already invited.

`teamId` string

The team ID to invite the user to.

### Accept Invitation

When a user receives an invitation email, they can click on the invitation link to accept the invitation. The invitation link should include the invitation ID, which will be used to accept the invitation.

Make sure to call the `acceptInvitation` function after the user is logged in.

POST/organization/accept-invitation

```
const { data, error } = await authClient.organization.acceptInvitation({
    invitationId: "invitation-id", // required, The ID of the invitation to accept.
});
```

Parameters

`invitationId` stringrequired

The ID of the invitation to accept.

#### Email Verification Requirement

By default, accepting an invitation requires the invitation ID from the email link and a logged-in session whose email matches the invitation. When Better Auth uses built-in opaque invitation IDs, including the default generator or `advanced.database.generateId: "uuid"`, that remains enough for the normal emailed-invitation flow. If invitation IDs are externally controlled or predictable, such as `advanced.database.generateId: "serial"` / `false` or custom ID generation, Better Auth also requires verified email unless you explicitly set `requireEmailVerificationOnInvitation` to `false`.

Set `requireEmailVerificationOnInvitation` to `true` for the stricter posture. This is recommended when invitation IDs can be visible outside the invited user's mailbox, when your app exposes organization invitation lists to members, when you use custom invitation delivery, or when unverified email/password sessions are allowed and organization membership is sensitive. Requiring verified email before sign-in provides the same ownership proof earlier in the flow.

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    organization({
      requireEmailVerificationOnInvitation: true, 
      async sendInvitationEmail(data) {
        // ... your email sending logic
      },
    }),
  ],
});
```

### Cancel Invitation

If a user has sent out an invitation, you can use this method to cancel it.

If you're looking for how a user can reject an invitation, you can find that in the [reject invitation section](#reject-invitation).

POST/organization/cancel-invitation

```
await authClient.organization.cancelInvitation({
    invitationId: "invitation-id", // required, The ID of the invitation to cancel.
});
```

Parameters

`invitationId` stringrequired

The ID of the invitation to cancel.

### Reject Invitation

If this user has received an invitation, but wants to decline it, this method will allow you to do so by rejecting it.

POST/organization/reject-invitation

```
await authClient.organization.rejectInvitation({
    invitationId: "invitation-id", // required, The ID of the invitation to reject.
});
```

Parameters

`invitationId` stringrequired

The ID of the invitation to reject.

### Get Invitation

To get an invitation you can use the `organization.getInvitation` function provided by the client. You need to provide the invitation id as a query parameter.

GET/organization/get-invitation

```
const { data, error } = await authClient.organization.getInvitation({
    query: {
        id: "invitation-id", // required, The ID of the invitation to get.
    },
});
```

Parameters

`id` stringrequired

The ID of the invitation to get.

### List Invitations

To list all invitations for a given organization you can use the `listInvitations` function provided by the client.

GET/organization/list-invitations

```
const { data, error } = await authClient.organization.listInvitations({
    query: {
        organizationId: "organization-id", // An optional ID of the organization to list invitations for. If not provided, will default to the user's active organization.
    },
});
```

Parameters

`organizationId` string

An optional ID of the organization to list invitations for. If not provided, will default to the user's active organization.

The response includes invitation IDs. Treat those IDs as action-capable invitation links. If users who can list organization invitations should not be able to use those IDs with unverified recipient sessions, enable `requireEmailVerificationOnInvitation` or add app-level permission checks around this endpoint.

### List user invitations

To list all invitations for a given user you can use the `listUserInvitations` function provided by the client.

```
import { authClient } from "@/lib/auth-client"

const invitations = await authClient.organization.listUserInvitations();
```

Client-side `listUserInvitations` calls require the session user's email to be verified. This endpoint enumerates pending invitation IDs from the session email, so email string matching alone is not enough ownership proof.

On the server, you can pass the user's email address as a query parameter.

```
const invitations = await auth.api.listUserInvitations({
  query: {
    email: "user@example.com",
  },
});
```

## Members

### List Members

To list all members of an organization you can use the `listMembers` function.

GET/organization/list-members

```
const { data, error } = await authClient.organization.listMembers({
    query: {
        organizationId: "organization-id", // An optional organization ID to list members for. If not provided, will default to the user's active organization.
        limit: 100, // The limit of members to return.
        offset: 0, // The offset to start from.
        sortBy: "createdAt", // The field to sort by.
        sortDirection: "desc", // The direction to sort by.
        filterField: "createdAt", // The field to filter by.
        filterOperator: "eq", // The operator to filter by.
        filterValue: "value", // The value to filter by.
    },
});
```

Parameters

`organizationId` string

An optional organization ID to list members for. If not provided, will default to the user's active organization.

`limit` number

The limit of members to return.

`offset` number

The offset to start from.

`sortBy` string

The field to sort by.

`sortDirection` "asc" | "desc"

The direction to sort by.

`filterField` string

The field to filter by.

`filterOperator` "eq" | "ne" | "lt" | "lte" | "gt" | "gte" | "in" | "not\_in" | "contains" | "starts\_with" | "ends\_with"

The operator to filter by.

`filterValue` string | number | boolean | string\[\] | number\[\]

The value to filter by.

### Remove Member

To remove you can use `organization.removeMember`

POST/organization/remove-member

```
const { data, error } = await authClient.organization.removeMember({
    memberIdOrEmail: "user@example.com", // required, The ID or email of the member to remove.
    organizationId: "org-id", // The ID of the organization to remove the member from. If not provided, the active organization will be used.
});
```

Parameters

`memberIdOrEmail` stringrequired

The ID or email of the member to remove.

`organizationId` string

The ID of the organization to remove the member from. If not provided, the active organization will be used.

### Update Member Role

To update the role of a member in an organization, you can use the `organization.updateMemberRole`. If the user has the permission to update the role of the member, the role will be updated.

POST/organization/update-member-role

```
await authClient.organization.updateMemberRole({
    role: ["admin", "sale"], // required, The new role to be applied. This can be a string or array of strings representing the roles.
    memberId: "member-id", // required, The member id to apply the role update to.
    organizationId: "organization-id", // An optional organization ID which the member is a part of to apply the role update. If not provided, you must provide session headers to get the active organization.
});
```

Parameters

`role` string | string\[\]required

The new role to be applied. This can be a string or array of strings representing the roles.

`memberId` stringrequired

The member id to apply the role update to.

`organizationId` string

An optional organization ID which the member is a part of to apply the role update. If not provided, you must provide session headers to get the active organization.

### Get Active Member

To get the current member of the active organization you can use the `organization.getActiveMember` function. This function will return the user's member details in their active organization.

GET/organization/get-active-member

```
const { data: member, error } = await authClient.organization.getActiveMember();
```

### Get Active Member Role

To get the current role member of the active organization you can use the `organization.getActiveMemberRole` function. This function will return the user's member role in their active organization.

GET/organization/get-active-member-role

```
const { data: { role }, error } = await authClient.organization.getActiveMemberRole();
```

### Add Member

If you want to add a member directly to an organization without sending an invitation, you can use the `addMember` function which can only be invoked on the server.

```
const data = await auth.api.addMember({
    body: {
        userId: "user-id", // The user ID which represents the user to be added as a member. If \`null\` is provided, then it's expected to provide session headers.
        role: ["admin", "sale"], // required, The role(s) to assign to the new member.
        organizationId: "org-id", // An optional organization ID to pass. If not provided, will default to the user's active organization.
        teamId: "team-id", // An optional team ID to add the member to.
    },
});
```

Parameters

`userId` string | null

The user ID which represents the user to be added as a member. If `null` is provided, then it's expected to provide session headers.

`role` string | string\[\]required

The role(s) to assign to the new member.

`organizationId` string

An optional organization ID to pass. If not provided, will default to the user's active organization.

`teamId` string

An optional team ID to add the member to.

### Leave Organization

To leave organization you can use `organization.leave` function. This function will remove the current user from the organization.

POST/organization/leave

```
await authClient.organization.leave({
    organizationId: "organization-id", // required, The organization ID for the member to leave.
});
```

Parameters

`organizationId` stringrequired

The organization ID for the member to leave.

## Access Control

The organization plugin provides a very flexible access control system. You can control the access of the user based on the role they have in the organization. You can define your own set of permissions based on the role of the user.

### Roles

`owner`: The user who created the organization by default. The owner has full control over the organization and can perform any action.

`admin`: Users with the admin role have full control over the organization except for deleting the organization or changing the owner.

`member`: Users with the member role have limited control over the organization. They can only read organization data and have no permissions to create, update, or delete resources.

### Permissions

By default, there are three resources, and these have two to three actions.

**organization**:

`update` `delete`

**member**:

`create` `update` `delete`

**invitation**:

`create` `cancel`

The owner has full control over all the resources and actions. The admin has full control over all the resources except for deleting the organization or changing the owner. The member has no control over any of those actions other than reading the data.

### Custom Permissions

The plugin provides an easy way to define your own set of permissions for each role.

#### Create Access Control

You first need to create access controller by calling `createAccessControl` function and passing the statement object. The statement object should have the resource name as the key and the array of actions as the value.

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

const statement = {
    project: ["create", "share", "update", "delete"],
} as const;

const ac = createAccessControl(statement);

const member = ac.newRole({ 
    project: ["create"], 
}); 

const admin = ac.newRole({ 
    project: ["create", "update"], 
}); 

const owner = ac.newRole({ 
    project: ["create", "update", "delete"], 
}); 

const myCustomRole = ac.newRole({ 
    project: ["create", "update", "delete"], 
    organization: ["update"], 
});
```

When you create custom roles for existing roles, the predefined permissions for those roles will be overridden. To add the existing permissions to the custom role, you need to import `defaultStatements` and merge it with your new statement, plus merge the roles' permissions set with the default roles.

```
import { createAccessControl } from "better-auth/plugins/access";
import { defaultStatements, adminAc } from 'better-auth/plugins/organization/access'

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

Once you have created the roles you can pass them to the organization plugin both on the client and the server.

```
import { betterAuth } from "better-auth"
import { organization } from "better-auth/plugins"
import { ac, owner, admin, member } from "@/auth/permissions"

export const auth = betterAuth({
    plugins: [
        organization({
            ac,
            roles: {
                owner,
                admin,
                member,
                myCustomRole
            }
        }),
    ],
});
```

You also need to pass the access controller and the roles to the client plugin.

```
import { createAuthClient } from "better-auth/client"
import { organizationClient } from "better-auth/client/plugins"
import { ac, owner, admin, member, myCustomRole } from "@/auth/permissions"

export const authClient = createAuthClient({
    plugins: [
        organizationClient({
            ac,
            roles: {
                owner,
                admin,
                member,
                myCustomRole
            }
        })
  ]
})
```

### Access Control Usage

**Has Permission**:

You can use the `hasPermission` action provided by the `api` to check the permission of the user.

```
import { auth } from "@/lib/auth"

await auth.api.hasPermission({
  headers: await headers(),
  body: {
    permissions: {
      project: ["create"], // This must match the structure in your access control
    },
  },
});

// You can also check multiple resource permissions at the same time
await auth.api.hasPermission({
  headers: await headers(),
  body: {
    permissions: {
      project: ["create"], // This must match the structure in your access control
      sale: ["create"],
    },
  },
});
```

If you want to check the permission of the user on the client from the server you can use the `hasPermission` function provided by the client.

```
const canCreateProject = await authClient.organization.hasPermission({
  permissions: {
    project: ["create"],
  },
});

// You can also check multiple resource permissions at the same time
const canCreateProjectAndCreateSale =
  await authClient.organization.hasPermission({
    permissions: {
      project: ["create"],
      sale: ["create"],
    },
  });
```

**Check Role Permission**:

Once you have defined the roles and permissions to avoid checking the permission from the server you can use the `checkRolePermission` function provided by the client.

```
import { authClient } from "@/lib/auth-client"

const canCreateProject = authClient.organization.checkRolePermission({
  permissions: {
    organization: ["delete"],
  },
  role: "admin",
});

// You can also check multiple resource permissions at the same time
const canCreateProjectAndCreateSale =
  authClient.organization.checkRolePermission({
    permissions: {
      organization: ["delete"],
      member: ["delete"],
    },
    role: "admin",
  });
```

---

## Dynamic Access Control

Dynamic access control allows you to create roles at runtime for organizations. This is achieved by storing the created roles and permissions associated with an organization in a database table.

### Enabling Dynamic Access Control

To enable dynamic access control, pass the `dynamicAccessControl` configuration option with `enabled` set to `true` to both server and client plugins.

Ensure you have pre-defined an `ac` instance on the server auth plugin. This is important as this is how we can infer the permissions that are available for use.

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";
import { ac } from "@/auth/permissions";

export const auth = betterAuth({
    plugins: [
        organization({ 
            ac, // Must be defined in order for dynamic access control to work
            dynamicAccessControl: { 
              enabled: true, 
            }, 
        }) 
    ]
})
```

```
import { createAuthClient } from "better-auth/client";
import { organizationClient } from "better-auth/client/plugins";

export const authClient = createAuthClient({
    plugins: [
        organizationClient({ 
            dynamicAccessControl: { 
              enabled: true, 
            }, 
        }) 
    ]
})
```

### Creating a role

To create a new role for an organization at runtime, you can use the `createRole` function.

Only users with roles which contain the `ac` resource with the `create` permission can create a new role. By default, only the `admin` and `owner` roles have this permission. You also cannot add permissions that your current role in that organization can't already access.

POST/organization/create-role

```
// To use custom resources or permissions,
// make sure they are defined in the \`ac\` instance of your organization config.
const permission = {
  project: ["create", "update", "delete"]
}
await authClient.organization.createRole({
    role: "my-unique-role", // required, A unique name of the role to create.
    permission: permission, // The permissions to assign to the role.
    organizationId: "organization-id", // The organization ID which the role will be created in. Defaults to the active organization.
});
```

Parameters

`role` stringrequired

A unique name of the role to create.

`permission` Record<string, string\[\]>

The permissions to assign to the role.

`organizationId` string

The organization ID which the role will be created in. Defaults to the active organization.

Now you can freely call [`updateMemberRole`](#update-member-role) to update the role of a member with your newly created role!

### Deleting a role

To delete a role, you can use the `deleteRole` function, then provide either a `roleName` or `roleId` parameter along with the `organizationId` parameter.

POST/organization/delete-role

```
await authClient.organization.deleteRole({
    roleName: "my-role", // The name of the role to delete. Alternatively, you can pass a \`roleId\` parameter instead.
    roleId: "role-id", // The id of the role to delete. Alternatively, you can pass a \`roleName\` parameter instead.
    organizationId: "organization-id", // The organization ID which the role will be deleted in. Defaults to the active organization.
});
```

Parameters

`roleName` string

The name of the role to delete. Alternatively, you can pass a `roleId` parameter instead.

`roleId` string

The id of the role to delete. Alternatively, you can pass a `roleName` parameter instead.

`organizationId` string

The organization ID which the role will be deleted in. Defaults to the active organization.

### Listing roles

To list roles, you can use the `listOrgRoles` function. This requires the `ac` resource with the `read` permission for the member to be able to list roles.

GET/organization/list-roles

```
const { data: roles, error } = await authClient.organization.listRoles({
    query: {
        organizationId: "organization-id", // The organization ID which the roles are under to list. Defaults to the user's active organization.
    },
});
```

Parameters

`organizationId` string

The organization ID which the roles are under to list. Defaults to the user's active organization.

### Getting a specific role

To get a specific role, you can use the `getOrgRole` function and pass either a `roleName` or `roleId` parameter. This requires the `ac` resource with the `read` permission for the member to be able to get a role.

GET/organization/get-role

```
const { data: role, error } = await authClient.organization.getRole({
    query: {
        roleName: "my-role", // The name of the role to get. Alternatively, you can pass a \`roleId\` parameter instead.
        roleId: "role-id", // The id of the role to get. Alternatively, you can pass a \`roleName\` parameter instead.
        organizationId: "organization-id", // The organization ID from which the role will be retrieved. Defaults to the active organization.
    },
});
```

Parameters

`roleName` string

The name of the role to get. Alternatively, you can pass a `roleId` parameter instead.

`roleId` string

The id of the role to get. Alternatively, you can pass a `roleName` parameter instead.

`organizationId` string

The organization ID from which the role will be retrieved. Defaults to the active organization.

### Updating a role

To update a role, you can use the `updateOrgRole` function and pass either a `roleName` or `roleId` parameter.

POST/organization/update-role

```
const { data: updatedRole, error } = await authClient.organization.updateRole({
    roleName: "my-role", // The name of the role to update. Alternatively, you can pass a \`roleId\` parameter instead.
    roleId: "role-id", // The id of the role to update. Alternatively, you can pass a \`roleName\` parameter instead.
    organizationId: "organization-id", // The organization ID which the role will be updated in. Defaults to the active organization.
    data: { // required, The data which will be updated
        permission: { project: ["create", "update", "delete"] }, // Optionally update the permissions of the role.
        roleName: "my-new-role", // Optionally update the name of the role.
    },
});
```

Parameters

`roleName` string

The name of the role to update. Alternatively, you can pass a `roleId` parameter instead.

`roleId` string

The id of the role to update. Alternatively, you can pass a `roleName` parameter instead.

`organizationId` string

The organization ID which the role will be updated in. Defaults to the active organization.

`data` Objectrequired

The data which will be updated

`permission` Record<string, string\[\]>

Optionally update the permissions of the role.

`roleName` string

Optionally update the name of the role.

### Configuration Options

Below is a list of options that can be passed to the `dynamicAccessControl` object.

#### enabled

This option is used to enable or disable dynamic access control. By default, it is disabled.

```
organization({
  dynamicAccessControl: {
    enabled: true
  }
})
```

#### maximumRolesPerOrganization

This option is used to limit the number of roles that can be created for an organization.

By default, the maximum number of roles that can be created for an organization is infinite.

```
organization({
  dynamicAccessControl: {
    maximumRolesPerOrganization: 10
  }
})
```

You can also pass a function that returns a number.

```
organization({
  dynamicAccessControl: {
    maximumRolesPerOrganization: async (organizationId) => { 
      const organization = await getOrganization(organizationId); 
      return organization.plan === "pro" ? 100 : 10; 
    } 
  }
})
```

### Additional Fields

To add additional fields to the `organizationRole` table, you can pass the `additionalFields` configuration option to the `organization` plugin.

```
organization({
  schema: {
    organizationRole: {
      additionalFields: {
        // Role colors!
        color: {
          type: "string",
          defaultValue: "#ffffff",
        },
        //... other fields
      },
    },
  },
})
```

Then, if you don't already use `inferOrgAdditionalFields` to infer the additional fields, you can use it to infer the additional fields.

```
import { createAuthClient } from "better-auth/client"
import { organizationClient, inferOrgAdditionalFields } from "better-auth/client/plugins"
import type { auth } from "@/lib/auth" // import the auth object type only

export const authClient = createAuthClient({
    plugins: [
        organizationClient({
            schema: inferOrgAdditionalFields<typeof auth>()
        })
    ]
})
```

Otherwise, you can pass the schema values directly, the same way you do on the org plugin in the server.

```
import { createAuthClient } from "better-auth/client"
import { organizationClient } from "better-auth/client/plugins"

export const authClient = createAuthClient({
    plugins: [
        organizationClient({
            schema: {
                organizationRole: {
                    additionalFields: {
                        color: {
                            type: "string",
                            defaultValue: "#ffffff",
                        }
                    }
                }
            }
        })
    ]
})
```

---

## Teams

Teams allow you to group members within an organization. The teams feature provides additional organization structure and can be used to manage permissions at a more granular level.

### Enabling Teams

To enable teams, pass the `teams` configuration option to both server and client plugins:

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    organization({
      teams: {
        enabled: true,
        maximumTeams: 10, // Optional: limit teams per organization
        allowRemovingAllTeams: false, // Optional: prevent removing the last team
      },
    }),
  ],
});
```

```
import { createAuthClient } from "better-auth/client";
import { organizationClient } from "better-auth/client/plugins";

export const authClient = createAuthClient({
  plugins: [
    organizationClient({
      teams: {
        enabled: true,
      },
    }),
  ],
});
```

### Managing Teams

#### Create Team

Create a new team within an organization:

POST/organization/create-team

```
const { data, error } = await authClient.organization.createTeam({
    name: "my-team", // required, The name of the team.
    organizationId: "organization-id", // The organization ID which the team will be created in. Defaults to the active organization.
});
```

Parameters

`name` stringrequired

The name of the team.

`organizationId` string

The organization ID which the team will be created in. Defaults to the active organization.

#### List Teams

Get all teams in an organization:

GET/organization/list-teams

```
const { data, error } = await authClient.organization.listTeams({
    query: {
        organizationId: "organization-id", // The organization ID which the teams are under to list. Defaults to the user's active organization.
    },
});
```

Parameters

`organizationId` string

The organization ID which the teams are under to list. Defaults to the user's active organization.

#### Update Team

Update a team's details:

POST/organization/update-team

```
const { data, error } = await authClient.organization.updateTeam({
    teamId: "team-id", // required, The ID of the team to be updated.
    data: { // required, A partial object containing options for you to update.
        name: "My new team name", // The name of the team to be updated.
        organizationId: "My new organization ID for this team", // The organization ID which the team falls under.
        createdAt: new Date(), // The timestamp of when the team was created.
        updatedAt: new Date(), // The timestamp of when the team was last updated.
    },
});
```

Parameters

`teamId` stringrequired

The ID of the team to be updated.

`data` Objectrequired

A partial object containing options for you to update.

`name` string

The name of the team to be updated.

`organizationId` string

The organization ID which the team falls under.

`createdAt` Date

The timestamp of when the team was created.

`updatedAt` Date

The timestamp of when the team was last updated.

#### Remove Team

Delete a team from an organization:

POST/organization/remove-team

```
const { data, error } = await authClient.organization.removeTeam({
    teamId: "team-id", // required, The team ID of the team to remove.
    organizationId: "organization-id", // The organization ID which the team falls under. If not provided, it will default to the user's active organization.
});
```

Parameters

`teamId` stringrequired

The team ID of the team to remove.

`organizationId` string

The organization ID which the team falls under. If not provided, it will default to the user's active organization.

#### Set Active Team

Sets the given team as the current active team for the current active organization. If `teamId` is `null` the current active team is unset.

POST/organization/set-active-team

```
const { data, error } = await authClient.organization.setActiveTeam({
    teamId: "team-id", // The team ID of the team to set as the current active team. The team must belong to the current active organization.
});
```

Parameters

`teamId` string | null

The team ID of the team to set as the current active team. The team must belong to the current active organization.

#### List User Teams

List all teams that a user is a part of. Defaults to the current user and returns teams across every organization the user belongs to.

- Pass `userId` to list teams for another member. This is gated behind the `member:update` permission in the target organization.
- Pass `organizationId` to scope the result to a single organization without having to switch the session's active organization. When omitted, queries for another user use the session's active organization.

GET/organization/list-user-teams

```
const { data, error } = await authClient.organization.listUserTeams({
    query: {
        userId, // The user ID to list teams for. Defaults to the current session user.
        organizationId, // The organization ID to scope the team list to. When omitted on a self-query, teams are returned across every organization the user belongs to. When querying another user, falls back to the session's active organization.
    },
});
```

Parameters

`userId` string

The user ID to list teams for. Defaults to the current session user.

`organizationId` string

The organization ID to scope the team list to. When omitted on a self-query, teams are returned across every organization the user belongs to. When querying another user, falls back to the session's active organization.

#### List Team Members

List the members of the given team.

POST/organization/list-team-members

```
const { data, error } = await authClient.organization.listTeamMembers({
    query: {
        teamId: "team-id", // The team whose members we should return. If this is not provided the members of the current active team get returned.
    },
});
```

Parameters

`teamId` string

The team whose members we should return. If this is not provided the members of the current active team get returned.

#### Add Team Member

Add a member to a team.

POST/organization/add-team-member

```
const { data, error } = await authClient.organization.addTeamMember({
    teamId: "team-id", // required, The team the user should be a member of.
    userId: "user-id", // required, The user ID which represents the user to be added as a member.
});
```

Parameters

`teamId` stringrequired

The team the user should be a member of.

`userId` stringrequired

The user ID which represents the user to be added as a member.

#### Remove Team Member

Remove a member from a team.

POST/organization/remove-team-member

```
const { data, error } = await authClient.organization.removeTeamMember({
    teamId: "team-id", // required, The team the user should be removed from.
    userId: "user-id", // required, The user which should be removed from the team.
});
```

Parameters

`teamId` stringrequired

The team the user should be removed from.

`userId` stringrequired

The user which should be removed from the team.

### Team Permissions

Teams follow the organization's permission system. To manage teams, users need the following permissions:

- `team:create` - Create new teams
- `team:update` - Update team details
- `team:delete` - Remove teams

By default:

- Organization owners and admins can manage teams
- Regular members cannot create, update, or delete teams

### Team Configuration Options

The teams feature supports several configuration options:

- `maximumTeams`: Limit the number of teams per organization
	```
	teams: {
	  enabled: true,
	  maximumTeams: 10 // Fixed number
	  // OR
	  maximumTeams: async ({ organizationId, session }, ctx) => {
	    // Dynamic limit based on organization plan
	    const plan = await getPlan(organizationId)
	    return plan === 'pro' ? 20 : 5
	  },
	  maximumMembersPerTeam: 10 // Fixed number
	  // OR
	  maximumMembersPerTeam: async ({ teamId, session, organizationId }, ctx) => {
	    // Dynamic limit based on team plan
	    const plan = await getPlan(organizationId, teamId)
	    return plan === 'pro' ? 50 : 10
	  },
	}
	```
	`maximumMembersPerTeam` is enforced when accepting team invitations, when adding an existing organization member to a team, and when adding a new organization member with a `teamId`.
- `allowRemovingAllTeams`: Control whether the last team can be removed
	```
	teams: {
	  enabled: true,
	  allowRemovingAllTeams: false // Prevent removing the last team
	}
	```

### Team Members

When inviting members to an organization, you can specify a team:

```
await authClient.organization.inviteMember({
  email: "user@example.com",
  role: "member",
  teamId: "team-id",
});
```

The invited member will be added to the specified team upon accepting the invitation.

### Database Schema

When teams are enabled, new `team` and `teamMember` tables are added to the database.

Table Name: `team`

Table

Field

Type

Attributes

Description

id

string

PK

Unique identifier for each team

name

string

\-

The name of the team

memberCount

number

\-

Durable member count used to enforce team capacity

organizationId

string

FK

The ID of the organization

createdAt

Date

\-

Timestamp of when the team was created

updatedAt?

Date

\-

Timestamp of when the team was created

Table Name: `teamMember`

Table

Field

Type

Attributes

Description

id

string

PK

Unique identifier for each team member

teamId

string

FK

Unique identifier for each team

userId

string

FK

The ID of the user

membershipKey?

string

\-

Internal unique key for the team and user membership pair

createdAt?

Date

\-

Timestamp of when the team member was created

## Schema

The organization plugin adds the following tables to the database:

### Organization

Table Name: `organization`

Table

Field

Type

Attributes

Description

id

string

PK

Unique identifier for each organization

name

string

\-

The name of the organization

slug

string

UQ

The slug of the organization

logo?

string

\-

The logo of the organization

metadata?

string

\-

Additional metadata for the organization

createdAt

Date

\-

Timestamp of when the organization was created

### Member

Table Name: `member`

Table

Field

Type

Attributes

Description

id

string

PK

Unique identifier for each member

userId

string

FK

The ID of the user

organizationId

string

FK

The ID of the organization

role

string

\-

The role of the user in the organization

createdAt

Date

\-

Timestamp of when the member was added to the organization

### Invitation

Table Name: `invitation`

Table

Field

Type

Attributes

Description

id

string

PK

Unique identifier for each invitation

email

string

\-

The email address of the user

inviterId

string

FK

The ID of the inviter

organizationId

string

FK

The ID of the organization

role?

string

\-

The role of the user in the organization

status

string

\-

The status of the invitation

createdAt

Date

\-

Timestamp of when the invitation was created

expiresAt

Date

\-

Timestamp of when the invitation expires

If teams are enabled, you need to add the following fields to the invitation table:

Table

Field

Type

Attributes

Description

teamId?

string

\-

The ID of the team

### Session

Table Name: `session`

You need to add two more fields to the session table to store the active organization ID and the active team ID.

Table

Field

Type

Attributes

Description

activeOrganizationId?

string

\-

The ID of the active organization

activeTeamId?

string

\-

The ID of the active team

### Organization Role (optional)

Table Name: `organizationRole`

Table

Field

Type

Attributes

Description

id

string

\-

Unique identifier for each organization role

organizationId

string

FK

The ID of the organization

role

string

\-

The name of the role

permission

string

\-

The permission of the role

createdAt

Date

\-

Timestamp of when the organization role was created

updatedAt?

Date

\-

Timestamp of when the organization role was updated

### Teams (optional)

Table Name: `team`

Table

Field

Type

Attributes

Description

id

string

PK

Unique identifier for each team

name

string

\-

The name of the team

memberCount

number

\-

Durable member count used to enforce team capacity

organizationId

string

FK

The ID of the organization

createdAt

Date

\-

Timestamp of when the team was created

updatedAt?

Date

\-

Timestamp of when the team was created

Table Name: `teamMember`

Table

Field

Type

Attributes

Description

id

string

PK

Unique identifier for each team member

teamId

string

FK

Unique identifier for each team

userId

string

FK

The ID of the user

membershipKey?

string

\-

Internal unique key for the team and user membership pair

createdAt?

Date

\-

Timestamp of when the team member was created

Table Name: `invitation`

Table

Field

Type

Attributes

Description

teamId?

string

\-

The ID of the team

### Customizing the Schema

To change the schema table name or fields, you can pass `schema` option to the organization plugin.

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

const auth = betterAuth({
  plugins: [
    organization({
      schema: {
        organization: {
          modelName: "organizations", //map the organization table to organizations
          fields: {
            name: "title", //map the name field to title
          },
          additionalFields: {
            // Add a new field to the organization table
            myCustomField: {
              type: "string",
              input: true,
              required: false,
            },
          },
        },
      },
    }),
  ],
});
```

#### Additional Fields

Starting with [Better Auth v1.3](https://github.com/better-auth/better-auth/releases/tag/v1.3.0), you can easily add custom fields to the `organization`, `invitation`, `member`, and `team` tables.

When you add extra fields to a model, the relevant API endpoints will automatically accept and return these new properties. For instance, if you add a custom field to the `organization` table, the `createOrganization` endpoint will include this field in its request and response payloads as needed.

```
import { betterAuth } from "better-auth";
import { organization } from "better-auth/plugins";

const auth = betterAuth({
  plugins: [
    organization({
      schema: {
        organization: {
          additionalFields: {
            myCustomField: {
              type: "string", 
              input: true, 
              required: false, 
            }, 
          },
        },
      },
    }),
  ],
});
```

For inferring the additional fields, you can use the `inferOrgAdditionalFields` function. This function will infer the additional fields from the auth object type.

```
import { createAuthClient } from "better-auth/client";
import {
  inferOrgAdditionalFields,
  organizationClient,
} from "better-auth/client/plugins";
import type { auth } from "@/lib/auth" // import the auth object type only

const client = createAuthClient({
  plugins: [
    organizationClient({
      schema: inferOrgAdditionalFields<typeof auth>(),
    }),
  ],
});
```

if you can't import the auth object type, you can use the `inferOrgAdditionalFields` function without the generic. This function will infer the additional fields from the schema object.

```
import { createAuthClient } from "better-auth/client";
import {
  inferOrgAdditionalFields,
  organizationClient,
} from "better-auth/client/plugins";

const client = createAuthClient({
  plugins: [
    organizationClient({
      schema: inferOrgAdditionalFields({
        organization: {
          additionalFields: {
            newField: {
              type: "string", 
            }, 
          },
        },
      }),
    }),
  ],
});
```

#### Example usage

```
await client.organization.create({
  name: "Test",
  slug: "test",
  newField: "123", //this should be allowed
  //@ts-expect-error - this field is not available
  unavailableField: "123", //this should be not allowed
});
```

## Options

**allowUserToCreateOrganization**: `boolean` | `((user: User) => Promise<boolean> | boolean)` - A function that determines whether a user can create an organization. By default, it's `true`. You can set it to `false` to restrict users from creating organizations.

**organizationLimit**: `number` | `((user: User) => Promise<boolean> | boolean)` - The maximum number of organizations allowed for a user. By default, it's `unlimited`. You can set it to any number you want, or a function that returns a boolean. **If you provide a function, it should return `true` if the user has reached their organization limit (blocking further creation), or `false` if they have not reached their limit (allowing further creation).**

**creatorRole**: `admin | owner` - The role of the user who creates the organization. By default, it's `owner`. You can set it to `admin`.

**membershipLimit**: `number` | `((user: User, organization: Organization) => Promise<number> | number)` - The maximum number of members allowed in an organization. By default, it's `100`. You can set it to any number you want or a function that returns the limit number.

**sendInvitationEmail**: `async (data) => Promise<void>` - A function that sends an invitation email to the user.

**invitationExpiresIn**: `number` - How long the invitation link is valid for in seconds. By default, it's 48 hours (2 days).

**cancelPendingInvitationsOnReInvite**: `boolean` - Whether to cancel pending invitations if the user is already invited to the organization. By default, it's `false`.

**invitationLimit**: `number` | `((user: User) => Promise<boolean> | boolean)` - The maximum number of invitations allowed for a user. By default, it's `100`. You can set it to any number you want or a function that returns a boolean.

**requireEmailVerificationOnInvitation**: `boolean | undefined` - Whether to require email verification before recipient invitation calls that carry an invitation ID (`acceptInvitation`, `rejectInvitation`, `getInvitation`). When unset, Better Auth preserves the normal emailed-invitation flow for built-in opaque invitation IDs, including the default generator and `advanced.database.generateId: "uuid"`. It requires verification for externally controlled or predictable invitation IDs, such as `advanced.database.generateId: "serial"` / `false` or custom ID generation. Set this option to `true` when invitation IDs may be visible outside the invited user's mailbox, when organization invitation lists are exposed to members, or when you want verified email to be the ownership proof for by-ID invitation actions. Client-side `listUserInvitations` always requires a verified session email because it enumerates invitation IDs from the session email.
