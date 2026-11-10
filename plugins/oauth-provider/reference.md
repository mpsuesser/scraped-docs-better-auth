---
url: https://better-auth.com/llms.txt/docs/plugins/oauth-provider/reference
title: "Reference"
description: ""
access_date: 2026-11-10T15:42:54.528Z
current_date: 2026-11-10T15:42:54.528Z
---

OAuth provider options, endpoints, errors, rate limits, extensions, and database schema.

## Plugin options

Pass these options to `oauthProvider()`. Only `loginPage` and `consentPage` are required.

### Sign-in flow

`loginPage` `string`

Required. Page the authorize endpoint redirects to when the user must sign in. See [Login page](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow#login-page).

`consentPage` `string`

Required. Page the authorize endpoint redirects to when the user must consent. See [Consent page](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow#consent-page).

`signup` `object`

Settings for `prompt=create`. See [Sign-up account screen](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow#sign-up-account-screen).

`selectAccount` `object`

Settings for `prompt=select_account`. See [Select account screen](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow#select-account-screen).

`postLogin` `object`

A step between login and consent, such as choosing an organization. See [Post-login screen](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow#post-login-screen).

`allowPublicClientPrelogin` `boolean`

Lets `/oauth2/public-client-prelogin` answer before sign-in when the request carries a valid signed `oauth_query`. The requested `client_id` must match the client in the signed query. Defaults to `false`.

### Scopes and claims

`scopes` `string[]`

Scopes clients may request. Without `openid`, the provider is OAuth only. Defaults to `["openid", "profile", "email", "offline_access"]`. See [Scopes](https://better-auth.com/docs/plugins/oauth-provider/tokens#scopes).

`advertisedMetadata` `{ scopes_supported?, claims_supported? }`

Overrides what discovery advertises. Every `scopes_supported` value must be in `scopes`. See [Discovery metadata](https://better-auth.com/docs/plugins/oauth-provider/openid-connect#discovery-metadata).

`customAccessTokenClaims` `(info) => Awaitable<Record<string, any>>`

Adds access token claims, also returned by introspection. See [Custom claims](https://better-auth.com/docs/plugins/oauth-provider/tokens#custom-claims).

`customIdTokenClaims` `(info) => Awaitable<Record<string, any>>`

Adds ID token claims. Reserved names are stripped. See [ID token claims](https://better-auth.com/docs/plugins/oauth-provider/openid-connect#id-token-claims).

`customUserInfoClaims` `(info) => Awaitable<Record<string, any>>`

Adds UserInfo claims. See [UserInfo](https://better-auth.com/docs/plugins/oauth-provider/openid-connect#userinfo).

`customTokenResponseFields` `(info) => Awaitable<Record<string, unknown>>`

Adds fields to the token response body. Standard fields cannot be overridden. See [Custom token response fields](https://better-auth.com/docs/plugins/oauth-provider/tokens#custom-token-response-fields).

`pairwiseSecret` `string`

Enables pairwise `sub` values for clients with `subject_type: "pairwise"`. Must be at least 32 characters. See [Pairwise subject identifiers](https://better-auth.com/docs/plugins/oauth-provider/openid-connect#pairwise-subject-identifiers).

### Clients and registration

`grantTypes` `GrantType[]`

Grants the token endpoint accepts. `refresh_token` requires `authorization_code`. Extension grants are added on top. Defaults to `["authorization_code", "client_credentials", "refresh_token"]`.

`validateRedirectUri` `(uri, registeredUris, defaultResult) => Awaitable<boolean>`

Replaces exact redirect URI matching. See [Custom redirect URI validation](https://better-auth.com/docs/plugins/oauth-provider/clients#custom-redirect-uri-validation).

`allowInsecureRedirectUri` `(url: URL) => boolean`

Accepts `http` redirect and post-logout redirect URIs that the default policy rejects. See [HTTP redirect URIs](https://better-auth.com/docs/plugins/oauth-provider/clients#http-redirect-uris).

`cachedTrustedClients` `Set<string>`

Client IDs to cache. Cached clients cannot be changed through the CRUD endpoints. See [Trusted clients](https://better-auth.com/docs/plugins/oauth-provider/clients#trusted-clients).

`clientPrivileges` `(context) => Awaitable<boolean | undefined>`

Authorizes client actions: `create`, `read`, `update`, `delete`, `list`, `rotate`, and `configure-client-credentials-scopes`. See [Client CRUD privileges](https://better-auth.com/docs/plugins/oauth-provider/clients#client-crud-privileges).

`clientReference` `(context) => Awaitable<string | undefined>`

Returns the owner reference, such as an organization ID, for new clients. See [Organizations](https://better-auth.com/docs/plugins/oauth-provider/clients#organizations).

`allowDynamicClientRegistration` `boolean`

Enables `POST /oauth2/register`. Defaults to `false`. See [Dynamic client registration](https://better-auth.com/docs/plugins/oauth-provider/clients#dynamic-client-registration).

`allowUnauthenticatedClientRegistration` `boolean`

Accepts registration requests without a session. Defaults to `false`. Anyone on the internet can then register a client; read [Dynamic client registration](https://better-auth.com/docs/plugins/oauth-provider/clients#dynamic-client-registration) before you enable it.

`validateInitialAccessToken` `(context) => Awaitable<{ referenceId? } | false>`

Enables registration with an initial access token ([RFC 7591 §3](https://datatracker.ietf.org/doc/html/rfc7591#section-3)).

`clientRegistrationDefaultScopes` `string[]`

Scopes stored on every dynamically registered client. Defaults to `scopes`, so registered clients get every scope your server defines. Narrow it when you enable open registration.

`clientRegistrationAllowedScopes` `string[]`

Extra scopes a registration may ask for. Every value must be in `scopes`. Defaults to `[]`.

`clientRegistrationRequirePKCE` `boolean`

Whether dynamically registered confidential clients require PKCE. Defaults to `true`. `false` removes the protection against intercepted and injected codes for every such client, including anonymous ones. See [PKCE](https://better-auth.com/docs/plugins/oauth-provider/tokens#pkce).

`clientRegistrationDefaultApplicationType` `"web" | "native" | "infer"`

`application_type` used when a registration omits it. Defaults to `"web"`.

`clientRegistrationClientSecretExpiration` `number | string | Date`

The `client_secret_expires_at` value returned to dynamically registered clients that receive a secret. A string such as `"30d"` is a duration from registration. A number (Unix seconds) or a `Date` is a fixed time. Better Auth does not reject the secret after that time. Defaults to no expiry.

`clientRegistrationDefaultResources` `string[]`

Resources linked to every new client, including CIMD, anonymously registered, and admin-created clients. Each must be in `resources`. Every client that can register can then get tokens for these APIs.

`clientRegistrationAllowedResources` `string[]`

Resources a registration may request. Each must be in `resources`. When both resource options are omitted, requested resources are rejected.

`assertionMaxLifetime` `number`

Maximum lifetime in seconds of a `private_key_jwt` assertion. Defaults to `300`. See [Private key JWT](https://better-auth.com/docs/plugins/oauth-provider/clients#private-key-jwt).

`allowClientKeyRotation` `boolean`

Enables `POST /oauth2/client/rotate-keys`. Startup fails on the memory adapter or with `advanced.database.generateId: "serial"`, which cannot keep assertions single-use. Defaults to `false`. See [Rotating client keys](https://better-auth.com/docs/plugins/oauth-provider/clients#rotating-client-keys).

### Resources

`resources` `Array<string | OAuthResourceInput>`

Protected resources seeded into `oauthResource` at startup. See [Resources](https://better-auth.com/docs/plugins/oauth-provider/tokens#resources).

`resourceSeedMode` `"insertOnly" | "merge" | "overwrite"`

How seeding treats existing rows: skip them, update only the configured fields, or replace them. Defaults to `"insertOnly"`.

`enforcePerClientResources` `boolean`

Requires a client to be linked to every `resource` it requests ([RFC 8707 §2](https://datatracker.ietf.org/doc/html/rfc8707#section-2)). Defaults to `true`. `false` lets any client, including self-registered ones, get tokens for every enabled resource.

`cachedResources` `Set<string>`

Resource identifiers to cache. Writes through the admin endpoints invalidate the cache.

`identifierValidator` `(identifier) => Awaitable<boolean>`

Validates identifiers created through the admin endpoints or registration. The default requires an absolute URI without a fragment ([RFC 8707 §2](https://datatracker.ietf.org/doc/html/rfc8707#section-2)).

`resourcePrivileges` `(context) => Awaitable<boolean | undefined>`

Authorizes resource actions: `create`, `read`, `update`, `delete`, `list`, `link`, and `unlink`.

### Tokens

`accessTokenExpiresIn` `number`

Access token lifetime in seconds. Defaults to `3600`. See [Lifetimes](https://better-auth.com/docs/plugins/oauth-provider/tokens#lifetimes).

`m2mAccessTokenExpiresIn` `number`

Lifetime in seconds of `client_credentials` access tokens. Defaults to `3600`.

`idTokenExpiresIn` `number`

ID token lifetime in seconds. Defaults to `36000`.

`refreshTokenExpiresIn` `number`

Refresh token lifetime in seconds. Defaults to `2592000`. Each refresh starts a new window, so a regularly used grant does not expire.

`codeExpiresIn` `number`

Authorization code lifetime in seconds. Defaults to `600`.

`scopeExpirations` `Record<scope, number | string | Date>`

Shorter access token lifetimes per scope. The earliest expiry wins.

`refreshTokenReuseInterval` `number`

Seconds a rotated refresh token can replay the same response. Defaults to `0`. See [Refresh tokens](https://better-auth.com/docs/plugins/oauth-provider/tokens#refresh-tokens).

`formatRefreshToken` `{ encrypt, decrypt }`

Custom refresh token encoding sent to clients.

`disableJwtPlugin` `boolean`

Issues opaque access tokens and HS256 ID tokens. Defaults to `false`. See [Without the JWT plugin](https://better-auth.com/docs/plugins/oauth-provider/tokens#without-the-jwt-plugin).

`dpop` `{ proofMaxAgeSeconds?, signingAlgorithms? }`

DPoP proof validation. `proofMaxAgeSeconds` defaults to `300`, and `signingAlgorithms` to `["EdDSA", "ES256", "ES512", "PS256", "RS256"]`. See [DPoP](https://better-auth.com/docs/plugins/oauth-provider/tokens#dpop).

`requestUriResolver` `(input) => Promise<Record<string, string> | null>`

Resolves a `request_uri` at the authorize endpoint for pushed authorization requests ([RFC 9126 §4](https://datatracker.ietf.org/doc/html/rfc9126#section-4)). Return `null` for an unknown or expired URI.

### Storage and generation

`storeClientSecret` `"hashed" | "encrypted" | { hash, verify? } | { encrypt, decrypt }`

How client secrets are stored. Hashing requires the JWT plugin; encryption requires `disableJwtPlugin`. Defaults to `"hashed"`, or `"encrypted"` with `disableJwtPlugin`. See [Storage](https://better-auth.com/docs/plugins/oauth-provider/tokens#storage).

`storeTokens` `"hashed" | { hash }`

How opaque access tokens and refresh tokens are stored. Defaults to `"hashed"`.

`prefix` `{ opaqueAccessToken?, refreshToken?, clientSecret? }`

Prefixes added to returned values and not stored. Set them before your first deployment; after that, add the prefix in the matching `generate*` function instead.

`generateClientId` `() => string`

Generates client IDs. Defaults to 32 random letters.

`generateClientSecret` `() => string`

Generates client secrets. Defaults to 32 random letters.

`generateOpaqueAccessToken` `() => Awaitable<string>`

Generates opaque access tokens. Defaults to 32 random letters.

`generateRefreshToken` `() => Awaitable<string>`

Generates refresh tokens. Defaults to 32 random letters.

### Other

`rateLimit` `object`

Per-endpoint rate limits. For the defaults, see [Rate limits](#rate-limits).

`extensions` `OAuthProviderExtension[]`

Companion extensions. Plugins usually call `extendOAuthProvider()` instead. See [Provider extensions](#provider-extensions).

`schema` `object`

Custom model and field names for the plugin tables.

## Endpoints

Paths are relative to your auth base path. "Server-only" endpoints are callable through `auth.api` only.

| Method | Path | Server method | Auth | Server-only |
| --- | --- | --- | --- | --- |
| `GET` | `/.well-known/oauth-authorization-server` | `getOAuthServerConfig` | None | Yes |
| `GET` | `/.well-known/openid-configuration` | `getOpenIdConfig` | None | Yes |
| `GET` `POST` | `/oauth2/authorize` | `oauth2Authorize` | Session, or redirect to `loginPage` | No |
| `POST` | `/oauth2/consent` | `oauth2Consent` | Session and signed `oauth_query` | No |
| `POST` | `/oauth2/continue` | `oauth2Continue` | Session and signed `oauth_query` | No |
| `POST` | `/oauth2/token` | `oauth2Token` | Client authentication | No |
| `POST` | `/oauth2/introspect` | `oauth2Introspect` | Client authentication | No |
| `POST` | `/oauth2/revoke` | `oauth2Revoke` | Client authentication | No |
| `GET` `POST` | `/oauth2/userinfo` | `oauth2UserInfo` | Access token with `openid` | No |
| `GET` `POST` | `/oauth2/end-session` | `oauth2EndSession` | `id_token_hint`, or user confirmation | No |
| `POST` | `/oauth2/end-session/confirm` | — | Signed confirmation cookie | HTTP only |
| `POST` | `/oauth2/register` | `registerOAuthClient` | Session, initial access token, or none | No |
| `POST` | `/admin/oauth2/create-client` | `adminCreateOAuthClient` | Server, with a session | Yes |
| `PATCH` | `/admin/oauth2/update-client` | `adminUpdateOAuthClient` | Server, with a session | Yes |
| `POST` | `/oauth2/create-client` | `createOAuthClient` | Session | No |
| `GET` | `/oauth2/get-client` | `getOAuthClient` | Session | No |
| `GET` | `/oauth2/public-client` | `getOAuthClientPublic` | Session | No |
| `POST` | `/oauth2/public-client-prelogin` | `getOAuthClientPublicPrelogin` | Signed `oauth_query`, with `allowPublicClientPrelogin` | No |
| `GET` | `/oauth2/get-clients` | `getOAuthClients` | Session | No |
| `POST` | `/oauth2/update-client` | `updateOAuthClient` | Session | No |
| `POST` | `/oauth2/client/rotate-secret` | `rotateClientSecret` | Session | No |
| `POST` | `/oauth2/client/rotate-keys` | `rotateClientKeys` | Client assertion | No |
| `POST` | `/oauth2/delete-client` | `deleteOAuthClient` | Session | No |
| `GET` | `/oauth2/get-consent` | `getOAuthConsent` | Session | No |
| `GET` | `/oauth2/get-consents` | `getOAuthConsents` | Session | No |
| `POST` | `/oauth2/update-consent` | `updateOAuthConsent` | Session | No |
| `POST` | `/oauth2/delete-consent` | `deleteOAuthConsent` | Session | No |
| `POST` | `/admin/oauth2/resources` | `adminCreateOAuthResource` | Session and `resourcePrivileges` | Yes |
| `GET` | `/admin/oauth2/resources` | `adminListOAuthResources` | Session and `resourcePrivileges` | Yes |
| `GET` | `/admin/oauth2/resources/:identifier` | `adminGetOAuthResource` | Session and `resourcePrivileges` | Yes |
| `PATCH` | `/admin/oauth2/resources/:identifier` | `adminUpdateOAuthResource` | Session and `resourcePrivileges` | Yes |
| `DELETE` | `/admin/oauth2/resources/:identifier` | `adminDeleteOAuthResource` | Session and `resourcePrivileges` | Yes |
| `POST` | `/admin/oauth2/resources/:identifier/clients/:client_id` | `adminLinkClientResource` | Session and `resourcePrivileges` | Yes |
| `DELETE` | `/admin/oauth2/resources/:identifier/clients/:client_id` | `adminUnlinkClientResource` | Session and `resourcePrivileges` | Yes |

The provider also serves both well-known documents at their issuer-relative paths ([RFC 8414 §3](https://datatracker.ietf.org/doc/html/rfc8414#section-3)). Their `auth.api` methods are server-only. In the browser, `oauthProviderClient()` adds the signed `oauth_query` from the page URL to POST requests. `/oauth2/register` rejects requests unless `allowDynamicClientRegistration` is on, and `/oauth2/client/rotate-keys` unless `allowClientKeyRotation` is on.

## Token errors

Errors use the [RFC 6749 §5.2](https://datatracker.ietf.org/doc/html/rfc6749#section-5.2) format. The built-in grants authenticate the client before they look up the grant, so an unknown `client_id` returns `invalid_client`, not `invalid_grant`.

| Error | Status | Returned when |
| --- | --- | --- |
| `invalid_request` | 400 | A required request field is missing or malformed. |
| `invalid_client` | 400 | Client authentication failed, or the client used a method other than its registered `token_endpoint_auth_method`. |
| `invalid_client` | 401 | `client_secret_basic` authentication failed. The response carries a `WWW-Authenticate: Basic` challenge. |
| `invalid_grant` | 400 | The code, refresh token, or PKCE verifier is invalid, expired, or issued to another client. |
| `invalid_user` | 400 | The user who approved an authorization code was deleted before the code was exchanged. |
| `unauthorized_client` | 400 | The client is not registered for the grant, a public client used `client_credentials`, or the client has no `client_credentials` scopes. |
| `unsupported_grant_type` | 400 | The grant is not in `grantTypes` and no extension handles it. |
| `invalid_scope` | 400 | A requested scope is not allowed for the client, or none are allowed for a requested resource. |
| `invalid_target` | 400 | A `resource` is unknown, disabled, or not linked to the client while `enforcePerClientResources` is on. |
| `invalid_dpop_proof` | 400 | The DPoP proof is missing or invalid for a DPoP-bound client or resource. |

When `adminCreateOAuthClient` links resources, an invalid identifier fails validation with `VALIDATION_ERROR`, an unknown or disabled resource returns `invalid_target`, and a `resourcePrivileges` denial returns `401`. The call creates nothing in each case.

## Rate limits

Limits apply per IP and per endpoint, and only when Better Auth [rate limiting](https://better-auth.com/docs/concepts/rate-limit) is enabled, which by default is production only.

| Option key | Endpoint | Window | Max |
| --- | --- | --- | --- |
| `token` | `/oauth2/token` | 60s | 20 |
| `authorize` | `/oauth2/authorize` | 60s | 30 |
| `introspect` | `/oauth2/introspect` | 60s | 100 |
| `revoke` | `/oauth2/revoke` | 60s | 30 |
| `register` | `/oauth2/register` | 60s | 5 |
| `rotateClientKeys` | `/oauth2/client/rotate-keys` | 60s | 5 |
| `userinfo` | `/oauth2/userinfo` | 60s | 60 |

Override a limit with `{ window, max }`, or set it to `false` to fall back to the global limits:

```
oauthProvider({
  rateLimit: {
    token: { window: 60, max: 50 },
    introspect: false,
  },
})
```

## Provider extensions

Companion plugins call `extendOAuthProvider()` from their `init()` hook to add token grants, assertion-based client authentication, discovery metadata, claims, and client-id discovery. The [CIMD plugin](https://better-auth.com/docs/plugins/cimd) uses this to add URL-based client discovery.

```
import type { BetterAuthPlugin } from "better-auth";
import { extendOAuthProvider } from "@better-auth/oauth-provider";

export const customOAuthExtension = () =>
  ({
    id: "custom-oauth-extension",
    init(ctx) {
      extendOAuthProvider(ctx, {
        grants: {
          "urn:example:params:oauth:grant-type:custom": async ({ provider }) => {
            const { client } = await provider.authenticateClient();
            return provider.issueTokens({ client, scopes: ["openid"] });
          },
        },
        metadata: () => ({ custom_grant_supported: true }),
      });
    },
  }) satisfies BetterAuthPlugin;
```

- `grants` and `clientAuthentication` must not overlap across extensions. A duplicate grant type, auth method, or assertion type fails at startup. Both are advertised in discovery.
- `metadata` and `claims` only add keys. A key the provider owns is kept, and a key two extensions contribute resolves to the first one registered. To advertise extension claims, set `advertisedMetadata.claims_supported`.
- A `clientDiscovery` entry has a stable `id` stored as client provenance. Changing it requires migrating the owned clients, and removing it makes them fail closed.

A grant handler receives `provider` (`getClient`, `authenticateClient`, `issueTokens`, `hashToken`, `validateAccessToken`, `requireActiveAccessToken`). Other endpoints get the same object from `getOAuthProviderApi(ctx, opts, grantType?)`. Without `grantType`, `issueTokens` throws. `validateAccessToken` can return an inactive payload; protected endpoints should use `requireActiveAccessToken`, which rejects inactive tokens with a bearer challenge.

To bind a token with `cnf` ([RFC 7800 §3](https://datatracker.ietf.org/doc/html/rfc7800#section-3)), pass `confirmation` to `issueTokens` or return it from a `clientAuthentication` strategy. Claim contributors cannot set `cnf`.

### Client authentication obligations

A `clientAuthentication` strategy proves which client sent the assertion and returns its `clientId`. The provider loads and authorizes the client record itself. After verifying the signature, the strategy must bind the audience, bound the lifetime, and reject replayed `jti` values ([RFC 7523 §3](https://datatracker.ietf.org/doc/html/rfc7523#section-3)). `consumeClientAssertion` performs all three:

```
import { consumeClientAssertion } from "@better-auth/oauth-provider";
import { APIError } from "better-auth/api";

authenticate: async ({ ctx, opts, assertion, expectedAudience }) => {
  if (!expectedAudience) throw new APIError("BAD_REQUEST", { error: "invalid_client" });
  const payload = await verifyAssertionSignature(assertion); // your key source
  await consumeClientAssertion(ctx, opts, {
    namespace: \`urn:example:attestation:${payload.sub}\`,
    payload,
    expectedAudience,
  });
  return { clientId: payload.sub };
};
```

### Claim precedence

Extension claims only add claim names. Your own callbacks can override profile claims. The provider always owns identity, lifetime, binding, and authentication-context claims.

| Token | Order, lowest to highest authority | Owned by the provider |
| --- | --- | --- |
| Access token | extension `claims.accessToken` < per-issuance `accessTokenClaims` < `customAccessTokenClaims` < per-resource `customClaims` | `iss`, `sub`, `aud`, `exp`, `iat`, `jti`, `client_id`, `scope`, `auth_time`, `acr`, `amr`, `cnf`, stripped before signing |
| ID token | subject and authentication claims < `customIdTokenClaims`. Extension and per-issuance `idTokenClaims` only add names and cannot set profile or email claims. | `iss`, `sub`, `aud`, `exp`, `nbf`, `iat`, `jti`, `nonce`, `sid`, `at_hash`, `c_hash`, `s_hash`, `auth_time`, `acr`, `amr`, `azp` |
| UserInfo | scope and `claims.userinfo` claims < extension `claims.userInfo` (add only) < `customUserInfoClaims` | `sub`, applied last |

Per-issuance `accessTokenClaims` exist only in JWT access tokens. A claim that introspection of an opaque token must return belongs in a `claims.accessToken` contributor that does not depend on the grant type.

## Schema

The plugin adds the following tables. Run `npx auth generate` for the exact schema of your adapter.

### OAuth Client

Table Name: `oauthClient`

Table

Field

Type

Attributes

Description

id

string

PK

Database ID of the client

clientId

string

UQ

Public client identifier

clientSecret?

string

\-

Hashed or encrypted secret. Absent for public clients.

clientDiscoveryId?

string

\-

ID of the client discovery that owns this client, such as \`cimd\`

disabled?

boolean

\-

Whether the client is disabled. Defaults to \`false\`.

skipConsent?

boolean

\-

Whether the consent screen is skipped

enableEndSession?

boolean

\-

Whether the client can use \`/oauth2/end-session\`

subjectType?

string

\-

\`public\` or \`pairwise\`

scopes?

string\[\]

\-

Scopes the client may request for users

clientCredentialsScopes?

string\[\]

\-

Scopes allowed for \`client\_credentials\`. Empty denies the grant.

userId?

string

FK

Owning user

referenceId?

string

\-

Owning reference when the owner is not a user

createdAt?

Date

\-

Creation time

updatedAt?

Date

\-

Last update time

name?

string

\-

Display name

uri?

string

\-

Client website

icon?

string

\-

Logo URI

contacts?

string\[\]

\-

Contact addresses

softwareId?

string

\-

Software identifier, stable across versions

softwareVersion?

string

\-

Software version

softwareStatement?

string

\-

Signed software statement JWT

redirectUris

string\[\]

\-

Registered redirect URIs

postLogoutRedirectUris?

string\[\]

\-

Registered post-logout redirect URIs

backchannelLogoutUri?

string

\-

URI that receives logout tokens

backchannelLogoutSessionRequired?

boolean

\-

Whether every logout token must carry \`sid\`

tokenEndpointAuthMethod?

string

\-

\`none\`, \`client\_secret\_basic\`, \`client\_secret\_post\`, \`private\_key\_jwt\`, or an extension method

applicationType?

string

\-

\`web\` or \`native\`

jwks?

string

\-

Inline public JWK Set as JSON. Exclusive with \`jwksUri\`.

jwksUri?

string

\-

HTTPS URI of the public JWK Set. Exclusive with \`jwks\`.

grantTypes?

string\[\]

\-

Allowed grant types

responseTypes?

string\[\]

\-

Allowed response types. Only \`code\`.

requirePKCE?

boolean

\-

Whether PKCE is required

dpopBoundAccessTokens?

boolean

\-

Whether tokens must be DPoP-bound. Defaults to \`false\`.

metadata?

json

\-

Additional client metadata

### OAuth Resource

Table Name: `oauthResource`

A `null` policy field uses the plugin default at issuance.

Table

Field

Type

Attributes

Description

id

string

PK

Database ID of the resource

identifier

string

UQ

\`resource\` parameter value and \`aud\` claim

name

string

\-

Display name

accessTokenTtl?

number

\-

Access token lifetime in seconds

refreshTokenTtl?

number

\-

Refresh token lifetime in seconds

signingAlgorithm?

string

\-

JWS algorithm for access tokens

signingKeyId?

string

\-

Signing key ID for access tokens

allowedScopes?

string\[\]

\-

Scopes allowed for this resource. Requested scopes are narrowed to this set.

customClaims?

json

\-

Claims added to access tokens. Reserved names are stripped.

dpopBoundAccessTokensRequired?

boolean

\-

Whether access tokens must be DPoP-bound. Defaults to \`false\`.

disabled?

boolean

\-

Stops new issuance. Existing tokens verify until they expire. Defaults to \`false\`.

createdAt?

Date

\-

Creation time

updatedAt?

Date

\-

Last update time

policyVersion?

number

\-

Policy format version. Defaults to \`1\`.

metadata?

json

\-

Additional resource metadata

### OAuth Client Resource

Table Name: `oauthClientResource`

Links a client to a resource. The pair `(clientId, resourceId)` is unique. `enforcePerClientResources` controls whether authorization and token requests check these rows. [Introspection](https://better-auth.com/docs/plugins/oauth-provider/resource-server#introspection) always uses them to decide whether a resource server can introspect another client's token.

Table

Field

Type

Attributes

Description

id

string

PK

Database ID of the link

clientId

string

FK

Linked client

resourceId

string

FK

Linked resource identifier

metadata?

json

\-

Additional link metadata

createdAt?

Date

\-

Creation time

### OAuth Refresh Token

Table Name: `oauthRefreshToken`

Table

Field

Type

Attributes

Description

id

string

PK

Database ID of the refresh token

token

string

UQ

Stored token value

clientId

string

FK

Client the token was issued to

sessionId?

string

FK

Session at issuance

userId

string

FK

User the token was issued for

referenceId?

string

\-

Consent reference

authorizationCodeId?

string

\-

Authorization code that started this token family

resources?

string\[\]

\-

Resources the token may request

requestedUserInfoClaims?

string\[\]

\-

UserInfo claims requested with \`claims\`

scopes

string\[\]

\-

Granted scopes

revoked?

Date

\-

When the token stopped being active

rotatedAt?

Date

\-

When rotation consumed the token

rotationReplayResponse?

string

\-

Encrypted response replayed within \`refreshTokenReuseInterval\`

rotationReplayExpiresAt?

Date

\-

When the replay window ends

authTime?

Date

\-

Original authentication time, kept for \`auth\_time\`

confirmation?

json

\-

\`cnf\` binding, such as a DPoP key thumbprint

createdAt

Date

\-

Creation time

expiresAt

Date

\-

Expiry time

### OAuth Access Token

Table Name: `oauthAccessToken`

Stores opaque access tokens only. JWT access tokens are not stored.

Table

Field

Type

Attributes

Description

id

string

PK

Database ID of the access token

token

string

UQ

Stored token value

clientId

string

FK

Client the token was issued to

sessionId?

string

FK

Session at issuance

userId?

string

FK

User the token was issued for. Absent for \`client\_credentials\`.

referenceId?

string

\-

Consent reference

authorizationCodeId?

string

\-

Authorization code that started this token family

refreshId?

string

FK

Refresh token this access token belongs to

resources?

string\[\]

\-

Resources the token is valid for

requestedUserInfoClaims?

string\[\]

\-

UserInfo claims requested with \`claims\`

scopes

string\[\]

\-

Granted scopes

revoked?

Date

\-

When the token was revoked

confirmation?

json

\-

\`cnf\` binding, returned by introspection

createdAt

Date

\-

Creation time

expiresAt

Date

\-

Expiry time

### OAuth Consent

Table Name: `oauthConsent`

Table

Field

Type

Attributes

Description

id

string

PK

Database ID of the consent

clientId

string

FK

Client the consent was given to

userId?

string

FK

User who consented

referenceId?

string

\-

Consent reference

resources?

string\[\]

\-

Consented resources

requestedUserInfoClaims?

string\[\]

\-

Consented UserInfo claims

scopes

string\[\]

\-

Consented scopes

createdAt

Date

\-

Creation time

updatedAt

Date

\-

Last update time

### OAuth Client Assertion

Table Name: `oauthClientAssertion`

Records each `private_key_jwt` assertion `jti` so it can be used once. The check needs string IDs, so with `advanced.database.generateId: "serial"` Better Auth logs a warning. Rows are never pruned; delete expired rows.

Table

Field

Type

Attributes

Description

id

string

PK

Digest of the client ID and assertion \`jti\`

expiresAt

Date

\-

When the row is safe to delete

## Optimizations

Database adapters may map `clientId` on `oauthClient` to `id` for faster lookups. The `id` column must then accept strings shaped like UUIDs and URLs.
