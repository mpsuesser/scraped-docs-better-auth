---
url: https://better-auth.com/llms.txt/docs/plugins/cimd
title: "Cimd"
description: ""
access_date: 2026-08-18T00:08:46.984Z
current_date: 2026-08-18T00:08:46.984Z
---

Discover OAuth clients from HTTPS metadata documents.

`OAuth2` `MCP`

The [Client ID Metadata Document draft-02](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-02) lets an OAuth client identify itself without registering first. The client's `client_id` is the exact HTTPS URL of a metadata document it hosts. Better Auth validates the document and persists the client through the OAuth Provider's canonical registration path.

MCP 2026-07-28 normatively references CIMD draft-00. Use `metadataProfile: "mcp-2026-07-28"` when composing `cimd()` with `mcp()`. This explicit profile adds MCP's required `client_name` and `redirect_uris` fields without imposing those draft-00 restrictions on generic draft-02 clients.

## Installation

### Install the plugin

#### npm

```
npm install @better-auth/cimd
```

#### pnpm

#### yarn

#### bun

### Add the plugin to the server

```
import { betterAuth } from "better-auth";
import { jwt } from "better-auth/plugins";
import { oauthProvider } from "@better-auth/oauth-provider";
import { cimd } from "@better-auth/cimd";
import { fetchClientMetadataResource } from "@better-auth/cimd/node";

export const auth = betterAuth({
  plugins: [
    jwt(),
    oauthProvider({
      loginPage: "/login",
      consentPage: "/consent",
      scopes: ["openid", "profile", "email", "offline_access"],
    }),
    cimd({
      fetchClientMetadataResource,
    }),
  ],
});
```

`fetchClientMetadataResource` is required. Node.js deployments can use the `@better-auth/cimd/node` implementation shown above. It resolves the original hostname once, rejects the request if any DNS result is not public-routable, pins one approved address on an isolated connection, preserves the original Host and TLS certificate identity, and never follows redirects. This transport intentionally supports only `GET` and `HEAD`, which are the methods CIMD uses to retrieve metadata resources.

Bun, Deno, Workers, and other runtimes must provide the equivalent secure transport at their network boundary. The same transport is used for the metadata document and discovery-owned resources such as `jwks_uri`.

The transport must:

- Parse the target as an HTTPS URL before resolving it.
- Resolve the hostname once and reject every RFC 6890 special-use result.
- Connect to that approved address while preserving TLS hostname and certificate verification for the original host.
- Refuse redirects and honor the supplied method, headers, and abort signal.

Do not perform a DNS check and then call `globalThis.fetch`. Standard Fetch exposes neither the connected peer address nor a portable way to pin it, so that pattern re-resolves the hostname and remains vulnerable to DNS rebinding.

## MCP configuration

```
import { cimd } from "@better-auth/cimd";
import { mcp } from "@better-auth/mcp";
import { fetchClientMetadataResource } from "@better-auth/cimd/node";

plugins: [
  mcp({
    resource: "https://mcp.example.com/mcp",
    scopes: ["mcp:tools"],
  }),
  cimd({
    fetchClientMetadataResource,
    metadataProfile: "mcp-2026-07-28",
  }),
];
```

The MCP profile requires `client_id`, `client_name`, and a non-empty `redirect_uris` array. DCR remains disabled unless the OAuth Provider explicitly enables it.

## Configuration

```
cimd({
  fetchClientMetadataResource,
  metadataRevalidationInterval: "10m",
  maxCacheEntries: 1000,
  metadataFetchPolicy: {
    minimumFetchInterval: 1,
    maximumConcurrentFetches: 16,
    maximumConcurrentFetchesPerOrigin: 4,
    maximumFetchesPerMinute: 120,
    maximumFetchesPerOriginPerMinute: 30,
  },
  originBoundFields: ["post_logout_redirect_uris", "client_uri"],
  isMetadataDocumentUrlAllowed(clientIdUrl) {
    return new URL(clientIdUrl).hostname.endsWith(".trusted.example");
  },
  onClientCreated({ client, clientMetadataDocument, context }) {
    // Assign local trust or audit state.
  },
  onClientRefreshed({
    client,
    previousClient,
    clientMetadataDocument,
    context,
  }) {
    // Compare the previous and current records and apply operator policy.
  },
});
```

Prop

## Draft-02 validation

Generic CIMD validation follows draft-02:

