---
url: https://better-auth.com/llms.txt/docs/infrastructure/services/sms
title: "Sms"
description: ""
access_date: 2026-08-03T18:54:22.481Z
current_date: 2026-08-03T18:54:22.481Z
---

# SMS Service

Better Auth Infrastructure provides a managed SMS service for sending OTP codes for phone verification and two-factor authentication. Send verification codes without managing SMS providers.



Overview [#overview]

The SMS service offers:

* Pre-built SMS templates for common auth flows
* E.164 phone number format support
* Type-safe template variables
* No infrastructure to manage
* Global delivery support

<Callout type="warn">
  Note that SMS delivery is intended to only be used for authentication flows.
</Callout>

Installation [#installation]

The SMS service is included in the `@better-auth/infra` package:

```ts
import { sendSMS, createSMSSender } from "@better-auth/infra";
```

Quick Start [#quick-start]

Send a Single SMS [#send-a-single-sms]

```ts
import { sendSMS } from "@better-auth/infra";

await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "phone-verification",
});
```

Create a Reusable Sender [#create-a-reusable-sender]

```ts
import { createSMSSender } from "@better-auth/infra";

const smsSender = createSMSSender({
  apiKey: process.env.BETTER_AUTH_API_KEY,
  apiUrl: process.env.BETTER_AUTH_API_URL,
});

// Send multiple SMS messages
await smsSender.send({
  to: "+1234567890",
  code: "123456",
  template: "two-factor",
});
```

Available Templates [#available-templates]

phone-verification [#phone-verification]

Sends a verification code for phone number verification.

```ts
await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "phone-verification",
});
```

**Example message:**

> Your verification code is 123456. It expires in 10 minutes.

two-factor [#two-factor]

Sends a two-factor authentication code.

```ts
await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "two-factor",
});
```

**Example message:**

> Your two-factor authentication code is 123456. Do not share this code with anyone.

sign-in-otp [#sign-in-otp]

Sends a one-time password for passwordless sign-in.

```ts
await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "sign-in-otp",
});
```

**Example message:**

> Your sign-in code is 123456. It expires in 10 minutes.

Default (No Template) [#default-no-template]

If you don't specify a template, a generic verification message is sent:

```ts
await sendSMS({
  to: "+1234567890",
  code: "123456",
});
```

**Example message:**

> Your verification code is 123456.

Phone Number Format [#phone-number-format]

Phone numbers must be in E.164 format:

```
+[country code][number]
```

**Examples:**

* US: `+14155551234`
* UK: `+447911123456`
* Germany: `+4915112345678`
* Japan: `+819012345678`

**Common mistakes:**

* Missing `+` prefix: `14155551234` ❌
* Including spaces: `+1 415 555 1234` ❌
* Including dashes: `+1-415-555-1234` ❌
* Including parentheses: `+1 (415) 555-1234` ❌

Configuration [#configuration]

SMSConfig [#smsconfig]

```ts
interface SMSConfig {
  apiKey?: string;   // Your Better Auth Infrastructure API key
  apiUrl?: string;   // Custom API URL (optional)
}
```

Environment Variables [#environment-variables]

The SMS service automatically reads from environment variables:

```dotenv
BETTER_AUTH_API_KEY=your_api_key_here
BETTER_AUTH_API_URL=https://api.betterauth.com  # Optional
```

API Reference [#api-reference]

sendSMS [#sendsms]

Send a single SMS message.

```ts
async function sendSMS(
  options: SendSMSOptions,
  config?: SMSConfig
): Promise<SendSMSResult>
```

SendSMSOptions [#sendsmsoptions]

| Property   | Type            | Required | Description                           |
| ---------- | --------------- | -------- | ------------------------------------- |
| `to`       | `string`        | Yes      | Phone number in E.164 format          |
| `code`     | `string`        | Yes      | The OTP code to send                  |
| `template` | `SMSTemplateId` | No       | Template to use (defaults to generic) |

createSMSSender [#createsmssender]

Create a reusable SMS sender instance.

```ts
const sender = createSMSSender(config?: SMSConfig);

// Use the sender
await sender.send(options: SendSMSOptions);
```

Response Format [#response-format]

SendSMSResult [#sendsmsresult]

```ts
interface SendSMSResult {
  success: boolean;
  messageId?: string;  // SMS provider message ID
  error?: string;      // Error message if failed
}
```

Example Usage [#example-usage]

```ts
const result = await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "phone-verification",
});

if (result.success) {
  console.log("SMS sent:", result.messageId);
} else {
  console.error("Failed to send SMS:", result.error);
}
```

Error Handling [#error-handling]

Common error scenarios:

```ts
const result = await sendSMS({
  to: "+1234567890",
  code: "123456",
});

if (!result.success) {
  switch (result.error) {
    case "API key not configured":
      // Missing BETTER_AUTH_API_KEY
      break;
    case "Invalid phone number":
      // Phone number not in E.164 format
      break;
    default:
      // Other delivery error
      console.error("SMS error:", result.error);
  }
}
```

Integration with Better Auth [#integration-with-better-auth]

When using the `dash()` or `sentinel()` plugins with Better Auth's phone authentication, SMS messages are automatically sent for:

* Phone number verification
* Phone-based two-factor authentication
* Phone OTP sign-in

You don't need to call `sendSMS()` manually for these flows - the plugins handle it automatically.

Better Auth Phone Plugin Integration [#better-auth-phone-plugin-integration]

```ts
import { betterAuth } from "better-auth";
import { phoneNumber } from "better-auth/plugins";
import { dash } from "@better-auth/infra";

export const auth = betterAuth({
  plugins: [
    phoneNumber({
      sendOTP: async ({ phoneNumber, code }) => {
        // This is handled automatically when dash() is configured
        // But you can customize if needed:
        await sendSMS({
          to: phoneNumber,
          code,
          template: "phone-verification",
        });
      },
    }),
    dash({
      apiKey: process.env.BETTER_AUTH_API_KEY,
    }),
  ],
});
```

Plan Requirements [#plan-requirements]

| Feature           | Starter | Pro | Business | Enterprise |
| ----------------- | ------- | --- | -------- | ---------- |
| Transactional SMS | -       | Yes | Yes      | Yes        |

Transactional SMS is available on Pro plans and above.
