---
url: https://better-auth.com/llms.txt/docs/plugins/email-otp
title: "Email Otp"
description: ""
access_date: 2026-11-10T07:27:19.038Z
current_date: 2026-11-10T07:27:19.038Z
---

Email OTP plugin for Better Auth.

The Email OTP plugin allows user to sign in, verify their email, or reset their password using a one-time password (OTP) sent to their email address.

## Installation

### Add the plugin to your auth config

Add the `emailOTP` plugin to your auth config and implement the `sendVerificationOTP()` method.

```
import { betterAuth } from "better-auth"
import { emailOTP } from "better-auth/plugins"

export const auth = betterAuth({
    // ... other config options
    plugins: [
        emailOTP({ 
            async sendVerificationOTP({ email, otp, type }) { 
                if (type === "sign-in") { 
                    // Send the OTP for sign in
                } else if (type === "email-verification") { 
                    // Send the OTP for email verification
                } else { 
                    // Send the OTP for password reset
                } 
            }, 
        }) 
    ]
})
```

### Add the client plugin

```
import { createAuthClient } from "better-auth/client"
import { emailOTPClient } from "better-auth/client/plugins"

export const authClient = createAuthClient({
    plugins: [
        emailOTPClient() 
    ]
})
```

## Usage

### Send an OTP

Use the `sendVerificationOtp()` method to send an OTP to the user's email address.

POST/email-otp/send-verification-otp

```
const { data, error } = await authClient.emailOtp.sendVerificationOtp({
    email: "user@example.com", // required, Email address to send the OTP.
    type: "sign-in", // required, Type of the OTP. \`sign-in\`, \`email-verification\`, or \`forget-password\`.
});
```

Parameters

`email` stringrequired

Email address to send the OTP.

`type` "email-verification" | "sign-in" | "forget-password"required

Type of the OTP. `sign-in`, `email-verification`, or `forget-password`.

### Check an OTP (optional)

Use the `checkVerificationOtp()` method to check if an OTP is valid. A wrong OTP counts toward `allowedAttempts`, and a correct one stays usable for the matching sign-in, email verification, or password reset call.

POST/email-otp/check-verification-otp

```
const { data, error } = await authClient.emailOtp.checkVerificationOtp({
    email: "user@example.com", // required, Email address to send the OTP.
    type: "sign-in", // required, Type of the OTP. \`sign-in\`, \`email-verification\`, or \`forget-password\`.
    otp: "123456", // required, OTP sent to the email.
});
```

Parameters

`email` stringrequired

Email address to send the OTP.

`type` "email-verification" | "sign-in" | "forget-password"required

Type of the OTP. `sign-in`, `email-verification`, or `forget-password`.

`otp` stringrequired

OTP sent to the email.

### Sign In with OTP

To sign in with OTP, use the `sendVerificationOtp()` method to send a "sign-in" OTP to the user's email address.

POST/email-otp/send-verification-otp

```
const { data, error } = await authClient.emailOtp.sendVerificationOtp({
    email: "user@example.com", // required, Email address to send the OTP.
    type: "sign-in", // required, Type of the OTP.
});
```

Parameters

`email` stringrequired

Email address to send the OTP.

`type` "sign-in"required

Type of the OTP.

Once the user provides the OTP, you can sign in the user using the `signIn.emailOtp()` method.

POST/sign-in/email-otp

```
const { data, error } = await authClient.signIn.emailOtp({
    email: "user@example.com", // required, Email address to sign in.
    otp: "123456", // required, OTP sent to the email.
    name: "John Doe", // User display name. Only used when the user is registering for the first time.
    image: "https://example.com/image.png", // User profile image URL. Only used when the user is registering for the first time.
});
```

Parameters

`otp` stringrequired

OTP sent to the email.

`name` string

User display name. Only used when the user is registering for the first time.

`image` string

User profile image URL. Only used when the user is registering for the first time.

### Verify Email with OTP

To verify the user's email address with OTP, use the `sendVerificationOtp()` method to send an "email-verification" OTP to the user's email address.

POST/email-otp/send-verification-otp

```
const { data, error } = await authClient.emailOtp.sendVerificationOtp({
    email: "user@example.com", // required, Email address to send the OTP.
    type: "email-verification", // required, Type of the OTP.
});
```

Parameters

`email` stringrequired

Email address to send the OTP.

