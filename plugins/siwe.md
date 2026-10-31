---
url: https://better-auth.com/llms.txt/docs/plugins/siwe
title: "Siwe"
description: ""
access_date: 2026-10-31T22:51:02.801Z
current_date: 2026-10-31T22:51:02.801Z
---

The Sign in with Ethereum (SIWE) plugin allows users to authenticate using their Ethereum wallets following the [ERC-4361 standard](https://eips.ethereum.org/EIPS/eip-4361). This plugin provides flexibility by allowing you to implement your own message verification and nonce generation logic.

## Installation

### Add the Server Plugin

Add the SIWE plugin to your auth configuration:

```
import { betterAuth } from "better-auth";
import { siwe } from "better-auth/plugins"; 

export const auth = betterAuth({
    plugins: [
        siwe({
            domain: "example.com",
            emailDomainName: "example.com", // optional
            anonymous: false, // optional, default is true
            getNonce: async () => {
                // Return an ERC-4361 nonce: 8-250 alphanumeric characters
                return "A1b2C3d4E5f6G7h8J";
            },
            verifyMessage: async (args) => {
                // Implement your SIWE message verification logic here
                // This should verify the signature against the message
                return true; // return true if signature is valid
            },
            ensLookup: async (args) => {
                // Optional: Implement ENS lookup for user names and avatars
                return {
                    name: "user.eth",
                    avatar: "https://example.com/avatar.png"
                };
            },
        }),
    ],
});
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

### Add the Client Plugin

```
import { createAuthClient } from "better-auth/client";
import { siweClient } from "better-auth/client/plugins"; 

export const authClient = createAuthClient({
plugins: [
    siweClient() 
]
});
```

## Usage

### Generate a Nonce

Before asking the wallet to sign, issue a nonce for the sign-in attempt. The nonce is not bound to a wallet address or Chain ID because one-step wallet flows may not know either value until the wallet signs the SIWE message.

```
import { authClient } from "@/lib/auth-client";

const { data, error } = await authClient.siwe.nonce();

if (data) {
  console.log("Nonce:", data.nonce);
}
```

### Sign In with Ethereum

After generating a nonce and creating a SIWE message, verify the signature to authenticate:

```
import { authClient } from "@/lib/auth-client";

const { data, error } = await authClient.siwe.verify({
  message: "Your ERC-4361 SIWE message string",
  signature: "0x...", // The signature from the user's wallet
  email: "user@example.com", // optional, required if anonymous is false
});

if (data) {
  console.log("Authentication successful:", data.user);
}
```

### Chain-Specific Messages

Chain selection belongs in the ERC-4361 message, not in the verification request body. Generate a nonce, build the SIWE message with the wallet address and target Chain ID, ask the wallet to sign that exact message, and then verify the signed message:

```
import { authClient } from "@/lib/auth-client";

const nonce = await authClient.siwe.nonce();

const { data, error } = await authClient.siwe.verify({
  // The signed ERC-4361 message contains the wallet address,
  // Chain ID: 137, and Nonce: nonce.data?.nonce.
  message,
  signature,
});
```

## Configuration Options

### Server Options

The SIWE plugin accepts the following configuration options:

- **domain**: The domain name of your application (required for SIWE message generation)
- **emailDomainName**: A custom email domain for wallet-derived addresses. If omitted, the plugin uses `{walletAddress}@siwe.placeholder.invalid`
- **anonymous**: Whether to allow anonymous sign-ins without requiring an email. Default is `true`
- **getNonce**: Function to generate a globally unique nonce for each sign-in attempt. You must implement this function to return a cryptographically secure ERC-4361 nonce: 8-250 alphanumeric characters. Must return a `Promise<string>`
- **verifyMessage**: Function to verify the signature over the SIWE message. It only needs to perform signature recovery for the supplied address (e.g. viem's `verifyMessage`) and return `Promise<boolean>` — the plugin independently validates the message's nonce, domain, address, Chain ID, and time bounds before creating a session
- **ensLookup**: Optional function to lookup ENS names and avatars for Ethereum addresses

### Client Options

The SIWE client plugin doesn't require any configuration options:

```
import { createAuthClient } from "better-auth/client";
import { siweClient } from "better-auth/client/plugins";

export const authClient = createAuthClient({
  plugins: [
    siweClient(),
  ],
});
```

## Schema

The SIWE plugin adds a `walletAddress` table to store user wallet associations:

| Field | Type | Description |
| --- | --- | --- |
| id | string | Primary key |
| userId | string | Reference to user.id |
| address | string | Ethereum wallet address |
| chainId | number | Chain ID (e.g., 1 for Ethereum mainnet) |
| isPrimary | boolean | Whether this is the user's primary wallet |
| createdAt | date | Creation timestamp |

## Example Implementation

Here's a complete example showing how to implement SIWE authentication:

```
import { betterAuth } from "better-auth";
import { siwe } from "better-auth/plugins";
import { generateRandomString } from "better-auth/crypto";
import { verifyMessage, createPublicClient, http } from "viem";
import { mainnet } from "viem/chains";

export const auth = betterAuth({
  database: {
    // your database configuration
  },
  plugins: [
    siwe({
      domain: "myapp.com",
      emailDomainName: "myapp.com",
      anonymous: false,
      getNonce: async () => {
        // Generate a cryptographically secure random nonce
        return generateRandomString(32, "a-z", "A-Z", "0-9");
      },
      verifyMessage: async ({ message, signature, address }) => {
        try {
          // Verify the signature using viem (recommended)
          const isValid = await verifyMessage({
            address: address as \`0x${string}\`,
            message,
            signature: signature as \`0x${string}\`,
          });
          return isValid;
        } catch (error) {
          console.error("SIWE verification failed:", error);
          return false;
        }
      },
      ensLookup: async ({ walletAddress }) => {
        try {
          // Optional: lookup ENS name and avatar using viem
          // You can use viem's ENS utilities here
          const client = createPublicClient({
            chain: mainnet,
            transport: http(),
          });

          const ensName = await client.getEnsName({
            address: walletAddress as \`0x${string}\`,
          });

          const ensAvatar = ensName
            ? await client.getEnsAvatar({
                name: ensName,
              })
            : null;

          return {
            name: ensName || walletAddress,
            avatar: ensAvatar || "",
          };
        } catch {
          return {
            name: walletAddress,
            avatar: "",
          };
        }
      },
    }),
  ],
});
```
