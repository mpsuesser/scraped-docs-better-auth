---
url: https://better-auth.com/llms.txt/docs/infrastructure/services/sms
title: "Sms"
description: ""
access_date: 2026-09-01T02:53:20.078Z
current_date: 2026-09-01T02:53:20.078Z
---

Better Auth Infrastructure provides a managed SMS service for sending OTP codes for phone verification and two-factor authentication. Send verification codes without managing SMS providers.

## Overview

The SMS service offers:

- Pre-built SMS templates for common auth flows
- E.164 phone number format support
- Type-safe template variables
- No infrastructure to manage
- Global delivery support

## Installation

The SMS service is included in the `@better-auth/infra` package:

```
import { sendSMS, createSMSSender } from "@better-auth/infra";
```

## Quick Start

### Send a Single SMS

```
import { sendSMS } from "@better-auth/infra";

await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "phone-verification",
});
```

### Create a Reusable Sender

```
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

## Available Templates

### phone-verification

Sends a verification code for phone number verification.

```
await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "phone-verification",
});
```

**Example message:**

> Your verification code is 123456. It expires in 10 minutes.

### two-factor

Sends a two-factor authentication code.

```
await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "two-factor",
});
```

**Example message:**

> Your two-factor authentication code is 123456. Do not share this code with anyone.

### sign-in-otp

Sends a one-time password for passwordless sign-in.

```
await sendSMS({
  to: "+1234567890",
  code: "123456",
  template: "sign-in-otp",
});
```

**Example message:**

> Your sign-in code is 123456. It expires in 10 minutes.

### Default (No Template)

If you don't specify a template, a generic verification message is sent:

```
await sendSMS({
  to: "+1234567890",
  code: "123456",
});
```

**Example message:**

> Your verification code is 123456.

## Phone Number Format

Phone numbers must be in E.164 format:

```
+[country code][number]
```

**Examples:**

- US: `+14155551234`
- UK: `+447911123456`
- Germany: `+4915112345678`
- Japan: `+819012345678`

**Common mistakes:**

- Missing `+` prefix: `14155551234` ❌
- Including spaces: `+1 415 555 1234` ❌
- Including dashes: `+1-415-555-1234` ❌
- Including parentheses: `+1 (415) 555-1234` ❌

## Configuration

### SMSConfig

```
interface SMSConfig {
  apiKey?: string;   // Your Better Auth Infrastructure API key
  apiUrl?: string;   // Custom API URL (optional)
}
```

### Environment Variables

The SMS service automatically reads from environment variables:

```
BETTER_AUTH_API_KEY=your_api_key_here
BETTER_AUTH_API_URL=https://api.betterauth.com  # Optional
```

## API Reference

### sendSMS

Send a single SMS message.

```
async function sendSMS(
  options: SendSMSOptions,
  config?: SMSConfig
): Promise<SendSMSResult>
```

### SendSMSOptions

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| `to` | `string` | Yes | Phone number in E.164 format |
| `code` | `string` | Yes | The OTP code to send |
| `template` | `SMSTemplateId` | No | Template to use (defaults to generic) |

### createSMSSender

Create a reusable SMS sender instance.

```
const sender = createSMSSender(config?: SMSConfig);

// Use the sender
await sender.send(options: SendSMSOptions);
```

## Response Format

### SendSMSResult

```
interface SendSMSResult {
  success: boolean;
  messageId?: string;  // SMS provider message ID
  error?: string;      // Error message if failed
}
```

### Example Usage

```
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

## Error Handling

Common error scenarios:

```
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

## Integration with Better Auth

When using the `dash()` or `sentinel()` plugins with Better Auth's phone authentication, SMS messages are automatically sent for:

- Phone number verification
- Phone-based two-factor authentication
- Phone OTP sign-in

You don't need to call `sendSMS()` manually for these flows - the plugins handle it automatically.

### Better Auth Phone Plugin Integration

```
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

## Plan Requirements

| Feature | Starter | Pro | Business | Enterprise |
| --- | --- | --- | --- | --- |
| Transactional SMS | \- | Yes | Yes | Yes |

Transactional SMS is available on Pro plans and above.[Email Service](https://better-auth.com/docs/infrastructure/services/email)

[

Better Auth Infrastructure provides a managed transactional email service with pre-built templates for common authentication flows. Send verification emails, password resets, invitations, and more without managing email infrastructure.

](https://better-auth.com/docs/infrastructure/services/email)
