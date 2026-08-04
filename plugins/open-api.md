---
url: https://better-auth.com/llms.txt/docs/plugins/open-api
title: "Open Api"
description: ""
access_date: 2026-08-04T22:44:13.989Z
current_date: 2026-08-04T22:44:13.989Z
---

Open API reference for Better Auth.

This is a plugin that provides an Open API reference for Better Auth. It shows all endpoints added by plugins and the core. It also provides a way to test the endpoints. It uses [Scalar](https://scalar.com/) to display the Open API reference.

## Installation

### Add the plugin to your auth config

```
import { betterAuth } from "better-auth"
import { openAPI } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        openAPI(), 
    ]
})
```

### Navigate to /api/auth/reference to view the Open API reference

Each plugin endpoints are grouped by the plugin name. The core endpoints are grouped under the `Default` group. And Model schemas are grouped under the `Models` group.

![Open API reference](https://better-auth.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fopen-api-reference.4369w7jqubrsa.png&w=3840&q=75&dpl=dpl_7xojRigL65XEaoUN3iUdGhB19upm)

## Usage

The Open API reference is generated using the [OpenAPI 3.1.1](https://swagger.io/specification/) specification. You can use the reference to generate client libraries, documentation, and more. If you use the generated schema with client generators or other tooling, make sure they support OpenAPI 3.1 semantics.

The reference is generated using the [Scalar](https://scalar.com/) library. Scalar provides a way to view and test the endpoints. You can test the endpoints by clicking on the `Try it out` button and providing the required parameters.

![Open API reference](https://better-auth.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fopen-api-reference.4369w7jqubrsa.png&w=3840&q=75&dpl=dpl_7xojRigL65XEaoUN3iUdGhB19upm)

### Generated Schema

To get the generated Open API schema directly as JSON, you can do `auth.api.generateOpenAPISchema()`. This will return the Open API schema as a JSON object.

```
import { auth } from "@/lib/auth"

const openAPISchema = await auth.api.generateOpenAPISchema()
console.log(openAPISchema)
```

### Using Scalar with Multiple Sources

If you're using Scalar for your API documentation, you can add Better Auth as an additional source alongside your main API:

When using Hono with Scalar for OpenAPI documentation, you can integrate Better Auth by adding it as a source:

```
app.get("/docs", Scalar({
  pageTitle: "API Documentation", 
  sources: [
    { url: "/api/open-api", title: "API" },
    // Better Auth schema generation endpoint
    { url: "/api/auth/open-api/generate-schema", title: "Auth" },
  ],
}));
```

## Configuration

`path` - The path where the Open API reference is served. Default is `/api/auth/reference`. You can change it to any path you like, but keep in mind that it will be appended to the base path of your auth server.

`disableDefaultReference` - If set to `true`, the default Open API reference UI by Scalar will be disabled. Default is `false`.

This allows you to display both your application's API and Better Auth's authentication endpoints in a unified documentation interface.

`theme` - Allows you to change the theme of the OpenAPI reference page. Default is `default`.

`nonce` - Allows you to pass a nonce string to the inline scripts for Content Security Policy (CSP) compliance. Default is `undefined`.
