---
url: https://better-auth.com/llms.txt/docs/guides/clerk-migration-guide
title: "Clerk Migration Guide"
description: ""
access_date: 2026-11-11T11:29:38.576Z
current_date: 2026-11-11T11:29:38.576Z
---

A step-by-step guide to transitioning from Clerk to Better Auth.

In this guide, we'll walk through the steps to migrate a project from Clerk to Better Auth — including email/password with proper hashing, social/external accounts, phone number, two-factor data, and more.

## Before You Begin

Before starting the migration process, set up Better Auth in your project. Follow the [installation guide](https://better-auth.com/docs/installation) to get started. And go to

### Connect to your database

You'll need to connect to your database to migrate the users and accounts. You can use any database you want, but for this example, we'll use PostgreSQL.

#### npm

```
npm install pg
```

#### pnpm

#### yarn

#### bun

And then you can use the following code to connect to your database.

```
import { Pool } from "pg";

export const auth = betterAuth({
    database: new Pool({ 
        connectionString: process.env.DATABASE_URL 
    }),
})
```

### Enable Email and Password (Optional)

Enable the email and password in your auth config and implement your own logic for sending verification emails, reset password emails, etc.

First, install bcrypt:

#### npm

```
npm install pnpm add bcrypt
pnpm add -D @types/bcrypt
```

#### pnpm

#### yarn

#### bun

Then configure Better Auth to use bcrypt:

```
import { betterAuth } from "better-auth";
import bcrypt from "bcrypt";

export const auth = betterAuth({
    emailAndPassword: { 
        enabled: true,
        password: { 
            hash: async (password) => { 
                return await bcrypt.hash(password, 10); 
            }, 
            verify: async ({ hash, password }) => { 
                return await bcrypt.compare(password, hash); 
            }, 
        }, 
    }, 
})
```

See [Email and Password](https://better-auth.com/docs/authentication/email-password) for more configuration options.

### Setup Social Providers (Optional)

Add social providers you have enabled in your Clerk project in your auth config.

```
import { betterAuth } from "better-auth";

export const auth = betterAuth({
    database: new Pool({ 
        connectionString: process.env.DATABASE_URL 
    }),
    emailAndPassword: { 
        enabled: true,
    },
    socialProviders: { 
        github: { 
            clientId: process.env.GITHUB_CLIENT_ID, 
            clientSecret: process.env.GITHUB_CLIENT_SECRET, 
        } 
    } 
})
```

### Add Plugins (Optional)

You can add the following plugins to your auth config based on your needs.

[Admin](https://better-auth.com/docs/plugins/admin) Plugin will allow you to manage users, user impersonations and app level roles and permissions.

[Two Factor](https://better-auth.com/docs/plugins/2fa) Plugin will allow you to add two-factor authentication to your application.

[Phone Number](https://better-auth.com/docs/plugins/phone-number) Plugin will allow you to add phone number authentication to your application.

[Username](https://better-auth.com/docs/plugins/username) Plugin will allow you to add username authentication to your application.

```
import { Pool } from "pg";
import { betterAuth } from "better-auth";
import { admin, twoFactor, phoneNumber, username } from "better-auth/plugins";
import bcrypt from "bcrypt";

export const auth = betterAuth({
    database: new Pool({ 
        connectionString: process.env.DATABASE_URL 
    }),
    emailAndPassword: { 
        enabled: true,
        password: {
            hash: async (password) => {
                return await bcrypt.hash(password, 10);
            },
            verify: async ({ hash, password }) => {
                return await bcrypt.compare(password, hash);
            },
        },
    },
    socialProviders: {
        github: {
            clientId: process.env.GITHUB_CLIENT_ID!,
            clientSecret: process.env.GITHUB_CLIENT_SECRET!,
        }
    },
    plugins: [admin(), twoFactor(), phoneNumber(), username()], 
})
```

### Generate Schema

If you're using a custom database adapter, generate the schema:

```
npx auth generate
```

or if you're using the default adapter, you can use the following command:

```
npx auth migrate
```

### Export Clerk Users

Go to the Clerk dashboard and [export the users](https://clerk.com/docs/deployments/exporting-users#export-your-users-data-from-the-clerk-dashboard). It will download a CSV file with the users data. You need to save it as `exported_users.csv` and put it in the root of your project.

The export includes each user's TOTP secret but not their backup codes, so the migration script generates new backup codes.

### Create the migration script

Install a CSV parser to read the export:

#### npm

```
npm install csv-parse
```

#### pnpm

#### yarn

#### bun

Create a new file called `migrate-clerk.ts` in the `scripts` folder and add the following code:

```
import { parse } from "csv-parse/sync";
import { generateRandomString, symmetricEncrypt } from "better-auth/crypto";
import type { SecretConfig } from "better-auth/crypto";

import { auth } from "@/lib/auth"; // import your auth instance

function getCSVData(csv: string) {
  return parse(csv, {
      columns: true,
      skip_empty_lines: true,
  }) as Array<{
      id: string;
      first_name: string;
      last_name: string;
      username: string;
      primary_email_address: string;
      primary_phone_number: string;
      verified_email_addresses: string;
      unverified_email_addresses: string;
      verified_phone_numbers: string;
      unverified_phone_numbers: string;
      totp_secret: string;
      password_digest: string;
      password_hasher: string;
  }>;
}

const exportedUserCSV = await Bun.file("exported_users.csv").text(); // this is the file you downloaded from Clerk

async function getClerkUsers(totalUsers: number) {
  const clerkUsers: {
      id: string;
      first_name: string;
      last_name: string;
      username: string;
      image_url: string;
      password_enabled: boolean;
      two_factor_enabled: boolean;
      totp_enabled: boolean;
      backup_code_enabled: boolean;
      banned: boolean;
      locked: boolean;
      lockout_expires_in_seconds: number;
      created_at: number;
      updated_at: number;
      external_accounts: {
          id: string;
          provider: string;
          identification_id: string;
          provider_user_id: string;
          approved_scopes: string;
          email_address: string;
          first_name: string;
          last_name: string;
          image_url: string;
          created_at: number;
          updated_at: number;
      }[]
  }[] = [];
  for (let i = 0; i < totalUsers; i += 500) {
      const response = await fetch(\`https://api.clerk.com/v1/users?offset=${i}&limit=${500}\`, {
          headers: {
              'Authorization': \`Bearer ${process.env.CLERK_SECRET_KEY}\`
          }
      });
      if (!response.ok) {
          throw new Error(\`Failed to fetch users: ${response.statusText}\`);
      }
      const clerkUsersData = await response.json();
      // biome-ignore lint/suspicious/noExplicitAny: <explanation>
      clerkUsers.push(...clerkUsersData as any);
  }
  return clerkUsers;
}

export async function generateBackupCodes(
  key: string | SecretConfig,
) {
  const backupCodes = Array.from({ length: 10 })
      .fill(null)
      .map(() => generateRandomString(10, "a-z", "0-9", "A-Z"))
      .map((code) => \`${code.slice(0, 5)}-${code.slice(5)}\`);
  const encCodes = await symmetricEncrypt({
      data: JSON.stringify(backupCodes),
      key: key,
  });
  return encCodes
}

// Helper function to safely convert timestamp to Date
function safeDateConversion(timestamp?: number): Date {
  if (!timestamp) return new Date();

  const date = new Date(timestamp);

  // Check if the date is valid
  if (isNaN(date.getTime())) {
      console.warn(\`Invalid timestamp: ${timestamp}, falling back to current date\`);
      return new Date();
  }

  // Check for unreasonable dates (before 2000 or after 2100)
  const year = date.getFullYear();
  if (year < 2000 || year > 2100) {
      console.warn(\`Suspicious date year: ${year}, falling back to current date\`);
      return new Date();
  }

  return date;
}

async function migrateFromClerk() {
  const jsonData = getCSVData(exportedUserCSV);
  const clerkUsers = await getClerkUsers(jsonData.length);
  const ctx = await auth.$context
  const isAdminEnabled = ctx.options?.plugins?.find(plugin => plugin.id === "admin");
  const isTwoFactorEnabled = ctx.options?.plugins?.find(plugin => plugin.id === "two-factor");
  const isUsernameEnabled = ctx.options?.plugins?.find(plugin => plugin.id === "username");
  const isPhoneNumberEnabled = ctx.options?.plugins?.find(plugin => plugin.id === "phone-number");
  for (const user of jsonData) {
      const { id, first_name, last_name, username, primary_email_address, primary_phone_number, verified_email_addresses, unverified_email_addresses, verified_phone_numbers, unverified_phone_numbers, totp_secret, password_digest, password_hasher } = user;
      const clerkUser = clerkUsers.find(clerkUser => clerkUser?.id === id);

      // create user
      const createdUser = await ctx.adapter.create<{
          id: string;
      }>({
          model: "user",
          data: {
              id,
              email: primary_email_address,
              emailVerified: verified_email_addresses.length > 0,
              name: \`${first_name} ${last_name}\`,
              image: clerkUser?.image_url,
              createdAt: safeDateConversion(clerkUser?.created_at),
              updatedAt: safeDateConversion(clerkUser?.updated_at),
              // # Two Factor (if you enabled two factor plugin)
              ...(isTwoFactorEnabled ? {
                  twoFactorEnabled: Boolean(totp_secret)
              } : {}),
              // # Admin (if you enabled admin plugin)
              ...(isAdminEnabled ? {
                  banned: clerkUser?.banned,
                  banExpires: clerkUser?.lockout_expires_in_seconds
                     ? new Date(Date.now() + clerkUser.lockout_expires_in_seconds * 1000)
                     : undefined,
                  role: "user"
              } : {}),
              // # Username (if you enabled username plugin)
              ...(isUsernameEnabled ? {
                  // Empty CSV cells parse as empty strings
                  username: username || undefined,
              } : {}),
              // # Phone Number (if you enabled phone number plugin)  
              ...(isPhoneNumberEnabled ? {
                  // Empty CSV cells parse as empty strings
                  phoneNumber: primary_phone_number || undefined,
                  phoneNumberVerified: verified_phone_numbers.length > 0,
              } : {}),
          },
          forceAllowId: true
      }).catch(async e => {
          return await ctx.adapter.findOne<{
              id: string;
          }>({
              model: "user",
              where: [{
                  field: "id",
                  value: id
              }]
          })
      })
      if (!createdUser) {
          throw new Error(\`Unable to create or find user ${id}\`);
      }
      // create credential account
      // Clerk's API lists only OAuth accounts, so passwords come from the export
      if (password_digest) {
          await ctx.adapter.create({
              model: "account",
              data: {
                  providerId: "credential",
                  accountId: createdUser.id,
                  userId: createdUser.id,
                  password: password_digest,
                  createdAt: safeDateConversion(clerkUser?.created_at),
                  updatedAt: safeDateConversion(clerkUser?.updated_at),
              }
          })
      }
      // create external account
      const externalAccounts = clerkUser?.external_accounts;
      if (externalAccounts) {
          for (const externalAccount of externalAccounts) {
              const { id, provider, identification_id, provider_user_id, approved_scopes, email_address, first_name, last_name, image_url, created_at, updated_at } = externalAccount;
              const providerId = provider.replace("oauth_", "");
              await ctx.adapter.create({
                  model: "account",
                  data: {
                      id,
                      providerId,
                      accountId: externalAccount.provider_user_id,
                      scope: approved_scopes,
                      userId: createdUser.id,
                      createdAt: safeDateConversion(created_at),
                      updatedAt: safeDateConversion(updated_at),
                  },
                  forceAllowId: true
              })
          }
      }

      //two factor
      if (isTwoFactorEnabled && totp_secret) {
          await ctx.adapter.create({
              model: "twoFactor",
              data: {
                  userId: createdUser.id,
                  // Clerk exports Base32 secrets, so mark them for decoding
                  secret: await symmetricEncrypt({
                      key: ctx.secretConfig,
                      data: \`base32:${totp_secret}\`,
                  }),
                  backupCodes: await generateBackupCodes(ctx.secretConfig)
              }
          })
      }
  }
}

migrateFromClerk()
  .then(() => {
      console.log('Migration completed');
      process.exit(0);
  })
  .catch((error) => {
      console.error('Migration failed:', error);
      process.exit(1);
  });
```

Make sure to replace the `process.env.CLERK_SECRET_KEY` with your own Clerk secret key. Feel free to customize the script to your needs.

### Run the migration

Run the migration:

```
bun run scripts/migrate-clerk.ts # you can use any thing you like to run the script
```

### Verify the migration

After running the migration, verify that all users have been properly migrated by checking the database.

### Update your components

Now that the data is migrated, you can start updating your components to use Better Auth. Here's an example for the sign-in component:

```
import { authClient } from "@/lib/auth-client";

export const SignIn = () => {
  const handleSignIn = async () => {
    const { data, error } = await authClient.signIn.email({
      email: "user@example.com",
      password: "password",
    });
    
    if (error) {
      console.error(error);
      return;
    }
    // Handle successful sign in
  };

  return (
    <form onSubmit={handleSignIn}>
      <button type="submit">Sign in</button>
    </form>
  );
};
```

### Update the middleware

Replace your Clerk middleware with Better Auth's middleware:

```
import { NextRequest, NextResponse } from "next/server";
import { getSessionCookie } from "better-auth/cookies";
export async function middleware(request: NextRequest) {
  const sessionCookie = getSessionCookie(request);
  const { pathname } = request.nextUrl;
  if (sessionCookie && ["/login", "/signup"].includes(pathname)) {
    return NextResponse.redirect(new URL("/dashboard", request.url));
  }
  if (!sessionCookie && pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard", "/login", "/signup"],
};
```

### Remove Clerk Dependencies

Once you've verified that everything is working correctly with Better Auth, you can remove Clerk:

```
pnpm remove @clerk/nextjs @clerk/themes @clerk/types
```

## Additional Resources

[Goodbye Clerk, Hello Better Auth – Full Migration Guide!](https://www.youtube.com/watch?v=Za_QihbDSuk)

## Wrapping Up

Congratulations! You've successfully migrated from Clerk to Better Auth.

Better Auth offers greater flexibility and more features—be sure to explore the [documentation](https://better-auth.com/docs) to unlock its full potential.
