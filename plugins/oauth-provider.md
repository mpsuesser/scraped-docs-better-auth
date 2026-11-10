---
url: https://better-auth.com/llms.txt/docs/plugins/oauth-provider
title: "Oauth Provider"
description: ""
access_date: 2026-11-10T12:07:46.846Z
current_date: 2026-11-10T12:07:46.846Z
---

Make your app an OAuth 2.1 and OpenID Connect provider that other apps sign in with.

The OAuth Provider plugin turns your Better Auth server into an authorization server that implements [OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1), [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html), and [RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414) metadata. Other applications send users to your app to sign in and receive access tokens for your APIs.

## What the provider supports

| OAuth concept | Support |
| --- | --- |
| Grants | Authorization code, [client credentials](https://better-auth.com/docs/plugins/oauth-provider/clients#client-credentials-scopes), and [rotating refresh tokens](https://better-auth.com/docs/plugins/oauth-provider/tokens#refresh-tokens) with `offline_access`. [Device code](https://better-auth.com/docs/plugins/device-authorization) through `oauthDeviceAuthorization()`. Other grants through [extensions](https://better-auth.com/docs/plugins/oauth-provider/reference#provider-extensions). |
| Authorization requests | [`response_type=code`](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow#how-the-redirect-works) with the `query` response mode. `prompt` accepts `none`, `login`, `consent`, `select_account`, and `create`. |
| PKCE | Required by default, `S256` only (RFC 7636). An admin can [exempt a confidential client](https://better-auth.com/docs/plugins/oauth-provider/tokens#pkce). |
| Redirect URIs | [Exact match](https://better-auth.com/docs/plugins/oauth-provider/clients#redirect-uri-forms) by default. A native client may vary the loopback port. |
| Client authentication | `client_secret_basic`, `client_secret_post`, [`private_key_jwt`](https://better-auth.com/docs/plugins/oauth-provider/clients#private-key-jwt) (RFC 7523), and `none` for public clients. Other assertion-based methods through [extensions](https://better-auth.com/docs/plugins/oauth-provider/reference#provider-extensions). |
| Client registration | Admin and user [client APIs](https://better-auth.com/docs/plugins/oauth-provider/clients#manage-clients), [dynamic registration](https://better-auth.com/docs/plugins/oauth-provider/clients#dynamic-client-registration) (RFC 7591), and [Client ID Metadata Documents](https://better-auth.com/docs/plugins/cimd). |
| Resource indicators | [Resources](https://better-auth.com/docs/plugins/oauth-provider/tokens#resources) (RFC 8707). By default, a client gets tokens only for the resources it is linked to. |
| Access tokens | [JWT](https://better-auth.com/docs/plugins/oauth-provider/resource-server#jwt-or-opaque-tokens) (RFC 9068) when the request names a resource and the JWT plugin is enabled. Otherwise opaque. |
| Sender-constrained tokens | [DPoP](https://better-auth.com/docs/plugins/oauth-provider/tokens#dpop) (RFC 9449). |
| Introspection and revocation | [Introspection](https://better-auth.com/docs/plugins/oauth-provider/resource-server#introspection) (RFC 7662) and [revocation](https://better-auth.com/docs/plugins/oauth-provider/tokens#revocation) (RFC 7009). |
| Discovery | [Authorization server metadata](#expose-the-well-known-metadata) (RFC 8414) and OpenID Connect discovery. `iss` in every authorization response (RFC 9207). [Protected resource metadata](https://better-auth.com/docs/plugins/oauth-provider/resource-server#protected-resource-metadata) for your API (RFC 9728). |
| Consent | Stored per user, client, and optional [reference](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow#post-login-screen), such as an organization. Users can [list, update, and delete](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow#manage-consents) their consents. |
| OpenID Connect | [ID tokens and UserInfo](https://better-auth.com/docs/plugins/oauth-provider/openid-connect), the `claims` parameter, and [pairwise subjects](https://better-auth.com/docs/plugins/oauth-provider/openid-connect#pairwise-subject-identifiers). |
| Logout | [RP-initiated](https://better-auth.com/docs/plugins/oauth-provider/openid-connect#rp-initiated-logout) and [back-channel](https://better-auth.com/docs/plugins/oauth-provider/openid-connect#back-channel-logout) logout. |

Not built in:

- Implicit and password grants. OAuth 2.1 removes both.
- The `id_token` and hybrid response types, and the `form_post` response mode.
- Request objects (JAR).
- A pushed authorization request endpoint. [`requestUriResolver`](https://better-auth.com/docs/plugins/oauth-provider/reference#tokens) can resolve `request_uri` values from your own endpoint.
- `client_secret_jwt` and mutual-TLS client authentication.
- Token exchange (RFC 8693). An [extension grant](https://better-auth.com/docs/plugins/oauth-provider/reference#provider-extensions) can add it.
- Front-channel logout and OpenID Connect session management.
- The RFC 7592 client configuration endpoint. Use the [client APIs](https://better-auth.com/docs/plugins/oauth-provider/clients#manage-clients) instead.

## Installation

### Install the plugin

#### npm

```
npm install @better-auth/oauth-provider
```

#### pnpm

#### yarn

#### bun

### Add the plugin to the server

Your provider signs tokens with the [JWT plugin](https://better-auth.com/docs/plugins/jwt). Disable the JWT plugin's `/token` route and its `set-auth-jwt` header. Both give the browser session JWTs signed with the same key as access tokens. Clients get tokens from `/oauth2/token` instead. See [OAuth Provider Mode](https://better-auth.com/docs/plugins/jwt#oauth-provider-mode).

```
import { betterAuth } from "better-auth";
import { jwt } from "better-auth/plugins";
import { oauthProvider } from "@better-auth/oauth-provider"; 

export const auth = betterAuth({
  disabledPaths: ["/token"], 
  plugins: [
    jwt({ disableSettingJwtHeader: true }), 
    oauthProvider({ 
      loginPage: "/sign-in", 
      consentPage: "/consent", 
    }), 
  ],
});
```

### Migrate the database

To add the plugin's fields and tables to your database, run the migration or generate the schema.

#### migrate

#### npm

#### generate

```
npx auth migrate
```

#### pnpm

#### yarn

#### bun

See the [Schema](https://better-auth.com/docs/plugins/oauth-provider/reference#schema) section to add the fields manually.

### Expose the well-known metadata

Clients fetch your provider's metadata from the paths below, and the auth handler serves them when requests reach it. Most frameworks forward only your auth base path, such as `/api/auth/*`, to `auth.handler`. For each path outside it, such as the RFC 8414 path at the root of your domain, add a route.

| Metadata | Path | Helper |
| --- | --- | --- |
| [RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414#section-3) authorization server | `{issuer}/.well-known/oauth-authorization-server` and `/.well-known/oauth-authorization-server/{issuer-path}` | `oauthProviderAuthServerMetadata` |
| [OpenID Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig) | `{issuer}/.well-known/openid-configuration` | `oauthProviderOpenIdConfigMetadata` |

The OpenID configuration is served only when `scopes` includes `openid`, which it does by default. Without a custom issuer, the issuer path is your `basePath`, such as `/api/auth`.

```
import { oauthProviderAuthServerMetadata } from "@better-auth/oauth-provider";
import { auth } from "@/lib/auth";

export const GET = oauthProviderAuthServerMetadata(auth);
```

### Add the client plugins

To resume the flow after each step, add `oauthProviderClient` to the app that renders your login and consent pages. It carries the authorization request through sign-in and consent.

```
import { createAuthClient } from "better-auth/client";
import { oauthProviderClient } from "@better-auth/oauth-provider/client"; 

export const authClient = createAuthClient({
  plugins: [oauthProviderClient()], 
});
```

To verify access tokens in your API, use `oauthProviderResourceClient`; see [Protect your API](https://better-auth.com/docs/plugins/oauth-provider/resource-server#verify-access-tokens).

## Login and consent pages

The authorize endpoint redirects to `loginPage` when the user has no session, and to `consentPage` when the client needs the user's approval. Both receive a signed copy of the authorization request.

On your login page, sign the user in as usual. When the session is created, the plugin continues the authorization request. On your consent page, call `authClient.oauth2.consent`. On both pages, the auth client sends the browser to the next step, so your pages need no redirect code. To build both, see [Sign-in and consent screens](https://better-auth.com/docs/plugins/oauth-provider/sign-in-flow).

## Run your first flow

To try the provider end to end, follow this flow. It issues an access token for one API, `https://api.example.com`. Build your login and consent pages first, because step 3 redirects to them.

### Register the API and a client

Add the API to `resources`, then create a client linked to it. The authorize and token endpoints reject a `resource` the client is not linked to; see [Resources](https://better-auth.com/docs/plugins/oauth-provider/tokens#resources).

To create the client, call `adminCreateOAuthClient`. It runs only on the server and needs a signed-in user's session, so call it from a route that:

- only administrators can reach
- [checks the request's `Origin`](https://better-auth.com/docs/reference/security#csrf-protection)
- passes the request's headers

```
oauthProvider({
  loginPage: "/sign-in",
  consentPage: "/consent",
  resources: ["https://api.example.com"], 
})
```

```
// Inside an admin-only server route that receives \`request\`; check its Origin first
const client = await auth.api.adminCreateOAuthClient({
  headers: request.headers,
  body: {
    client_name: "Example App",
    redirect_uris: ["https://app.example.com/callback"],
    resources: ["https://api.example.com"],
  },
});
// Store client.client_id and client.client_secret in the client app.
```

To register a public client, such as a single-page or native app, add `token_endpoint_auth_method: "none"`. The client gets no secret and authenticates with PKCE alone.

### Send the user to the authorize endpoint

The client app generates a PKCE verifier, then redirects with its S256 challenge as described in [RFC 7636 §4.2](https://datatracker.ietf.org/doc/html/rfc7636#section-4.2).

```
https://auth.example.com/api/auth/oauth2/authorize
  ?response_type=code
  &client_id=CLIENT_ID
  &redirect_uri=https://app.example.com/callback
  &scope=openid profile offline_access
  &state=STATE
  &code_challenge=CODE_CHALLENGE
  &code_challenge_method=S256
  &resource=https://api.example.com
```

### Exchange the code for tokens

The client app posts the code and the PKCE verifier to the token endpoint.

```
curl -X POST https://auth.example.com/api/auth/oauth2/token \
  -u "CLIENT_ID:CLIENT_SECRET" \
  -d grant_type=authorization_code \
  -d code=CODE \
  -d redirect_uri=https://app.example.com/callback \
  -d code_verifier=CODE_VERIFIER \
  -d resource=https://api.example.com
```

The response holds an access token for `https://api.example.com`, an ID token, and a refresh token.

### Check the token

Call UserInfo with the access token. A 200 response with the user's `sub` and `name` means the flow worked.

```
curl https://auth.example.com/api/auth/oauth2/userinfo \
  -H "Authorization: Bearer ACCESS_TOKEN"
```
