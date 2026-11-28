---
url: https://better-auth.com/llms.txt/docs/plugins/commet
title: "Commet"
description: ""
access_date: 2026-11-28T17:35:03.884Z
current_date: 2026-11-28T17:35:03.884Z
---

Better Auth Plugin for Billing and Subscriptions using Commet

[Commet](https://commet.co/) is a Merchant of Record that handles subscriptions, usage-based billing, feature gating, taxes and global payments. This plugin integrates Commet with Better Auth, connecting your authentication layer to billing and feature access through a composable set of sub-plugins.

## Features

- Automatic customer creation on signup
- Customer portal for self-service billing management
- Subscription management (get, cancel)
- Feature access control (boolean, metered and seat-based)
- Usage tracking for metered billing
- Seat management for per-user pricing
- Secure webhook handling with signature verification

## Installation

#### npm

```
npm install better-auth @commet/better-auth @commet/node
```

#### pnpm

#### yarn

#### bun

## Preparation

Get your API key from the [Commet dashboard](https://commet.co/) and add it to your environment.

```
COMMET_API_KEY=ck_...
```

### Configuring BetterAuth Server

The Commet plugin comes with a set of sub-plugins that add functionality to your stack. Add only the ones you need.

- Portal — Redirect customers to a self-service billing portal
- Subscriptions — Get and cancel the customer's subscription
- Features — Check feature access for the authenticated user
- Usage — Track usage events for metered billing
- Seats — Manage seat-based licenses
- Webhooks — Handle Commet webhooks with signature verification

```
import { betterAuth } from "better-auth";
import {
  commet,
  portal,
  subscriptions,
  features,
  usage,
  seats,
} from "@commet/better-auth";
import { Commet } from "@commet/node";

const commetClient = new Commet({
  apiKey: process.env.COMMET_API_KEY!,
});

export const auth = betterAuth({
  // ... Better Auth config
  plugins: [
    commet({
      client: commetClient,
      createCustomerOnSignUp: true,
      use: [
        portal({ returnUrl: "/dashboard" }),
        subscriptions(),
        features(),
        usage(),
        seats(),
      ],
    }),
  ],
});
```

### Configuring BetterAuth Client

You will use the Better Auth client to interact with the Commet functionalities.

```
import { createAuthClient } from "better-auth/react";
import { commetClient } from "@commet/better-auth/client";

export const authClient = createAuthClient({
  plugins: [commetClient()],
});
```

## Configuration Options

```
commet({
  client: commetClient,
  createCustomerOnSignUp: true,
  getCustomerCreateParams: ({ user }) => ({
    fullName: user.name,
    metadata: { source: "signup" },
  }),
  use: [
    // Commet sub-plugins
  ],
});
```

### Required Options

- `client`: Commet SDK client instance
- `use`: Array of Commet sub-plugins (at least one)

### Optional Options

- `createCustomerOnSignUp`: Automatically create a Commet customer when a user signs up
- `getCustomerCreateParams`: Custom function to provide additional customer creation parameters (`fullName`, `domain`, `metadata`)

### Customers

When `createCustomerOnSignUp` is enabled, a Commet customer is automatically created when a new user signs up. The customer is created with its `id` set to the Better Auth user ID, so you don't need any mapping between your users and Commet customers.

## Portal Plugin

Redirects customers to the Commet customer portal for self-service billing management.

```
import { commet, portal } from "@commet/better-auth";

commet({
  client: commetClient,
  use: [portal({ returnUrl: "/dashboard" })],
});
```

The portal plugin adds a `portal` method scoped under `authClient.customer`, which redirects the user to the Commet customer portal.

```
await authClient.customer.portal();
```

### Configuration

- `returnUrl`: URL to return to after the customer leaves the portal

## Subscriptions Plugin

Get and cancel the authenticated user's subscription.

```
import { commet, subscriptions } from "@commet/better-auth";

commet({
  client: commetClient,
  use: [subscriptions()],
});
```

```
// Get the current subscription
const { data: subscription } = await authClient.subscription.get();

// Cancel the subscription
await authClient.subscription.cancel({
  reason: "Too expensive",
  immediate: false, // Cancel at the end of the billing period
});
```

The `cancel` method accepts an optional `reason` and an `immediate` flag. By default, cancellation takes effect at the end of the current billing period.

## Features Plugin

Check feature access for the authenticated user. Supports boolean, metered and seat-based features.

```
import { commet, features } from "@commet/better-auth";

commet({
  client: commetClient,
  use: [features()],
});
```

```
// List all features
const { data: features } = await authClient.features.list();

// Get a specific feature
const { data: feature } = await authClient.features.get("api_calls");

// Check if a boolean feature is enabled
const { data: check } = await authClient.features.check("sso");
// { allowed: boolean }

// Check if the user can use one more unit of a metered feature
const { data: canUse } = await authClient.features.canUse("api_calls");
// { allowed: boolean, willBeCharged: boolean }
```

## Usage Plugin

Track usage events for metered billing.

```
import { commet, usage } from "@commet/better-auth";

commet({
  client: commetClient,
  use: [usage()],
});
```

```
await authClient.usage.track({
  feature: "api_calls",
  value: 1,
  idempotencyKey: "evt_123",
  properties: { endpoint: "/api/generate" },
});
```

The authenticated user is automatically associated with the event. The `feature` field maps to a feature code in your Commet plan.

## Seats Plugin

Manage seat-based licenses for the authenticated user.

```
import { commet, seats } from "@commet/better-auth";

commet({
  client: commetClient,
  use: [seats()],
});
```

```
// List all seat balances
const { data: seatBalances } = await authClient.seats.list();

// Add seats
await authClient.seats.add({ featureCode: "member", count: 5 });

// Remove seats
await authClient.seats.remove({ featureCode: "member", count: 2 });

// Set an exact count
await authClient.seats.set({ featureCode: "admin", count: 3 });

// Set multiple seat types at once
await authClient.seats.setAll({ admin: 2, member: 10, viewer: 50 });
```

## Webhooks Plugin

Handle Commet webhooks with signature verification. Webhooks are optional — you can always query the current state through the other sub-plugins.

```
import { commet, webhooks } from "@commet/better-auth";

commet({
  client: commetClient,
  use: [
    webhooks({
      secret: process.env.COMMET_WEBHOOK_SECRET!,
      onSubscriptionActivated: (payload) => {},
      onSubscriptionCanceled: (payload) => {},
      onPaymentReceived: (payload) => {},
      onPayload: (payload) => {}, // Catch-all
    }),
  ],
});
```

Configure a webhook endpoint in your Commet dashboard pointing to `/api/auth/commet/webhooks`, and add the signing secret to your environment.

```
COMMET_WEBHOOK_SECRET=whsec_...
```

The plugin supports handlers for all Commet webhook events:

- `onPayload` — Catch-all handler for any incoming event
- `onSubscriptionCreated` — Triggered when a subscription is created
- `onSubscriptionActivated` — Triggered when a subscription becomes active
- `onSubscriptionCanceled` — Triggered when a subscription is canceled
- `onSubscriptionUpdated` — Triggered when a subscription is updated
- `onSubscriptionPlanChanged` — Triggered when a subscription changes plan
- `onPaymentReceived` — Triggered when a payment is received
- `onPaymentFailed` — Triggered when a payment fails
- `onInvoiceCreated` — Triggered when an invoice is created