- `client_id` must equal the requested metadata URL using simple string comparison. Explicit default ports are therefore significant.
- Client Identifier URLs must use HTTPS and cannot contain credentials, fragments, dot segments, special-use literal/known hosts, or omit an explicit path. An explicit root `/` is accepted with a NOT RECOMMENDED warning; query strings produce a SHOULD NOT warning.
- `client_name` and `redirect_uris` are optional for generic clients. Non-redirect grants such as `client_credentials` are valid when enabled by the OAuth Provider.
- The OAuth Provider remains authoritative for supported grants, response types, redirect policy, and authentication capability.
- Declaring `client_credentials` does not grant machine scopes. New discovered clients persist an empty server-owned `clientCredentialsScopes` ceiling and cannot use the grant until an administrator assigns approved scopes. Metadata refresh preserves that ceiling and cannot create, widen, or clear it.
- Better Auth supports `none` and `private_key_jwt` for discovered clients. Inline `jwks` must use the RFC 7517 `{ "keys": [...] }` object shape. EC keys must use P-256, P-384, or P-521; OKP keys must use Ed25519. Bare arrays, symmetric secret methods, secret fields, private JWK material, other curves, and declared algorithms that do not match the key type and curve are rejected.
- `client_uri`, `logo_uri`, `tos_uri`, and `policy_uri` must be credential-free public HTTP(S) URLs. Better Auth does not fetch or render remote logos.
- `backchannel_logout_uri` and `backchannel_logout_session_required` are rejected because they would authorize an OP-side POST outside the injected GET transport.
- Server-owned fields, including resource links and administrative controls, cannot be supplied by the document.
- Unknown members are ignored and never persisted. Nonstandard camel-case storage aliases and both spellings of client-credentials scope authority are stripped; recognized credential, privilege, and server-control fields remain fatal.

```
{
  "client_id": "https://client.example.com/client-metadata.json",
  "grant_types": ["client_credentials"],
  "token_endpoint_auth_method": "private_key_jwt",
  "jwks_uri": "https://client.example.com/jwks.json"
}
```

## Persistence and refresh

Discovered clients are stored in `oauthClient` with `clientDiscoveryId: "cimd"`. This nullable provenance field prevents an existing managed or DCR client from being taken over merely because its identifier begins with `https://`, including during a fixed-ID registration race. After a process restart, a CIMD-owned record is fetched and refreshed only by the matching discovery. Managed records are returned without invoking CIMD, while a discovery-owned record fails closed if its owner is unavailable or cannot resolve it. Server-owned `clientCredentialsScopes` is never read from the document and is preserved exactly during refresh.

Valid documents follow shared-cache semantics. `s-maxage` takes precedence over `max-age` and `Expires`; validators support conditional revalidation. `Cache-Control: no-store`, `private`, and `Vary: *` prevent cache insertion. A `304` is accepted only when the request sent `If-None-Match` or `If-Modified-Since` for an existing validated entry. Modified responses must be exactly `200`.

The metadata fetch governor is independent from HTTP freshness. Fresh-cache hits and concurrent callers joining the same client fetch consume no budget. Every new fetch consumes its concurrency and rolling-window budget when it starts, and excess work is rejected instead of queued. A no-store response is never retained or reused; another request inside `minimumFetchInterval` therefore fails closed rather than causing an immediate refetch. The metadata cache, client pacing records, and origin budget records are each bounded by `maxCacheEntries`. The governor evicts only inactive records; if every record still protects a live interval, rolling window, or fetch, a new client or origin fails closed.

Refresh remains fail closed. A network, validation, or persistence failure preserves the previous database/cache state but rejects the current OAuth request. `onClientRefreshed` receives `previousClient`; operators can compare security-sensitive metadata and choose whether to revoke grants, tokens, or consent.

## Security boundary

The plugin enforces a five-second timeout with portable `AbortController` / `setTimeout`, a streaming 5 KB metadata-document limit, a streaming 64 KiB discovery-owned JWKS limit, JSON and `application/*+json` media types, and redirect rejection. These application-layer checks complement rather than replace the required network transport's DNS resolution and connection-pinning guarantees.

Loopback Client Identifier URLs are not supported. Tests can route HTTPS `.test` origins to in-process handlers through the injected transport without disabling URL or certificate policy. Native client loopback `redirect_uris` remain governed separately by OAuth redirect validation.

## Explicit discovery composition

```
oauthProvider({
  extensions: [
    {
      clientDiscovery: createCimdClientDiscovery({
        fetchClientMetadataResource,
        metadataRevalidationInterval: "60m",
      }),
    },
  ],
});
```

`ClientDiscovery.id` is persisted as client provenance. A discovery can also provide the focused `fetchClientMetadataResource` seam used for metadata-owned resources.

## Related

- [OAuth Provider plugin](https://better-auth.com/docs/plugins/oauth-provider)
- [IETF Client ID Metadata Document draft-02](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-02)
- [MCP authorization spec: draft-00 CIMD profile](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/client-registration#client-id-metadata-documents)
