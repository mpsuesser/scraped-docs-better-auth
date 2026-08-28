---
url: https://better-auth.com/llms.txt/docs/plugins/phone-number
title: "Phone Number"
description: ""
access_date: 2026-08-28T22:27:55.614Z
current_date: 2026-08-28T22:27:55.614Z
---

Phone number plugin

The phone number plugin extends the authentication system by allowing users to sign in and sign up using their phone number. It includes OTP (One-Time Password) functionality to verify phone numbers.

## Installation

### Add Plugin to the server

```
import { betterAuth } from "better-auth"
import { phoneNumber } from "better-auth/plugins"

const auth = betterAuth({
    plugins: [ 
        phoneNumber({  
            sendOTP: ({ phoneNumber, code }, ctx) => { 
                // Implement sending OTP code via SMS
            } 
        }) 
    ] 
})
```

### Migrate the database

Run the migration or generate the schema to add the necessary fields and tables to the database.

#### migrate

#### npm

#### generate

```
npx auth migrate
```

#### pnpm

#### yarn

#### bun

See the [Schema](#schema) section to add the fields manually.

### Add the client plugin

```
import { createAuthClient } from "better-auth/client"
import { phoneNumberClient } from "better-auth/client/plugins"

const authClient = createAuthClient({
    plugins: [
        phoneNumberClient() 
    ]
})
```

## Usage

### Send OTP for Verification

To send an OTP to a user's phone number for verification, you can use the `sendVerificationCode` endpoint.

POST/phone-number/send-otp

```
const { data, error } = await authClient.phoneNumber.sendOtp({
    phoneNumber: "+1234567890", // required, Phone number to send OTP.
});
```

Parameters

`phoneNumber` stringrequired

Phone number to send OTP.

### Verify Phone Number

After the OTP is sent, users can verify their phone number by providing the code.

POST/phone-number/verify

```
const { data, error } = await authClient.phoneNumber.verify({
    phoneNumber: "+1234567890", // required, Phone number to verify.
    code: "123456", // required, OTP code.
    disableSession: false, // Disable session creation after verification.
    updatePhoneNumber: false, // Update the phone number of an existing logged-in user. Requires an active session.
});
```

Parameters

`phoneNumber` stringrequired

Phone number to verify.

`code` stringrequired

OTP code.

`disableSession` boolean

Disable session creation after verification.

`updatePhoneNumber` boolean

Update the phone number of an existing logged-in user. Requires an active session.

### Consume an OTP on the Server

For custom sign-up or account-linking flows, consume an OTP without creating or updating a user or session. This server-only API is exposed through `auth.api`; it does not register an HTTP route or generate an `authClient` method.

```
const result = await auth.api.consumePhoneNumberOTP({
    body: {
        phoneNumber: "+1234567890",
        code: "123456"
    }
})
```

This method uses the same validation and consumption behavior as `verifyPhoneNumber`; it does not introduce a stronger concurrency guarantee. If your server flow can receive parallel redemption attempts and requires strict single-use acceptance, configure [`verifyOTP`](#verifyotp) with a provider that atomically consumes accepted codes.

Call this method inside the server-side flow that uses the verification result; it does not return a session or reusable proof of verification. Validate any inputs that do not depend on successful OTP verification first. If you expose this capability through your own public endpoint, apply rate limiting and appropriate abuse protection there.

### Allow Sign-Up with Phone Number

To allow users to sign up using their phone number, you can pass `signUpOnVerification` option to your plugin configuration. It requires you to pass `getTempEmail` function to generate a temporary email for the user.

```
import { betterAuth } from "better-auth";
import { phoneNumber } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        phoneNumber({
            sendOTP: ({ phoneNumber, code }, ctx) => {
                // Implement sending OTP code via SMS
            },
            signUpOnVerification: {
                getTempEmail: (phoneNumber) => {
                    return \`${phoneNumber}@my-site.com\`
                },
                //optionally, you can also pass \`getTempName\` function to generate a temporary name for the user
                getTempName: (phoneNumber) => {
                    return phoneNumber //by default, it will use the phone number as the name
                }
            }
        })
    ]
})
```

If you have additional required fields in your user schema, you can pass them in the verify request body:

```
await authClient.phoneNumber.verify({
    phoneNumber: "+1234567890",
    code: "123456",
    customField: "custom-value", // additional field
})
```

### Sign In with Phone Number

In addition to signing in a user using send-verify flow, you can also use phone number as an identifier and sign in a user using phone number and password.

POST/sign-in/phone-number

```
const { data, error } = await authClient.signIn.phoneNumber({
    phoneNumber: "+1234567890", // required, Phone number to sign in.
    password, // required, Password to use for sign in.
    rememberMe: true, // Remember the session.
});
```

Parameters

`rememberMe` boolean

Remember the session.

### Update Phone Number

Already logged-in users can change their phone number to a new one. First, send an OTP to the new phone number:

```
import { authClient } from "@/lib/auth-client";

await authClient.phoneNumber.sendOtp({
    phoneNumber: "+1234567890" // New phone number
})
```

Then verify the new phone number with `updatePhoneNumber: true`:

```
import { authClient } from "@/lib/auth-client";

const isVerified = await authClient.phoneNumber.verify({
    phoneNumber: "+1234567890",
    code: "123456",
    updatePhoneNumber: true
})
```

### Remove Phone Number

Logged-in users can release their phone number by passing `null` to `updateUser`. The plugin atomically clears the phone number and resets the verified flag, freeing the number so another account can claim it through the standard verification flow.

```
import { authClient } from "@/lib/auth-client";

await authClient.updateUser({
    phoneNumber: null
})
```

For security, non-null phone number updates through `updateUser` remain blocked. Changing to a different phone number always requires OTP verification via `verify` with `updatePhoneNumber: true`.

### Disable Session Creation

By default, the plugin creates a session for the user after verifying the phone number. You can disable this behavior by passing `disableSession: true` to the `verify` method.

```
import { authClient } from "@/lib/auth-client";

const isVerified = await authClient.phoneNumber.verify({
    phoneNumber: "+1234567890",
    code: "123456",
    disableSession: true
})
```

### Request Password Reset

To initiate a request password reset flow using `phoneNumber`, you can start by calling `requestPasswordReset` on the client to send an OTP code to the user's phone number.

POST/phone-number/request-password-reset

```
const { data, error } = await authClient.phoneNumber.requestPasswordReset({
    phoneNumber: "+1234567890", // required, The phone number which is associated with the user.
});
```

Parameters

`phoneNumber` stringrequired

The phone number which is associated with the user.

Then, you can reset the password by calling `resetPassword` on the client with the OTP code and the new password.

POST/phone-number/reset-password

```
const { data, error } = await authClient.phoneNumber.resetPassword({
    otp: "123456", // required, The one time password to reset the password.
    phoneNumber: "+1234567890", // required, The phone number to the account which intends to reset the password for.
    newPassword: "new-and-secure-password", // required, The new password.
});
```

Parameters

`otp` stringrequired

The one time password to reset the password.

`phoneNumber` stringrequired

The phone number to the account which intends to reset the password for.

`newPassword` stringrequired

The new password.

## Options

### otpLength

The length of the OTP code to be generated. Default is `6`.

### sendOTP

A function that sends the OTP code to the user's phone number. It takes the phone number and the OTP code as arguments.

### expiresIn

The time in seconds after which the OTP code expires. Default is `300` seconds.

### callbackOnVerification

A function that is called after the phone number is verified. It takes the phone number and the user object as the first argument and a request object as the second argument.

```
import { betterAuth } from "better-auth";
import { phoneNumber } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        phoneNumber({
            sendOTP: ({ phoneNumber, code }, ctx) => {
                // Implement sending OTP code via SMS
            },
            callbackOnVerification: async ({ phoneNumber, user }, ctx) => { 
                // Implement callback after phone number verification
            } 
        })
    ]
})
```

### sendPasswordResetOTP

A function that sends the OTP code to the user's phone number for password reset. It takes the phone number and the OTP code as arguments.

### phoneNumberValidator

A custom function to validate the phone number. It takes the phone number as an argument and returns a boolean indicating whether the phone number is valid.

### verifyOTP

A custom function to verify the OTP code. When provided, this function will be used instead of the default internal verification logic. This is useful when you want to integrate with external SMS providers that handle OTP verification (e.g., Twilio Verify, AWS SNS). The function takes an object with `phoneNumber` and `code` properties and a request object, and returns a boolean or a promise that resolves to a boolean indicating whether the OTP is valid.

```
import { betterAuth } from "better-auth";
import { phoneNumber } from "better-auth/plugins"

export const auth = betterAuth({
    plugins: [
        phoneNumber({
            sendOTP: ({ phoneNumber, code }, ctx) => {
                // Send OTP via your SMS provider
            },
            verifyOTP: async ({ phoneNumber, code }, ctx) => { 
                // Verify OTP with your desired logic (e.g., Twilio Verify)
                // This is just an example, not a real implementation.
                const isValid = await twilioClient.verify 
                    .services('YOUR_SERVICE_SID') 
                    .verificationChecks 
                    .create({ to: phoneNumber, code }); 
                return isValid.status === 'approved'; 
            } 
        })
    ]
})
```

### signUpOnVerification

An object with the following properties:

- `getTempEmail`: A function that generates a temporary email for the user. It takes the phone number as an argument and returns the temporary email.
- `getTempName`: A function that generates a temporary name for the user. It takes the phone number as an argument and returns the temporary name.

### requireVerification

When enabled, users cannot sign in with their phone number until it has been verified. If an unverified user attempts to sign in, the server will respond with a 401 error (PHONE\_NUMBER\_NOT\_VERIFIED) and automatically trigger an OTP send to start the verification process.

## Schema

The plugin requires 2 fields to be added to the user table

### User Table

Table

Field

Type

Attributes

Description

phoneNumber?

string

UQ

The phone number of the user

phoneNumberVerified?

boolean

\-

Whether the phone number is verified or not

### OTP Verification Attempts

The phone number plugin includes a built-in protection against brute force attacks by limiting the number of verification attempts for each OTP code.

```
phoneNumber({
  allowedAttempts: 3, // default is 3
  // ... other options
})
```

When a user exceeds the allowed number of verification attempts:

- The OTP code is automatically deleted
- Further verification attempts will return a 403 (Forbidden) status with "Too many attempts" message
- The user will need to request a new OTP code to continue

Example error response after exceeding attempts:

```
{
  "error": {
    "status": 403,
    "message": "Too many attempts"
  }
}
```
