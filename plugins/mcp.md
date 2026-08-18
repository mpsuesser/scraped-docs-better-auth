---
url: https://better-auth.com/llms.txt/docs/plugins/mcp
title: "Mcp"
description: ""
access_date: 2026-08-18T00:08:46.984Z
current_date: 2026-08-18T00:08:46.984Z
---

Turn your Better Auth server into an OAuth provider for MCP clients

`OAuth` `MCP`

The **MCP** plugin lets your app act as an OAuth authorization server and protected resource for [Model Context Protocol](https://modelcontextprotocol.io/) clients. It is built on the [OAuth 2.1 Provider](https://better-auth.com/docs/plugins/oauth-provider), so MCP clients discover your endpoints and obtain resource-bound access tokens through standard OAuth flows.

`mcp()` configures the OAuth provider with MCP resource binding and serves the [RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728) protected resource metadata. For the MCP 2026-07-28 profile, compose it with [Client ID Metadata Documents](https://better-auth.com/docs/plugins/cimd) and select the profile that pins CIMD draft-00. MCP deprecates Dynamic Client Registration (DCR), so Better Auth never enables DCR implicitly.

## Installation

### Install the packages

#### npm

```
npm install @better-auth/mcp @better-auth/cimd @modelcontextprotocol/server zod
```

#### pnpm

#### yarn

#### bun

### Configure authorization

Add the MCP plugin to your auth configuration alongside the [JWT plugin](https://better-auth.com/docs/plugins/jwt). The JWT plugin is required: it provides the stable signing key used for ID tokens and access tokens, and exposes the `/jwks` endpoint that resource servers use to verify them.

```
import { betterAuth } from "better-auth";
import { jwt } from "better-auth/plugins"; 
import { cimd } from "@better-auth/cimd"; 
import { mcp } from "@better-auth/mcp"; 
import { fetchClientMetadataResource } from "@better-auth/cimd/node"; 

export const auth = betterAuth({
    plugins: [
        jwt(), 
        mcp({ 
            loginPage: "/sign-in", // path to your login page
            consentPage: "/consent", // path to your consent page
            resource: "https://api.example.com/mcp" // protected resource identifier
        }), 
        cimd({ 
            fetchClientMetadataResource, 
            metadataProfile: "mcp-2026-07-28", 
        }), 
    ]
});
```

Node.js deployments can use the bundled transport shown above. Bun, Deno, Workers, and other runtimes must inject an equivalent transport that resolves each hostname once, rejects RFC 6890 special-use addresses, pins the approved address for the connection, and refuses redirects. The transport retrieves both the Client ID Metadata Document and discovery-owned resources such as `jwks_uri`. See the [CIMD security boundary](https://better-auth.com/docs/plugins/cimd#security-boundary) for the complete contract.

### Generate Schema

Run the migration or generate the schema to add the necessary tables to the database.

#### migrate

#### npm

#### generate

```
npx auth migrate
```

#### pnpm

#### yarn

#### bun

The MCP plugin uses the same schema as the OAuth Provider plugin (`oauthClient`, `oauthAccessToken`, `oauthRefreshToken`, `oauthConsent`, `oauthClientAssertion`). See the [OAuth Provider Schema](https://better-auth.com/docs/plugins/oauth-provider#schema) section for details.

## Endpoints

`mcp()` serves the standard OAuth 2.1 endpoints under `/oauth2/*`:

| Endpoint | Path |
| --- | --- |
| Authorization | `/oauth2/authorize` |
| Token | `/oauth2/token` |
| Dynamic registration (when explicitly enabled) | `/oauth2/register` |
| UserInfo | `/oauth2/userinfo` |

Discovery follows OAuth 2.0 Authorization Server Metadata ([RFC 8414](https://datatracker.ietf.org/doc/html/rfc8414)) and Protected Resource Metadata ([RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728)). The well-known URLs are derived from the issuer, so a server with a base path serves them at the issuer-inserted location rather than the bare root. Discovery advertises `client_id_metadata_document_supported` only when `cimd()` is installed, and `registration_endpoint` only when DCR is enabled.

## Usage

### Add device authorization for your own CLI

MCP clients normally discover the authorization server from protected resource metadata, then use the authorization code flow with PKCE. Keep that flow for general MCP compatibility.

If your product also ships a command-line application, the same authorization server can let the CLI authorize through a browser without opening a local callback listener. Add the OAuth Device Authorization integration:

```
import { betterAuth } from "better-auth";
import { jwt } from "better-auth/plugins";
import { oauthDeviceAuthorization } from "@better-auth/oauth-provider";
import { mcp } from "@better-auth/mcp";

export const auth = betterAuth({
  plugins: [
    jwt(),
    mcp({
      loginPage: "/sign-in",
      consentPage: "/consent",
      resource: "https://api.example.com/mcp",
      scopes: ["openid", "profile", "offline_access", "mcp:read"],
    }),
    oauthDeviceAuthorization({ verificationUri: "/device" }),
  ],
});
```

`mcp()` is already the OAuth Provider, so this composition does not add a separate `oauthProvider()` plugin. MCP clients continue to use discovery and authorization code with PKCE. Your registered public CLI can request the same MCP resource through `/device/code` and poll `/oauth2/token` for an audience-bound access token.

Only use the device grant in an MCP client that explicitly implements [RFC 8628](https://datatracker.ietf.org/doc/html/rfc8628). Adding `oauthDeviceAuthorization()` to the server does not change the standard flow chosen by existing MCP clients. See [Authorize a CLI to call an API](https://better-auth.com/docs/plugins/device-authorization#authorize-a-cli-to-call-an-api) for client registration and token polling.

### Protected Resource Metadata

The [RFC 9728](https://datatracker.ietf.org/doc/html/rfc9728) `/.well-known/oauth-protected-resource` document is served automatically at the well-known root (and the resource-path-inserted alias). It tells MCP clients which authorization server protects the resource, which scopes it supports, which `resource` identifier their access tokens must be bound to, and which DPoP proof algorithms are supported.

Set `resource` to the protected resource identifier MCP clients request and access tokens carry as `aud`:

```
mcp({
    loginPage: "/sign-in",
    consentPage: "/consent",
    resource: "https://api.example.com/mcp"
})
```

`resource` must be an HTTPS URL with no query, fragment, or credentials; HTTP is accepted only on loopback hosts for local development. A resource whose URL requires a query component cannot use `mcp()`, `requireMcpAuth`, or `createMcpProtectedRequestHandler`. Verify its tokens with `verifyAccessTokenRequest` from `better-auth/oauth2`, and build challenges with `createResourceServerChallenge` from `@better-auth/oauth-provider`.

`mcp()` also registers this identifier as a default client-registration resource. A dynamically registered client is linked to the MCP resource even when its registration request omits the non-standard `resources` field. Any `clientRegistrationDefaultResources` you provide are preserved, and the MCP resource is appended once.

### Optional DCR fallback

CIMD is the recommended client identity mechanism. If you deliberately support older clients that still require DCR, enable both provider controls explicitly:

```
mcp({
  loginPage: "/sign-in",
  consentPage: "/consent",
  resource: "https://api.example.com/mcp",
  allowDynamicClientRegistration: true,
  allowUnauthenticatedClientRegistration: true,
})
```

The registration endpoint is absent from discovery unless DCR is enabled. A DCR request does not need Better Auth's `resources` extension: `mcp()` adds its canonical resource as a server-owned default.

### Protecting an MCP Route

MCP 2026-07-28 uses a stateless request and response model: every client JSON-RPC request or notification is an independent HTTP `POST`, and the server does not maintain a protocol-level session between requests. Use the official MCP TypeScript SDK v2 to implement that transport, then wrap its `POST` handler with `requireMcpAuth`.

The following Next.js route creates a fresh MCP server for each request and rejects traffic from the session-oriented 2025 protocol. It exports only `POST`, so the framework responds to `GET` and `DELETE` with `405 Method Not Allowed`.

```
import { auth } from "@/lib/auth";
import { requireMcpAuth } from "@better-auth/mcp"; 
import { createMcpHandler, McpServer } from "@modelcontextprotocol/server"; 
import * as z from "zod";

const resource = "https://api.example.com/mcp";

const mcpServerHandler = createMcpHandler(
    () => {
        const server = new McpServer({
            name: "example-mcp-server",
            version: "1.0.0",
        });

        server.registerTool(
            "echo",
            {
                description: "Echo a message",
                inputSchema: z.object({
                    message: z.string(),
                }),
            },
            async ({ message }) => ({
                content: [{ type: "text", text: \`Tool echo: ${message}\` }],
            }),
        );

        return server;
    },
    {
        legacy: "reject", // accept only the MCP 2026-07-28 protocol
    },
);

const POST = requireMcpAuth(
    auth,
    (request) => mcpServerHandler.fetch(request),
    {
        resource, // must match mcp({ resource })
    },
);

export { POST };
```

`createMcpHandler` returns JSON or request-scoped Server-Sent Events (SSE) as required by the operation; it does not require a Redis-backed MCP session store. If you implement `subscriptions/listen` across multiple server instances, provide the SDK with a shared event bus for that extension. OAuth clients, consent, authorization codes, refresh tokens, CIMD metadata, and DPoP replay detection still use durable state because they are authorization and security records, not MCP transport sessions.

`requireMcpAuth` reads the `Authorization` header, verifies access tokens against the authorization server's JSON Web Key Set (JWKS), and checks the signature, issuer, audience, and expiry. It also enforces [RFC 9449](https://datatracker.ietf.org/doc/html/rfc9449) Demonstrating Proof of Possession (DPoP) when an access token is DPoP-bound. Unauthenticated requests receive a JSON-RPC `401` with the RFC 9728 `WWW-Authenticate` header, so MCP clients can start the authorization flow. Tokens missing a required scope receive a `403` with an [RFC 6750](https://datatracker.ietf.org/doc/html/rfc6750#section-3.1) `insufficient_scope` challenge, which MCP clients use to step up their authorization.

The wrapper passes the verified access-token claims as a second callback argument, not as a database record. The example keeps authentication at the route boundary; if a tool needs token context, convert those claims to the SDK's `AuthInfo` and pass it through `mcpServerHandler.fetch(request, { authInfo })`. `requireMcpAuth` never exposes a refresh token, and it checks the access token locally against the JWKS without a database round trip. DPoP replay protection uses your auth instance's database adapter by default so it works across server instances; pass `dpop.replayStore` only when you need a different shared store.

By default, `requireMcpAuth` reads the server's resolved Better Auth URL from the auth context and uses it as the expected issuer, resource, and JWKS base. If `mcp({ resource })` uses a different identifier, pass that same value to `requireMcpAuth`, as shown above. Override the issuer or JWKS URL when `jwt.issuer` is custom or the authorization server runs separately:

```
requireMcpAuth(auth, handler, {
    resource: "https://api.example.com/mcp", // protected resource identifier
    issuer: "https://auth.example.com", // override when jwt.issuer is custom
    jwksUrl: "https://auth.example.com/api/auth/jwks",
    challengeScopes: ["openid", "profile"] // advertised in the 401 challenge
})
```

To require scopes, pass `requiredScopes`. A token missing any of them is rejected with a `403` and an `insufficient_scope` challenge naming every missing scope, so the client can re-authorize for all of them at once ([step-up authorization](https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization#step-up-authorization-flow)):

```
requireMcpAuth(auth, handler, {
    resource: "https://api.example.com/mcp",
    requiredScopes: ["mcp:tools"], // enforced against the token's scope claim
})
```

When the required scopes depend on the request (one tool needs more than another), throw `createInsufficientScopeError` from the handler to challenge for exactly those scopes:

```
import { createInsufficientScopeError } from "better-auth/oauth2";

requireMcpAuth(auth, async (request, accessTokenClaims) => {
    if (isAdminTool(request) && !hasScope(accessTokenClaims, "mcp:admin")) {
        throw createInsufficientScopeError(["mcp:admin"]);
    }
    return executeMcpRequest(request, accessTokenClaims);
})
```

## Configuration

`mcp()` extends the [OAuth Provider](https://better-auth.com/docs/plugins/oauth-provider#configuration) options. All OAuth provider options are passed flat (there is no nested `oidcConfig`). The required MCP-specific option is `resource`.

For every client configured through `mcp()`, the plugin defaults `refreshTokenReuseInterval` to 30 seconds. This lets a client retry a refresh with the old token and receive the same rotated token response when another request already consumed that refresh token. OAuth Provider itself remains strict by default; set `refreshTokenReuseInterval: 0` on `mcp()` to disable the overlap window.

Prop

The most commonly tuned OAuth provider options are below. See the [OAuth Provider Configuration](https://better-auth.com/docs/plugins/oauth-provider#configuration) for the full list.

Prop

## Remote MCP Server

`requireMcpAuth` verifies tokens against your Better Auth server's JWKS, so the MCP route can run anywhere as long as it can reach that JWKS URL. When the resource server runs separately from the authorization server, or uses a dynamic `baseURL`, use `createMcpProtectedRequestHandler` with explicit verification options instead of `requireMcpAuth`.

```
import { createMcpProtectedRequestHandler } from "@better-auth/mcp"; 

const handler = createMcpProtectedRequestHandler( 
    {
        issuer: "https://auth.example.com",
        audience: "https://api.example.com/mcp",
        jwksUrl: "https://auth.example.com/api/auth/jwks",
    },
    async (request, accessTokenClaims) => { 
        // accessTokenClaims holds the verified access-token claims
        return new Response(JSON.stringify({
            jsonrpc: "2.0",
            result: { sub: accessTokenClaims.sub },
            id: 1
        }))
    }
)
```

`createMcpProtectedRequestHandler` returns the same RFC 9728 `WWW-Authenticate` response for unauthenticated requests. Set `requiredScopes` in its options to require scopes; a token missing any of them receives the same `403` `insufficient_scope` challenge as `requireMcpAuth`. Set `challengeScopes` in the same options object to advertise a scope hint on unauthenticated challenges.
