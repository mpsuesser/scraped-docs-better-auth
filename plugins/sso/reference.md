---
url: https://better-auth.com/llms.txt/docs/plugins/sso/reference
title: "Reference"
description: ""
access_date: 2026-11-04T23:37:22.575Z
current_date: 2026-11-04T23:37:22.575Z
---

SSO plugin options, provider configuration, endpoints, and database schema.

## Plugin options

Pass these options to `sso()`. All of them are optional.

### Users and accounts

`resolveUser` `(input, context) => Awaitable<SSOUserResolution>`

Maps a verified provider identity to a user on every SSO sign-in. Return `continue`, `link`, or `reject`. Runs in a database transaction, so the adapter needs native transaction support. See [Resolve SSO users](https://better-auth.com/docs/plugins/sso/provisioning#resolve-sso-users).

`provisionUser` `({ user, userInfo, token, provider }) => Awaitable<void>`

Runs after a new user and account are saved, before organization assignment. A thrown error fails the response, but the user stays created and the callback is not retried unless `provisionUserOnEveryLogin` is on. See [User provisioning](https://better-auth.com/docs/plugins/sso/provisioning#user-provisioning).

`provisionUserOnEveryLogin` `boolean`

Runs `provisionUser` on every SSO sign-in, not only when it creates the user. Keep the callback idempotent. Defaults to `false`.

`organizationProvisioning` `object`

Adds SSO users to the organization the provider is linked to. See [Organization provisioning](https://better-auth.com/docs/plugins/sso/provisioning#organization-provisioning).

`defaultOverrideUserInfo` `boolean`

Sets `overrideUserInfo` on OIDC providers registered while this is on, even when the body passes `false`. Existing providers keep their stored value. Defaults to `false`.

`disableImplicitSignUp` `boolean`

Stops SSO sign-in from creating new users. For OIDC, any caller can pass `requestSignUp: true` at sign-in to allow it. SAML ignores `requestSignUp`. Defaults to `false`. See [Sign in](https://better-auth.com/docs/plugins/sso#sign-in).

`trustEmailVerified` `boolean`

**Deprecated.** Reads the provider's email verified flag through `mapping.emailVerified`. Otherwise every SSO user has `emailVerified: false`. Defaults to `false`.

### Providers

`defaultSSO` `array`

Providers defined in code. They are checked before the database, by `providerId` or by email domain at sign-in. With `domainVerification.enabled`, they count as verified without a DNS check. See [Configure providers in code](https://better-auth.com/docs/plugins/sso#configure-providers-in-code).

`providersLimit` `number | (user) => Awaitable<number>`

Maximum providers each user can register. `0` disables registration. Any signed-in user can register by default. Defaults to `10`. See [Register a provider](https://better-auth.com/docs/plugins/sso#register-a-provider).

`redirectURI` `string`

One OIDC callback for every provider, as a path or a full URL. See [Shared redirect URI](https://better-auth.com/docs/plugins/sso/oidc#shared-redirect-uri).

`resolvePrivateKey` `({ providerId, keyId, issuer }) => Promise<{ privateKeyJwk?, privateKeyPem?, kid?, algorithm? }>`

Returns the signing key for `private_key_jwt`, so keys stay out of the database. Registering a `private_key_jwt` provider fails without it. See [Token endpoint authentication](https://better-auth.com/docs/plugins/sso/oidc#token-endpoint-authentication).

`guardProviderMutation` `(input, context) => Awaitable<void>`

Runs before a stored provider is updated or deleted. Throw to reject the change, which returns a conflict. Needs native transaction support. See [Manage providers](https://better-auth.com/docs/plugins/sso#manage-providers).

`domainVerification` `{ enabled?: boolean; tokenPrefix?: string }`

Requires DNS proof of domain ownership before a provider can sign users in. `tokenPrefix` defaults to `"better-auth-token"`. See [Domain verification](https://better-auth.com/docs/plugins/sso/domain-verification#enable-domain-verification).

### SAML

Set these under `saml`.

`enableInResponseToValidation` `boolean`

Matches each response to a stored AuthnRequest. Turning it off accepts unsolicited responses for every provider. Defaults to `true`. See [InResponseTo validation](https://better-auth.com/docs/plugins/sso/saml-security#inresponseto-validation).

`allowIdpInitiated` `boolean`

Accepts unsolicited responses. Applies only while InResponseTo validation is on. Defaults to `false`. See [IdP-initiated SSO](https://better-auth.com/docs/plugins/sso/saml#idp-initiated-sso).

`requestTTL` `number`

Lifetime of an AuthnRequest record, in milliseconds. Defaults to `300000`.

`clockSkew` `number`

Tolerance for assertion timestamps, in milliseconds. Defaults to `300000`. See [Timestamp validation](https://better-auth.com/docs/plugins/sso/saml-security#timestamp-validation).

`requireTimestamps` `boolean`

Rejects assertions without `NotBefore` and `NotOnOrAfter`. Defaults to `false`.

`algorithms` `object`

Rules for signature and encryption algorithms. See [Algorithm validation](https://better-auth.com/docs/plugins/sso/saml-security#algorithm-validation).

`maxResponseSize` `number`

Largest accepted SAML response, in bytes. Defaults to `262144`. See [Size limits](https://better-auth.com/docs/plugins/sso/saml-security#size-limits).

`maxMetadataSize` `number`

Largest accepted IdP or SP metadata XML, in bytes. Also enforced when a provider is saved. Defaults to `102400`.

`idpInitiatedCallbackUrl` `string`

Fallback redirect for IdP-initiated responses when no provider callback applies. An absolute URL or a path starting with `/`. See [ACS and callback URLs](https://better-auth.com/docs/plugins/sso/saml#acs-and-callback-urls).

`enableSingleLogout` `boolean`

Adds the Single Logout endpoints and advertises them in SP metadata. Defaults to `false`. See [Single Logout](https://better-auth.com/docs/plugins/sso/saml#single-logout).

`logoutRequestTTL` `number`

Lifetime of a LogoutRequest record, in milliseconds. Defaults to `300000`.

`wantLogoutRequestSigned` `boolean`

Requires signed LogoutRequests from the IdP. Defaults to `false`.

`wantLogoutResponseSigned` `boolean`

Requires signed LogoutResponses from the IdP. Defaults to `false`.

### Schema options

`schema.ssoProvider.modelName` `string`

Table name. Defaults to `"ssoProvider"`.

`schema.ssoProvider.fields` `object`

Column names for built-in fields, including `domainVerified`.

`schema.ssoProvider.additionalFields` `object`

Extra columns. See [Additional provider fields](#additional-provider-fields).

`modelName` `string`

Same as `schema.ssoProvider.modelName`, and takes precedence over it.

`fields` `object`

Same as `schema.ssoProvider.fields` without `domainVerified`, and takes precedence over it.

## Provider configuration

### Registration body

The body of `POST /sso/register`. `POST /sso/update-provider` takes `providerId` plus optional `issuer`, `domain`, `oidcConfig`, `samlConfig`, and additional fields. See [Manage providers](https://better-auth.com/docs/plugins/sso#manage-providers).

`providerId` `string`

Required and unique. The IDs of built-in providers, your configured social providers, `account.accountLinking.trustedProviders` entries, and `defaultSSO` entries are rejected with 422.

`issuer` `string`

Required URL. The OIDC issuer, or the SAML SP entity ID.

`domain` `string`

Required. A bare email domain, or several separated by commas.

`organizationId` `string`

Links the provider to an organization. The caller must be a member, and an owner or admin when the organization plugin is enabled. See [Link a provider to an organization](https://better-auth.com/docs/plugins/sso/provisioning#link-a-provider-to-an-organization).

`overrideUserInfo` `boolean`

OIDC only. On every sign-in, overwrites the user's name, email, image, and mapped additional fields with the provider's values. A changed email takes the provider's verified flag, which is `false` unless `trustEmailVerified` is on. Enable it only when the IdP controls its users' email addresses. Defaults to `defaultOverrideUserInfo`.

`oidcConfig` `object`

OIDC settings. Pass this, `samlConfig`, or both.

`samlConfig` `object`

SAML settings. Pass this, `oidcConfig`, or both.

**Additional fields**

Fields from `schema.ssoProvider.additionalFields`.

### oidcConfig

See [Register an OIDC provider](https://better-auth.com/docs/plugins/sso/oidc#register-an-oidc-provider).

`clientId` `string`

Required.

`clientSecret` `string`

Required for `client_secret_basic` and `client_secret_post`.

`discoveryEndpoint` `string`

Discovery document URL. Defaults to `{issuer}/.well-known/openid-configuration`. See [OIDC discovery](https://better-auth.com/docs/plugins/sso/oidc#oidc-discovery).

`skipDiscovery` `boolean`

Skips discovery at registration. Then pass `authorizationEndpoint`, `tokenEndpoint`, and `jwksEndpoint`. Defaults to `false`.

`authorizationEndpoint` `string`

Authorization endpoint URL. Discovery fills it in when omitted.

`tokenEndpoint` `string`

Token endpoint URL. Discovery fills it in when omitted.

`userInfoEndpoint` `string`

UserInfo endpoint URL. Discovery fills it in when omitted.

`jwksEndpoint` `string`

JWKS URL for ID Token verification. Discovery fills it in when omitted.

`tokenEndpointAuthentication` `"client_secret_basic" | "client_secret_post" | "private_key_jwt"`

Client authentication at the token endpoint. When omitted, discovery picks it, falling back to `client_secret_basic`. See [Token endpoint authentication](https://better-auth.com/docs/plugins/sso/oidc#token-endpoint-authentication).

`privateKeyId` `string`

Key id passed to `resolvePrivateKey` and sent as `kid`.

`privateKeyAlgorithm` `string`

Signing algorithm for `private_key_jwt`. Defaults to `"RS256"`.

`scopes` `string[]`

Scopes to request. Defaults to `["openid", "email", "profile", "offline_access"]`.

`pkce` `boolean`

Sends a PKCE challenge ([RFC 7636](https://datatracker.ietf.org/doc/html/rfc7636)). Defaults to `true`.

`overrideUserInfo` `boolean`

Where the top-level `overrideUserInfo` is stored. Ignored at registration, where you set the top-level field. Set it here on update.

`mapping` `object`

Claim names to read. See [Mapping](https://better-auth.com/docs/plugins/sso/oidc#mapping).

`allowIdpInitiated` `boolean`

Accepts callbacks without `state`. Set it only on `defaultSSO` entries; registration and update silently drop it. Defaults to `false`. See [IdP-initiated sign-in](https://better-auth.com/docs/plugins/sso/oidc#idp-initiated-sign-in).

### samlConfig

See [Register a SAML provider](https://better-auth.com/docs/plugins/sso/saml#register-a-saml-provider). On `defaultSSO` entries, `samlConfig.issuer` is also required.

`entryPoint` `string`

Required URL. The IdP SSO URL. Ignored when `idpMetadata.metadata` is set.

`cert` `string | string[]`

IdP signing certificates. Pass an array to rotate. Required unless `idpMetadata.metadata` or `idpMetadata.cert` is set. `idpMetadata.cert` takes precedence. See [Signing certificates and rotation](https://better-auth.com/docs/plugins/sso/saml#signing-certificates-and-rotation).

`audience` `string`

Expected assertion audience. Defaults to the SP entity ID: the one in `spMetadata.metadata`, else `spMetadata.entityID`, else `issuer`.

`callbackUrl` `string`

Where users land after an IdP-initiated or fallback sign-in. Not the ACS URL. See [ACS and callback URLs](https://better-auth.com/docs/plugins/sso/saml#acs-and-callback-urls).

`idpInitiatedCallbackUrl` `string`

Fallback for IdP-initiated responses, as an absolute URL or a `/path`. Takes precedence over the plugin option. Pass `null` on update to clear it.

`idpMetadata` `object`

Required. The IdP's metadata XML or its explicit settings.

`spMetadata` `object`

Service provider settings. Generated when omitted. See [Service provider settings](https://better-auth.com/docs/plugins/sso/saml#service-provider-settings).

`wantAssertionsSigned` `boolean`

Rejects unsigned assertions and advertises `WantAssertionsSigned` ([SAML Metadata §2.4.4](https://docs.oasis-open.org/security/saml/v2.0/saml-metadata-2.0-os.pdf)). Custom SP metadata cannot weaken it. Defaults to `false`. See [Signed assertions](https://better-auth.com/docs/plugins/sso/saml#signed-assertions).

`authnRequestsSigned` `boolean`

Signs AuthnRequests. Sign-in returns 400 without a private key. Defaults to `false`. See [Signed AuthnRequests](https://better-auth.com/docs/plugins/sso/saml#signed-authnrequests).

`signatureAlgorithm` `string`

Signature algorithm URI or short form such as `rsa-sha256`. Checked against `saml.algorithms` when the provider is saved.

`digestAlgorithm` `string`

Digest algorithm URI or short form such as `sha256`. Checked against `saml.algorithms` when the provider is saved.

`identifierFormat` `string`

NameID format advertised in generated SP metadata.

`privateKey` `string`

SP signing key, used when `spMetadata.privateKey` is not set.

`mapping` `object`

Attribute names to read. See [Attribute mapping](https://better-auth.com/docs/plugins/sso/saml#attribute-mapping).

## Endpoints

Paths are relative to your auth base path, `/api/auth` by default.

| Method | Path | Auth | Purpose |
| --- | --- | --- | --- |
| `POST` | `/sso/register` | Session | Register a provider. |
| `POST` | `/sign-in/sso` | None | Start an SSO sign-in. |
| `GET` | `/sso/callback/:providerId` | None | OIDC callback. |
| `GET` | `/sso/callback` | None | Shared OIDC callback, used with `redirectURI`. |
| `GET` `POST` | `/sso/saml2/sp/acs/:providerId` | IdP, no origin check | SAML Assertion Consumer Service. |
| `GET` | `/sso/saml2/sp/metadata?providerId=` | None | SP metadata as XML. |
| `GET` `POST` | `/sso/saml2/sp/slo/:providerId` | IdP, no origin check | SAML Single Logout, with `enableSingleLogout`. |
| `POST` | `/sso/saml2/logout/:providerId` | Session | Start SP-initiated Single Logout, with `enableSingleLogout`. |
| `GET` | `/sso/providers` | Session | List the providers you can manage. |
| `GET` | `/sso/get-provider` | Session | Get one provider by `providerId`. |
| `POST` | `/sso/update-provider` | Session | Update a provider. |
| `POST` | `/sso/delete-provider` | Session | Delete a provider and its linked accounts. |
| `POST` | `/sso/request-domain-verification` | Session | Get a DNS verification token, with domain verification. |
| `POST` | `/sso/verify-domain` | Session | Check the DNS record, with domain verification. |

For error codes, see [Discovery errors](https://better-auth.com/docs/plugins/sso/oidc#discovery-errors) and [SAML security](https://better-auth.com/docs/plugins/sso/saml-security).

## Additional provider fields

Add columns to `ssoProvider` with `schema.ssoProvider.additionalFields`.

```
sso({
  schema: {
    ssoProvider: {
      additionalFields: {
        displayName: { type: "string", required: true }, 
      },
    },
  },
});
```

Pass the field when you register or update a provider, for example `displayName: "Acme"`. Fields with `input: false` are rejected in requests, and fields with `returned: false` are left out of client types. Pass the same `schema` to `ssoClient()` so client types match.

## Schema

The plugin adds the `ssoProvider` table. Run `npx auth@latest generate` for the exact schema of your adapter.

Table

Field

Type

Attributes

Description

id

string

PK

Database ID of the provider

issuer

string

\-

OIDC issuer or SAML SP entity ID

domain

string

\-

Email domain, or several separated by commas

oidcConfig?

string

\-

OIDC configuration as a JSON string, including the client secret, unencrypted

samlConfig?

string

\-

SAML configuration as a JSON string, including private keys, unencrypted

userId

string

FK

User who registered the provider

providerId

string

UQ

Provider ID used at sign-in and in callback URLs

organizationId?

string

\-

Linked organization

domainVerified?

boolean

\-

Whether the domain is verified. Added when \`domainVerification.enabled\` is true.

### Verification records

The plugin also writes short-lived records to the verification table, or to secondary storage when one is configured. `saml-used-assertion:` records always need the database; see [Assertion replay protection](https://better-auth.com/docs/plugins/sso/saml-security#assertion-replay-protection).

| Identifier prefix | Stores |
| --- | --- |
| `saml-authn-request:` | AuthnRequest IDs for InResponseTo validation |
| `saml-used-assertion:` | Assertion IDs already consumed, for replay protection |
| `saml-session:` | NameID and SessionIndex for Single Logout |
| `saml-session-by-id:` | Lookup from a session ID to its SAML session record |
| `saml-logout-request:` | LogoutRequest IDs for SP-initiated Single Logout |
| `_{tokenPrefix}-{providerId}` | Domain verification tokens |

Expired rows can be deleted.
