---
url: https://better-auth.com/llms.txt/docs/plugins/have-i-been-pwned
title: "Have I Been Pwned"
description: ""
access_date: 2026-08-03T18:54:22.481Z
current_date: 2026-08-03T18:54:22.481Z
---

# Have I Been Pwned

A plugin to check if a password has been compromised



The Have I Been Pwned plugin helps protect user accounts by preventing the use of passwords that have been exposed in known data breaches. It uses the [Have I Been Pwned](https://haveibeenpwned.com/) API to check if a password has been compromised.

Installation [#installation]

Add the plugin to your auth config [#add-the-plugin-to-your-auth-config]

```ts title="auth.ts"
import { betterAuth } from "better-auth"
import { haveIBeenPwned } from "better-auth/plugins" // [!code highlight]

export const auth = betterAuth({
    plugins: [
        haveIBeenPwned() // [!code highlight]
    ]
})
```

Usage [#usage]

When a user attempts to create an account or update their password with a compromised password, they'll receive the following default error:

```json
{
  "code": "PASSWORD_COMPROMISED",
  "message": "The password you entered has been compromised. Please choose a different password."
}
```

Options [#options]

enabled [#enabled]

Enable or disable password checks against the HIBP database. Useful for skipping checks in development or testing without removing the plugin. Defaults to `true`.

```ts title="auth.ts"
import { betterAuth } from "better-auth"
import { haveIBeenPwned } from "better-auth/plugins"

const auth = betterAuth({
    plugins: [
        haveIBeenPwned({
            enabled: process.env.NODE_ENV === 'production' // [!code highlight]
        })
    ]
})
```

customPasswordCompromisedMessage [#custompasswordcompromisedmessage]

Customize the error message shown when a compromised password is detected.

```ts title="auth.ts"
import { betterAuth } from "better-auth"
import { haveIBeenPwned } from "better-auth/plugins"

const auth = betterAuth({
    plugins: [
        haveIBeenPwned({
            customPasswordCompromisedMessage: "Please choose a more secure password." // [!code highlight]
        })
    ]
})
```

Security Notes [#security-notes]

* Only the first 5 characters of the password hash are sent to the API
* The full password is never transmitted
* Provides an additional layer of account security
