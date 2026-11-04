---
url: https://better-auth.com/llms.txt/docs/plugins/sso
title: "Sso"
description: ""
access_date: 2026-11-04T23:37:22.575Z
current_date: 2026-11-04T23:37:22.575Z
---

Connect each organization to its own OIDC or SAML 2.0 identity provider.

The SSO plugin lets your users sign in through identity providers that you or your customers register at runtime, over [OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) or [SAML 2.0](https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf). Each registered connection is one row in the `ssoProvider` table. You can also [define connections in code](#configure-providers-in-code).

For a fixed set of providers configured in code, use [Generic OAuth](https://better-auth.com/docs/plugins/generic-oauth) or [social providers](https://better-auth.com/docs/concepts/oauth). To let other apps sign in with yours, use [OAuth Provider](https://better-auth.com/docs/plugins/oauth-provider).

## Features

- [OIDC discovery](https://better-auth.com/docs/plugins/sso/oidc#oidc-discovery) of provider endpoints
- [`private_key_jwt`](https://better-auth.com/docs/plugins/sso/oidc#token-endpoint-authentication) token endpoint authentication
- SAML 2.0 [SP-initiated](https://better-auth.com/docs/plugins/sso/saml#register-a-saml-provider) and [IdP-initiated](https://better-auth.com/docs/plugins/sso/saml#idp-initiated-sso) sign-in
- [SAML response validation](https://better-auth.com/docs/plugins/sso/saml-security): signatures, algorithms, timestamps, audience, and replay
- SAML [Single Logout](https://better-auth.com/docs/plugins/sso/saml#single-logout)
- [User resolution](https://better-auth.com/docs/plugins/sso/provisioning#resolve-sso-users) and provisioning
- [Organization provisioning](https://better-auth.com/docs/plugins/sso/provisioning#organization-provisioning)
- DNS [domain verification](https://better-auth.com/docs/plugins/sso/domain-verification#enable-domain-verification)

## Installation

### Install the plugin

#### npm

```
npm install @better-auth/sso
```

#### pnpm

#### yarn

#### bun

### Add the plugin to the server

```
import { betterAuth } from "better-auth";
import { sso } from "@better-auth/sso"; 

export const auth = betterAuth({
  plugins: [
    sso(), 
  ],
});
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

See the [Schema](https://better-auth.com/docs/plugins/sso/reference#schema) section to add the fields manually.

### Add the client plugin

`ssoClient()` takes the same `domainVerification` and `schema.ssoProvider.additionalFields` options as the server plugin, so the client types match.

```
import { createAuthClient } from "better-auth/client";
import { ssoClient } from "@better-auth/sso/client"; 

export const authClient = createAuthClient({
  plugins: [
    ssoClient(), 
  ],
});
```

## Register a provider

By default, any signed-in user can register up to 10 providers. Before you deploy, decide who should register providers:

| Who registers providers | Configure |
| --- | --- |
| Only your administrators | Set `providersLimit` to a function that returns `0` for everyone else. Owners and admins of an organization can still update its linked providers, including their `domain`; block that with [`guardProviderMutation`](https://better-auth.com/docs/plugins/sso/reference#plugin-options). |
| Your customers, each for their own domain | Enable [domain verification](https://better-auth.com/docs/plugins/sso/domain-verification). A provider then cannot sign anyone in until its domain is proven over DNS. |
| Nobody at runtime | Set `providersLimit: 0` and define the providers in [`defaultSSO`](#configure-providers-in-code). To link existing users by email, also enable domain verification. |

As a function, `providersLimit` receives the signed-in user and returns the number of providers that user can register. A limit of `0` rejects that user's registrations with 403.

```
sso({
  providersLimit: async (user) => ((await isAdmin(user.id)) ? 10 : 0), // your check
})
```

To register a provider, call `authClient.sso.register`. It calls `POST /sso/register` and creates a provider owned by the signed-in user. On the server, call `auth.api.registerSSOProvider` with the request's headers. Every registration needs:

- `providerId`: unique. The IDs of built-in providers, your configured social providers, `account.accountLinking.trustedProviders` entries, and `defaultSSO` entries are rejected with 422.
- `issuer`: the IdP issuer URL for OIDC, or your SP entity ID for SAML.
- `domain`: a bare email domain such as `example.com`. Separate several domains with commas.
- `oidcConfig`, `samlConfig`, or both.

To register an OIDC provider that uses discovery, first add the IdP's origin to `trustedOrigins`. Registration fetches the IdP's discovery document and fails with `discovery_untrusted_origin` when the origin is missing. `trustedOrigins` applies to the whole app, so list only IdP origins you have vetted. With `skipDiscovery: true` and every endpoint set in `oidcConfig`, registration fetches nothing. See [Trusted origins](https://better-auth.com/docs/plugins/sso/oidc#trusted-origins).

The response includes `redirectURI`, and `domainVerificationToken` when domain verification is enabled.

```
import { authClient } from "@/lib/auth-client";

const { data, error } = await authClient.sso.register({
  providerId: "acme",
  issuer: "https://idp.example.com",
  domain: "example.com",
  oidcConfig: {
    clientId: "CLIENT_ID",
    clientSecret: "CLIENT_SECRET",
  },
});

if (error) {
  // Show error.message. Discovery failures set error.code, such as discovery_untrusted_origin.
} else {
  // Register data.redirectURI as the redirect URI at the IdP.
}
```

For every field, see [Register an OIDC provider](https://better-auth.com/docs/plugins/sso/oidc#register-an-oidc-provider) and [Register a SAML provider](https://better-auth.com/docs/plugins/sso/saml#register-a-saml-provider).

### Configure providers in code

To keep providers in your config instead of the database, pass `defaultSSO`, an array of `{ providerId, domain, oidcConfig?, samlConfig?, privateKey? }` entries. Better Auth checks them before database providers. List only domains the IdP is authoritative for.

Whether a sign-in through these providers links to an existing user depends on domain verification:

- With `domainVerification.enabled: true`, they count as verified without a DNS check. A sign-in can link to an existing user at a listed domain when the other [linking conditions](https://better-auth.com/docs/plugins/sso/provisioning#how-sso-users-are-matched) hold.
- Without it, Better Auth does not link them by email. An existing user's first SSO sign-in then fails unless [`resolveUser`](https://better-auth.com/docs/plugins/sso/provisioning#resolve-sso-users) links it.

```
sso({
  defaultSSO: [
    {
      providerId: "acme-saml",
      domain: "example.com",
      samlConfig: {
        issuer: "https://app.example.com",
        entryPoint: "https://idp.example.com/sso",
        idpMetadata: {
          entityID: "https://idp.example.com",
          cert: "-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----",
        },
      },
    },
  ],
})
```

To use [`private_key_jwt`](https://better-auth.com/docs/plugins/sso/oidc#token-endpoint-authentication) without storing the key in the database, set `privateKey` (`{ privateKeyJwk?, privateKeyPem? }`) on the entry.

## Sign in

To sign in with an SSO provider, call `signIn.sso`.

```
import { authClient } from "@/lib/auth-client";

await authClient.signIn.sso({
  email: "user@example.com",
  callbackURL: "/dashboard",
});
```

In the browser, the auth client sends the user to the IdP by itself. On the server, call `auth.api.signInSSO`. It returns the IdP's `url`, and your code must send the browser there.

Pass the identifier you have:

- The user's email, as `email`, or their email domain, as `domain`.
- The organization's slug, as `organizationSlug`.
- The provider's ID, as `providerId`.

To choose the provider, Better Auth tries these in order:

1. If you pass `providerId`, it uses the provider with that ID. No other field is used.
2. If a `defaultSSO` entry's domains include `domain`, or the domain part of `email`, it uses that entry.
3. If you pass `organizationSlug`, it uses the provider linked to that organization. This needs the [organization plugin](https://better-auth.com/docs/plugins/organization). An unknown slug continues to the next step. A known organization without a linked provider returns 404.
4. Otherwise, it uses a registered provider for that domain.

With domain verification enabled, a provider whose domain is not verified returns 401.

Other sign-in fields:

- `loginHint` is sent to OIDC providers as [`login_hint`](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest), and falls back to `email`. SAML ignores it.
- `providerType` picks the protocol when a provider has both configurations, and OIDC is the default. Requesting a protocol the provider does not have returns 400.
- `additionalParams` adds parameters to the OIDC authorization request. Reserved parameters such as `state`, `client_id`, and `redirect_uri` are rejected, and SAML providers reject it with 400.
- To create a new user even when `disableImplicitSignUp` is on, pass `requestSignUp: true`. Any caller can set it, and SAML ignores it. To refuse users you have not provisioned, see [Block new users](https://better-auth.com/docs/plugins/sso/provisioning#block-new-users).

POST/sign-in/sso

```
const { data, error } = await authClient.signIn.sso({
    email: "user@example.com", // Email address whose domain identifies the provider.
    organizationSlug: "example-org", // Slug of the organization whose provider to use.
    providerId: "acme", // ID of the provider to use.
    domain: "example.com", // Email domain of the provider.
    callbackURL: "/dashboard", // required, URL to redirect to after a successful sign-in.
    errorCallbackURL: "/sign-in/error", // URL to redirect to if sign-in fails.
    newUserCallbackURL: "/welcome", // URL to redirect to when the user is new.
    scopes: ["openid", "email", "profile", "offline_access"], // OIDC scopes to request. Defaults to the provider's scopes, then \`["openid", "email", "profile", "offline_access"]\`.
    loginHint: "user@example.com", // Sent to OIDC providers as \`login_hint\`.
    additionalParams, // Extra OIDC authorization request parameters. Rejected for SAML.
    requestSignUp: true, // Create a new user even when \`disableImplicitSignUp\` is on. OIDC only.
    providerType: "oidc", // Protocol to use when the provider has both configurations. Defaults to \`oidc\`.
});
```

Parameters

`email` string

Email address whose domain identifies the provider.

`organizationSlug` string

Slug of the organization whose provider to use.

`providerId` string

ID of the provider to use.

`domain` string

Email domain of the provider.

`callbackURL` stringrequired

URL to redirect to after a successful sign-in.

`errorCallbackURL` string

URL to redirect to if sign-in fails.

`newUserCallbackURL` string

URL to redirect to when the user is new.

`scopes` string\[\]

OIDC scopes to request. Defaults to the provider's scopes, then `["openid", "email", "profile", "offline_access"]`.

`loginHint` string

Sent to OIDC providers as `login_hint`.

`additionalParams` Record<string, string>

Extra OIDC authorization request parameters. Rejected for SAML.

`requestSignUp` boolean

Create a new user even when `disableImplicitSignUp` is on. OIDC only.

`providerType` "oidc" | "saml"

Protocol to use when the provider has both configurations. Defaults to `oidc`.

## Manage providers

To list the providers you can manage, call `authClient.sso.providers()` (`GET /sso/providers`). It returns the providers the signed-in user registered without an organization, plus the providers of organizations where that user is an `owner` or `admin`. To fetch one, call `authClient.sso.getProvider({ query: { providerId } })` (`GET /sso/get-provider`). Both return a sanitized configuration: `clientIdLastFour` instead of the client ID, no secrets, and the parsed SAML signing certificates. You need the same access to update or delete a provider.

POST/sso/update-provider

```
const { data, error } = await authClient.sso.updateProvider({
    providerId: "acme", // required, ID of the provider to update.
    issuer: "https://idp.example.com", // New issuer URL.
    domain: "example.com", // New email domain or comma-separated domains.
    oidcConfig: { clientSecret: "NEW_CLIENT_SECRET" }, // Fields of the OIDC configuration to change.
    samlConfig, // Fields of the SAML configuration to change. Set \`idpInitiatedCallbackUrl\` to \`null\` to clear it.
});
```

Parameters

`providerId` stringrequired

ID of the provider to update.

`issuer` string

New issuer URL.

`domain` string

New email domain or comma-separated domains.

`oidcConfig` Partial<OIDCConfig>

Fields of the OIDC configuration to change.

`samlConfig` Partial<SAMLConfig>

Fields of the SAML configuration to change. Set `idpInitiatedCallbackUrl` to `null` to clear it.

- A supplied `mapping` replaces the whole previous mapping.
- Changing `domain` resets `domainVerified` to `false`.
- Changing an identity field returns 409 while accounts are linked to the provider. The identity fields are:
	- Both: `issuer`
		- OIDC: authorization, token, userinfo, JWKS, and discovery endpoints, and `clientId`
		- SAML: IdP entity ID, IdP SSO URLs, SP entity ID, NameID format, POST ACS URLs, `wantAssertionsSigned`, and `audience`

POST/sso/delete-provider

```
const { data, error } = await authClient.sso.deleteProvider({
    providerId: "acme", // required, ID of the provider to delete.
});
```

Parameters

`providerId` stringrequired

ID of the provider to delete.

To block an update or delete of a stored provider, throw from [`guardProviderMutation`](https://better-auth.com/docs/plugins/sso/reference#plugin-options). The change is rejected with 409 `SSO_PROVIDER_MUTATION_REJECTED`. The guard runs after the provider row is locked, and it needs a database adapter with native transactions and async-context support. It receives:

- `action`: `"update"` or `"delete"`.
- `provider`: the `id`, `providerId`, and `organizationId` of the stored row.
- `providerReference`: an opaque reference to that row's configuration. Never persist it.
- `isAuthenticationBoundaryChange`, on update only. It is `true` for the identity fields above and for SAML signing certificate changes. It is `false` for OIDC client secret and SAML SP private key rotation.

Read related state through the `database` in the guard's second argument, because it runs in the same transaction. For example, reject boundary changes while a directory sync is paired. To change the IdP trust configuration, unpair the directory, update the provider, and then pair it again.
