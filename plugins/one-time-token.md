---
url: https://better-auth.com/llms.txt/docs/plugins/one-time-token
title: "One Time Token"
description: ""
access_date: 2026-09-16T20:31:52.820Z
current_date: 2026-09-16T20:31:52.820Z
---

Generate and verify single-use token

The One-Time Token (OTT) plugin provides functionality to generate and verify secure, single-use session tokens. These are commonly used for across domains authentication.

## Installation

### Add the plugin to your auth config

To use the One-Time Token plugin, add it to your auth config.

```
import { betterAuth } from "better-auth";
import { oneTimeToken } from "better-auth/plugins/one-time-token"; 

export const auth = betterAuth({
    plugins: [
      oneTimeToken() 
    ]
    // ... other auth config
});
```

### Add the client plugin

Next, include the one-time-token client plugin in your authentication client instance.

```
import { createAuthClient } from "better-auth/client"
import { oneTimeTokenClient } from "better-auth/client/plugins"

export const authClient = createAuthClient({
    plugins: [
        oneTimeTokenClient() 
    ]
})
```

## Usage

### 1\. Generate a Token

Generate a token using `auth.api.generateOneTimeToken` or `authClient.oneTimeToken.generate`

GET/one-time-token/generate

```
const { data, error } = await authClient.oneTimeToken.generate();
```

This will return a `token` that is attached to the current session which can be used to verify the one-time token. By default, the token will expire in 3 minutes.

### 2\. Verify the Token

When the user clicks the link or submits the token, use the `auth.api.verifyOneTimeToken` or `authClient.oneTimeToken.verify` method in another API route to validate it.

POST/one-time-token/verify

```
const { data, error } = await authClient.oneTimeToken.verify({
    token: "some-token", // required, The token to verify.
});
```

Parameters

`token` stringrequired

The token to verify.

This will return the session that was attached to the token.

## Options

These options can be configured when adding the `oneTimeToken` plugin:

- **`disableClientRequest`** (boolean): Optional. If `true`, the token will only be generated on the server side. Default: `false`.
- **`expiresIn`** (number): Optional. The duration for which the token is valid in minutes. Default: `3`.

```
oneTimeToken({
    expiresIn: 10 // 10 minutes
})
```

- **`generateToken`**: A custom token generator function that takes `session` object and a `ctx` as parameters.
- **`storeToken`**: Optional. This option allows you to configure how the token is stored in your database.
	- **`plain`**: The token is stored in plain text. (Default)
		- **`hashed`**: The token is hashed using the default hasher.
		- **`custom-hasher`**: A custom hasher function that takes a token and returns a hashed token.

Examples:

```
oneTimeToken({
    storeToken: "plain"
})
```

```
oneTimeToken({
    storeToken: "hashed"
})
```

```
oneTimeToken({
    storeToken: {
        type: "custom-hasher",
        hash: async (token) => {
            return myCustomHasher(token);
        }
    }
})
```