`type` "email-verification"required

Type of the OTP.

Once the user provides the OTP, use the `verifyEmail()` method to complete email verification.

POST/email-otp/verify-email

```
const { data, error } = await authClient.emailOtp.verifyEmail({
    email: "user@example.com", // required, Email address to verify.
    otp: "123456", // required, OTP to verify.
});
```

Parameters

`email` stringrequired

Email address to verify.

`otp` stringrequired

OTP to verify.

### Reset Password with OTP

To reset the user's password with OTP, use the `emailOtp.requestPasswordReset()` method to send a "forget-password" OTP to the user's email address.

POST/email-otp/request-password-reset

```
const { data, error } = await authClient.emailOtp.requestPasswordReset({
    email: "user@example.com", // required, Email address to send the OTP.
});
```

Parameters

`email` stringrequired

Email address to send the OTP.

Once the user provides the OTP, use the `checkVerificationOtp()` method to check if it's valid (optional).

POST/email-otp/check-verification-otp

```
const { data, error } = await authClient.emailOtp.checkVerificationOtp({
    email: "user@example.com", // required, Email address to send the OTP.
    type: "forget-password", // required, Type of the OTP.
    otp: "123456", // required, OTP sent to the email.
});
```

Parameters

`email` stringrequired

Email address to send the OTP.

`type` "forget-password"required

Type of the OTP.

`otp` stringrequired

OTP sent to the email.

Then, use the `resetPassword()` method to reset the user's password.

POST/email-otp/reset-password

```
const { data, error } = await authClient.emailOtp.resetPassword({
    email: "user@example.com", // required, Email address to reset the password.
    otp: "123456", // required, OTP sent to the email.
    password: "new-secure-password", // required, New password.
});
```

Parameters

`email` stringrequired

Email address to reset the password.

`otp` stringrequired

OTP sent to the email.

`password` stringrequired

New password.

If the new password is rejected, for example by the [Have I Been Pwned](https://better-auth.com/docs/plugins/have-i-been-pwned) plugin, the OTP isn't consumed and can be used again. If a later step of the reset fails, such as revoking sessions or saving the password, the OTP stays usable only when transactions are enabled for your database adapter and verification values are stored in the database. That is the default without secondary storage; with [secondary storage](https://better-auth.com/docs/concepts/database#secondary-storage), set [`verification.storeInDatabase: true`](https://better-auth.com/docs/reference/options#verification). A wrong OTP still counts toward [`allowedAttempts`](#options).

### Change Email with OTP

To allow users to change their email with OTP, first enable the `changeEmail` feature, which is disabled by default. Set `changeEmail.enabled` to `true`:

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    plugins: [
        emailOTP({
            changeEmail: {
                enabled: true, 
            }
        })
    ]
})
```

By default, when a user requests to change their email, an OTP is sent to the **new** email address. The email is only updated after the user verifies the new email.

#### Usage

To change the user's email address with OTP, use the `emailOtp.requestEmailChange()` method to send a "change-email" OTP to the user's new email address.

POST/email-otp/request-email-change

```
const { data, error } = await authClient.emailOtp.requestEmailChange({
    newEmail: "user@example.com", // required, New email address to send the OTP.
    otp: "123456", // OTP sent to the current email. This is required when the \`changeEmail.verifyCurrentEmail\` option is set to \`true\`.
});
```

Parameters

`newEmail` stringrequired

New email address to send the OTP.

`otp` string

OTP sent to the current email. This is required when the `changeEmail.verifyCurrentEmail` option is set to `true`.

Once the user provides the OTP, use the `changeEmail()` method to change the user's email address.

POST/email-otp/change-email

```
const { data, error } = await authClient.emailOtp.changeEmail({
    newEmail: "user@example.com", // required, New email address to change to.
    otp: "123456", // required, OTP sent to the new email.
});
```

Parameters

`newEmail` stringrequired

New email address to change to.

`otp` stringrequired

OTP sent to the new email.

#### Confirming with Current Email

For added security, you can require users to confirm the change with an OTP sent to their **current** email before sending an OTP to the new email address. To enable this, set `changeEmail.verifyCurrentEmail` to `true` in the plugin options.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    plugins: [
        emailOTP({
            changeEmail: {
                enabled: true,
                verifyCurrentEmail: true, 
            }
        })
    ]
})
```

