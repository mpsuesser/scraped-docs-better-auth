---
url: https://better-auth.com/llms.txt/docs/plugins/dodopayments
title: "Dodopayments"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Better Auth Plugin for Dodo Payments

[Dodo Payments](https://dodopayments.com/) is a global Merchant-of-Record platform that lets AI, SaaS and digital businesses sell in 150+ countries without touching tax, fraud, or compliance. A single, developer-friendly API powers checkout, billing, and payouts so you can launch worldwide in minutes.

### [Get support on Dodo Payments' Discord](https://discord.gg/bYqAp4ayYh)

## Features

- Automatic customer creation on sign-up
- Type-safe checkout flows with product slug mapping
- Self-service customer portal
- Real-time webhook event processing with signature verification

### [Get started with Dodo Payments](https://app.dodopayments.com/)

## Installation

Run the following command in your project root:

```
npm install @dodopayments/better-auth dodopayments better-auth zod
```

Add these to your `.env` file:

```
DODO_PAYMENTS_API_KEY=your_api_key_here
DODO_PAYMENTS_WEBHOOK_SECRET=your_webhook_secret_here
```

Create or update `src/lib/auth.ts`:

```
import { betterAuth } from "better-auth";
import {
  dodopayments,
  checkout,
  portal,
  webhooks,
} from "@dodopayments/better-auth";
import DodoPayments from "dodopayments";

export const dodoPayments = new DodoPayments({
  bearerToken: process.env.DODO_PAYMENTS_API_KEY!,
  environment: "test_mode"
});

export const auth = betterAuth({
  plugins: [
    dodopayments({
      client: dodoPayments,
      createCustomerOnSignUp: true,
      use: [
        checkout({
          products: [
            {
              productId: "pdt_xxxxxxxxxxxxxxxxxxxxx",
              slug: "premium-plan",
            },
          ],
          successUrl: "/dashboard/success",
          authenticatedUsersOnly: true,
        }),
        portal(),
        webhooks({
          webhookKey: process.env.DODO_PAYMENTS_WEBHOOK_SECRET!,
          onPayload: async (payload) => {
            console.log("Received webhook:", payload.event_type);
          },
        }),
      ],
    }),
  ],
});
```

Create or update `src/lib/auth-client.ts`:

```
import { createAuthClient } from "better-auth/react";
import { dodopaymentsClient } from "@dodopayments/better-auth";

export const authClient = createAuthClient({
  baseURL: process.env.BETTER_AUTH_URL || "http://localhost:3000",
  plugins: [dodopaymentsClient()],
});
```

## Usage

### Creating a Checkout Session

```
const { data: checkoutSession, error } =
  await authClient.dodopayments.checkoutSession({
  slug: "premium-plan",
});

if (checkoutSession) {
  window.location.href = checkoutSession.url;
}
```

### Accessing the Customer Portal

```
const { data: customerPortal, error } = await authClient.dodopayments.customer.portal();
if (customerPortal && customerPortal.redirect) {
  window.location.href = customerPortal.url;
}
```

### Listing Customer Data

```
// Get subscriptions
const { data: subscriptions, error } =
  await authClient.dodopayments.customer.subscriptions.list({
    query: {
      limit: 10,
      page: 1,
      active: true,
    },
  });

// Get payment history
const { data: payments, error } = await authClient.dodopayments.customer.payments.list({
  query: {
    limit: 10,
    page: 1,
    status: "succeeded",
  },
});
```

### Webhooks

Generate a webhook secret for your endpoint URL (e.g., `https://your-domain.com/api/auth/dodopayments/webhooks`) in the Dodo Payments Dashboard and set it in your.env file:

```
DODO_PAYMENTS_WEBHOOK_SECRET=your_webhook_secret_here
```

Example handler:

```
webhooks({
  webhookKey: process.env.DODO_PAYMENTS_WEBHOOK_SECRET!,
  onPayload: async (payload) => {
    console.log("Received webhook:", payload.event_type);
  },
});
```

## Configuration Reference

### Plugin Options

- **client** (required): DodoPayments client instance
- **createCustomerOnSignUp** (optional): Auto-create customers on user signup
- **use** (required): Array of plugins to enable (checkout, portal, webhooks)

### Checkout Plugin Options

- **products**: Array of products or async function returning products
- **successUrl**: URL to redirect after successful payment
- **authenticatedUsersOnly**: Require user authentication (default: false)

If you encounter any issues, please refer to the [Dodo Payments documentation](https://docs.dodopayments.com/) for troubleshooting steps.
