---
url: https://better-auth.com/llms.txt/docs/plugins/autumn
title: "Autumn"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Better Auth Plugin for Autumn Billing

[Autumn](https://useautumn.com/) is open source infrastructure to run SaaS pricing plans. It sits between your app and Stripe, and acts as the database for your customers' subscription status, usage metering and feature permissions.

### [Get help on Autumn's Discord](https://discord.gg/STqxY92zuS)

## Features

- One function for all checkout, subscription and payment flows
- No webhooks required: query Autumn for the data you need
- Manages your application's free and paid plans
- Usage tracking for usage billing and periodic limits
- Custom plans and pricing changes through Autumn's dashboard

### Setup Autumn Account

First, create your pricing plans in Autumn's [dashboard](https://app.useautumn.com/), where you define what each plan and product gets access to and how it should be billed. In this example, we're handling the free and pro plans for an AI chatbot, which comes with a number of `messages` per month.

### Install Autumn SDK

#### npm

```
npm install autumn-js
```

#### pnpm

#### yarn

#### bun

### Add AUTUMN\_SECRET\_KEY to your environment variables

You can find it in Autumn's dashboard under " [Developer](https://app.useautumn.com/sandbox/onboarding) ".

```
AUTUMN_SECRET_KEY=am_sk_xxxxxxxxxx
```

### Add the Autumn plugin to your auth config

#### User

```
import { autumn } from "autumn-js/better-auth";

export const auth = betterAuth({
  // ...
  plugins: [autumn()],
});
```

#### Organization

#### User & Organization

#### Custom

### Add <AutumnProvider />

In your layout file, wrap your application with the AutumnProvider component, and pass in the `useBetterAuth` to true.

```
import { AutumnProvider } from "autumn-js/react";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <AutumnProvider useBetterAuth={true}>
          {children}
        </AutumnProvider>
      </body>
    </html>
  );
}
```

## Usage

### Handle payments

Call `attach` to redirect the customer to a Stripe checkout page when they want to purchase the Pro plan.

If their payment method is already on file, `AttachDialog` will open instead to let the customer confirm their new subscription or purchase, and handle the payment.

```
import { useCustomer, AttachDialog } from "autumn-js/react";

export default function PurchaseButton() {
  const { attach } = useCustomer();

  return (
    <button
      onClick={async () => {
        await attach({
          productId: "pro",
          dialog: AttachDialog,
        });
      }}
    >
      Upgrade to Pro
    </button>
  );
}
```

The AttachDialog component can be used directly from the `autumn-js/react` library (as shown in the example above), or downloaded as a [shadcn/ui component](https://docs.useautumn.com/quickstart/shadcn) to customize.

### Integrate Pricing Logic

Integrate your client and server pricing tiers logic with the following functions:

- `check` to see if the customer is `allowed` to send a message.
- `track` a usage event in Autumn (typically done server-side)
- `customer` to display any relevant billing data in your UI (subscriptions, feature balances)

Server-side, you can access Autumn's functions through the `auth` object.

#### Client

```
import { useCustomer } from "autumn-js/react";

export default function SendChatMessage() {
  const { customer, allowed, refetch } = useCustomer();

  return (
    <>
      <button
        onClick={async () => {
          if (allowed({ featureId: "messages" })) {
            //... send chatbot message server-side, then
            await refetch(); // refetch customer usage data
            alert(
              "Remaining messages: " + customer?.features.messages?.balance
            );
          } else {
            alert("You're out of messages");
          }
        }}
      >
        Send Message
      </button>
    </>
  );
}
```

#### Server

### Additional Functions

#### openBillingPortal()

Opens a billing portal where the customer can update their payment method or cancel their plan.

```
import { useCustomer } from "autumn-js/react";

export default function BillingSettings() {
  const { openBillingPortal } = useCustomer();

  return (
    <button
      onClick={async () => {
        await openBillingPortal({
          returnUrl: "/settings/billing",
        });
      }}
    >
      Manage Billing
    </button>
  );
}
```

#### cancel()

Cancel a product or subscription.

```
import { useCustomer } from "autumn-js/react";

export default function CancelSubscription() {
  const { cancel } = useCustomer();

  return (
    <button
      onClick={async () => {
        await cancel({ productId: "pro" });
      }}
    >
      Cancel Subscription
    </button>
  );
}
```

#### Get invoice history

Pass in an `expand` param into `useCustomer` to get additional information. You can expand `invoices`, `trials_used`, `payment_method`, or `rewards`.

```
import { useCustomer } from "autumn-js/react";

export default function CustomerProfile() {
  const { customer } = useCustomer({ expand: ["invoices"] });

  return (
    <div>
      <h2>Customer Profile</h2>
      <p>Name: {customer?.name}</p>
      <p>Email: {customer?.email}</p>
      <p>Balance: {customer?.features.chat_messages?.balance}</p>
    </div>
  );
}
```