Before requesting the email change, use the `sendVerificationOtp()` method with type `email-verification` to send an OTP to the user's email address.

POST/email-otp/send-verification-otp

```
const { data, error } = await authClient.emailOtp.sendVerificationOtp({
    email: "user@example.com", // required, Email address to send the OTP.
    type: "email-verification", // required, Type of the OTP. Must be \`email-verification\` for confirming email change.
});
```

Parameters

`email` stringrequired

Email address to send the OTP.

`type` stringrequired

Type of the OTP. Must be `email-verification` for confirming email change.

Then, the user can provide the OTP when calling `requestEmailChange()`. The system will first verify the OTP sent to the current email before sending an OTP to the new email.

### Override Default Email Verification

To override the default email verification, pass `overrideDefaultEmailVerification: true` in the options. This will make the system use an email OTP instead of the default verification link whenever email verification is triggered. In other words, the user will verify their email using an OTP rather than clicking a link.

```
import { betterAuth } from "better-auth";
import { emailOTP } from "better-auth/plugins"

export const auth = betterAuth({
  plugins: [
    emailOTP({
      overrideDefaultEmailVerification: true, 
      async sendVerificationOTP({ email, otp, type }) {
        // Implement the sendVerificationOTP method to send the OTP to the user's email address
      },
    }),
  ],
});
```

## Options

- `sendVerificationOTP`: A function that sends the OTP to the user's email address. The function receives an object with the following properties:
	- `email`: The user's email address.
		- `otp`: The OTP to send.
		- `type`: The type of OTP to send. Can be "sign-in", "email-verification", or "forget-password".
- `otpLength`: The length of the OTP. Defaults to `6`.
- `expiresIn`: The expiry time of the OTP in seconds. Defaults to `300` seconds.

```
import { betterAuth } from "better-auth"
import { emailOTP } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        emailOTP({
            otpLength: 8,
            expiresIn: 600
        })
    ]
})
```

- `sendVerificationOnSignUp`: A boolean value that determines whether to send the OTP when a user signs up. Defaults to `false`.
- `disableSignUp`: A boolean value that determines whether to prevent automatic sign-up when the user is not registered. Defaults to `false`.
- `generateOTP`: A function that generates the OTP. Defaults to a random 6-digit number.
- `allowedAttempts`: The maximum number of attempts allowed for verifying an OTP. Defaults to `3`. After exceeding this limit, the OTP becomes invalid and the user needs to request a new one.

```
import { betterAuth } from "better-auth"
import { emailOTP } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        emailOTP({
            allowedAttempts: 5, // Allow 5 attempts before invalidating the OTP
            expiresIn: 300
        })
    ]
})
```

When the maximum attempts are exceeded, the `verifyOTP`, `signIn.emailOtp`, `verifyEmail`, and `resetPassword` methods will return an error with code `TOO_MANY_ATTEMPTS`.

- `resendStrategy`: Controls what happens when a user requests a new OTP while an existing one is still valid. Defaults to `"rotate"`.
	- `"rotate"`: Always generates a new OTP (default behavior).
		- `"reuse"`: Resends the same OTP and extends its expiry. This prevents multiple valid codes from existing simultaneously when emails are delayed. Only works when the OTP is recoverable (`plain`, `encrypted`, or custom encrypt/decrypt). Falls back to `"rotate"` when the OTP is hashed. If the allowed attempts have been exhausted, a fresh OTP is generated instead of reusing the exhausted one.

```
import { betterAuth } from "better-auth"
import { emailOTP } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        emailOTP({
            resendStrategy: "reuse", 
            async sendVerificationOTP({ email, otp, type }) {
                // send the OTP
            },
        })
    ]
})
```

- `storeOTP`: The method used to transform the OTP before it is stored by Better Auth's verification layer, whether `encrypted`, `hashed` or `plain` text. Default is `plain` text.

Alternatively, you can pass a custom encryptor or hasher to control how the stored OTP value is persisted.

**Custom encryptor**

```
emailOTP({
    storeOTP: { 
        encrypt: async (otp) => {
            return myCustomEncryptor(otp);
        },
        decrypt: async (otp) => {
            return myCustomDecryptor(otp);
        },
    }
})
```

**Custom hasher**

```
emailOTP({
    storeOTP: {
        hash: async (otp) => {
            return myCustomHasher(otp);
        },
    }
})
```
