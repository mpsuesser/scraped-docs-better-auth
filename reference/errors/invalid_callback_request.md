---
url: https://better-auth.com/llms.txt/docs/reference/errors/invalid_callback_request
title: "Invalid_callback_request"
description: ""
access_date: 2026-09-25T19:50:31.093Z
current_date: 2026-09-25T19:50:31.093Z
---

The callback request is invalid.

## What is it?

This error is thrown during the OAuth callback when the incoming request cannot be parsed or is missing required fields.

## Common Causes

- Query or body parameters were stripped by a reverse proxy, CDN, or framework rewrite.
- Double-encoding or improper URL encoding of parameters causes parsing to fail.
- Callback URL mismatch at the provider triggers an intermediate redirect that drops parameters.
- Middleware or route grouping sends the request to a different handler than intended.
- Very long URLs get truncated by an intermediary (rare but possible with some proxies).

## How to resolve

### Verify callback method and parameters

- Ensure your provider is configured to use the method your route expects (commonly GET with query parameters for Authorization Code flow).
- Confirm the callback includes required parameters (e.g., `code` and `state` for standard OAuth flows).

### Preserve query/body through infrastructure

- Check that reverse proxies (Vercel, Cloudflare, Nginx) and app rewrites forward the full query string and request body intact.
- If middleware intercepts or rewrites the callback, make sure it forwards all parameters without modification.

### Debug locally

- In DevTools → Network, inspect the callback request and verify parameters are present and well-formed.
- Compare dev/staging/prod credentials to ensure there are no environment differences causing different flows or endpoints.

### Edge cases to consider

- Mobile/WebView or deep-link flows can drop query parameters during handoff.
- Some providers can return parameters in fragments; your server will not receive fragments—ensure the provider uses query/body for server-side callbacks.
- Multiple redirects (including HTTP → HTTPS) can lose parameters if not configured correctly.
