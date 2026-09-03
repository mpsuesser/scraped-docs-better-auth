---
url: https://better-auth.com/llms.txt/docs/integrations/expo
title: "Expo"
description: ""
access_date: 2026-09-03T06:16:49.065Z
current_date: 2026-09-03T06:16:49.065Z
---

Integrate Better Auth with Expo.

Expo is a popular framework for building cross-platform apps with React Native. Better Auth supports both Expo native and web apps.

## Installation

## Configure A Better Auth Backend

Before using Better Auth with Expo, make sure you have a Better Auth backend set up. You can either use a separate server or leverage Expo's new [API Routes](https://docs.expo.dev/router/reference/api-routes) feature to host your Better Auth instance.

To get started, check out our [installation](https://better-auth.com/docs/installation) guide for setting up Better Auth on your server. If you prefer, check out the [full example](https://github.com/better-auth/examples/tree/main/expo-example).

To use the new API routes feature in Expo to host your Better Auth instance you can create a new API route in your Expo app and mount the Better Auth handler.

```
import { auth } from "@/lib/auth"; // import Better Auth handler

const handler = auth.handler;
export { handler as GET, handler as POST }; // export handler for both GET and POST requests
```

## Install Server Dependencies

Install both the Better Auth package and Expo plugin into your server application.

#### npm

```
npm install better-auth @better-auth/expo
```

#### pnpm

#### yarn

#### bun

## Install Client Dependencies

- You also need to install both the Better Auth package and Expo plugin into your Expo application.

#### npm

```
npm install better-auth @better-auth/expo
```

#### pnpm

#### yarn

#### bun

- And you need to install `expo-network` for network state detection.

#### npm

```
npm install expo-network
```

#### pnpm

#### yarn

#### bun

- (Optional) If you're using the default Expo template, these dependencies are already included, so you can skip this step. Otherwise, if you plan to use our social providers (e.g. Google, Apple), your Expo app requires a few additional dependencies.

#### npm

```
npm install expo-linking expo-web-browser expo-constants
```

#### pnpm

#### yarn

#### bun

## Add the Expo Plugin on Your Server

Add the Expo plugin to your Better Auth server.

```
import { betterAuth } from "better-auth";
import { expo } from "@better-auth/expo";

export const auth = betterAuth({
    plugins: [expo()],
    emailAndPassword: { 
        enabled: true, // Enable authentication using email and password.
      }, 
});
```

## Initialize Better Auth Client

To initialize Better Auth in your Expo app, you need to call `createAuthClient` with the base URL of your Better Auth backend. Make sure to import the client from `/react`.

Make sure you install the `expo-secure-store` package into your Expo app. This is used to store the session data and cookies securely.

#### npm

```
npm install expo-secure-store
```

#### pnpm

#### yarn

#### bun

You need to also import client plugin from `@better-auth/expo/client` and pass it to the `plugins` array when initializing the auth client.

This is important because:

- **Social Authentication Support:** enables social auth flows by handling authorization URLs and callbacks within the Expo web browser.
- **Secure Cookie Management:** stores cookies securely and automatically adds them to the headers of your auth requests.

```
import { createAuthClient } from "better-auth/react";
import { expoClient } from "@better-auth/expo/client";
import * as SecureStore from "expo-secure-store";

export const authClient = createAuthClient({
    baseURL: "http://localhost:8081", // Base URL of your Better Auth backend.
    plugins: [
        expoClient({
            scheme: "myapp",
            storagePrefix: "myapp",
            storage: SecureStore,
        })
    ]
});
```

## Scheme and Trusted Origins

Better Auth uses deep links to redirect users back to your app after authentication. To enable this, you need to add your app's scheme to the `trustedOrigins` list in your Better Auth config.

First, make sure you have a scheme defined in your `app.json` file.

```
{
    "expo": {
        "scheme": "myapp"
    }
}
```

Then, update your Better Auth config to include the scheme in the `trustedOrigins` list.

```
export const auth = betterAuth({
    trustedOrigins: ["myapp://"]
})
```

If you have multiple schemes or need to support deep linking with various paths, you can use specific patterns or wildcards:

```
export const auth = betterAuth({
    trustedOrigins: [
        // Basic scheme
        "myapp://", 
        
        // Production & staging schemes
        "myapp-prod://",
        "myapp-staging://",
        
        // Wildcard support for all paths following the scheme
        "myapp://*"
    ]
})
```

### Development Mode

During development, Expo uses the `exp://` scheme with your device's local IP address. To support this, you can use wildcards to match common local IP ranges:

```
export const auth = betterAuth({
    trustedOrigins: [
        "myapp://",
        
        // Development mode - Expo's exp:// scheme with local IP ranges
        ...(process.env.NODE_ENV === "development" ? [
            "exp://",                      // Trust any host of the exp:// scheme
            "exp://**",                    // Trust all Expo URLs (wildcard matching)
            "exp://192.168.*.*:*/**",      // Trust 192.168.x.x IP range with any port and path
        ] : [])
    ]
})
```

For more information about trusted origins, see the [trusted origins documentation](https://better-auth.com/docs/reference/options#trustedorigins).

## Configure Metro Bundler

Better Auth relies on `package.json` exports to resolve its modules. Starting with **Expo SDK 53+** (including SDK 55), package exports support is enabled by default in Metro, so **no extra Metro configuration is needed**.

If you have a custom `metro.config.js`, make sure you're not disabling package exports:

```
const { getDefaultConfig } = require("expo/metro-config");

const config = getDefaultConfig(__dirname);

// Do NOT set this to false — Better Auth requires package exports
// config.resolver.unstable_enablePackageExports = false;

module.exports = config;
```

Don't forget to clear the cache after making changes to your Metro config.

```
npx expo start --clear
```

## Usage

### Authenticating Users

With Better Auth initialized, you can now use the `authClient` to authenticate users in your Expo app.

#### sign-in

```
import { useState } from "react"; 
import { View, TextInput, Button } from "react-native";
import { authClient } from "@/lib/auth-client";

export default function SignIn() {
    const [email, setEmail] = useState("");
    const [password, setPassword] = useState("");

    const handleLogin = async () => {
        await authClient.signIn.email({
            email,
            password,
        })
    };

    return (
        <View>
            <TextInput
                placeholder="Email"
                value={email}
                onChangeText={setEmail}
            />
            <TextInput
                placeholder="Password"
                value={password}
                onChangeText={setPassword}
            />
            <Button title="Login" onPress={handleLogin} />
        </View>
    );
}
```

#### sign-up

#### Social Sign-In

For social sign-in, you can use the `authClient.signIn.social` method with the provider name and a callback URL. When you pass a relative path like "/dashboard", the Expo plugin automatically converts it to a deep link using `Linking.createURL`.

```
import { Button } from "react-native";
import { router } from "expo-router";
import { authClient } from "@/lib/auth-client";

export default function SocialSignIn() {
    const handleLogin = async () => {
        const { error } = await authClient.signIn.social({
            provider: "google",
            callbackURL: "/dashboard"
        })
        if (error) {
            // handle error
            return;
        }
        router.replace("/dashboard"); 
    };
    return <Button title="Login with Google" onPress={handleLogin} />;
}
```

#### IdToken Sign-In

If you want to make provider request on the mobile device and then verify the ID token on the server, you can use the `authClient.signIn.social` method with the `idToken` option.

```
import { Button } from "react-native";

export default function SocialSignIn() {
    const handleLogin = async () => {
        await authClient.signIn.social({
            provider: "google", // only google, apple and facebook are supported for idToken signIn
            idToken: {
                token: "...", // ID token from provider
                nonce: "...", // nonce from provider (optional)
            }
            callbackURL: "/dashboard" // this will be converted to a deep link (eg. \`myapp://dashboard\`) on native
        })
    };
    return <Button title="Login with Google" onPress={handleLogin} />;
}
```

### Session

Better Auth provides a `useSession` hook to access the current user's session in your app.

```
import { Text } from "react-native";
import { authClient } from "@/lib/auth-client";

export default function Index() {
    const { data: session } = authClient.useSession();

    return <Text>Welcome, {session?.user.name}</Text>;
}
```

On native, the session data will be cached in SecureStore. This will allow you to remove the need for a loading spinner when the app is reloaded. You can disable this behavior by passing the `disableCache` option to the client.

### Making Authenticated Requests to Your Server

To make authenticated requests to your server that require the user's session, you have to retrieve the session cookie from `SecureStore` and manually add it to your request headers.

```
import { authClient } from "@/lib/auth-client";

const makeAuthenticatedRequest = async () => {
  const cookies = await authClient.getCookie(); 
  const headers = {
    "Cookie": cookies, 
  };
  const response = await fetch("http://localhost:8081/api/secure-endpoint", { 
    headers,
    // 'include' can interfere with the cookies we just set manually in the headers
    credentials: "omit"
  });
  const data = await response.json();
  return data;
};
```

**Example: Usage With TRPC**

```
//...other imports
import { authClient } from "@/lib/auth-client"; 

export const api = createTRPCReact<AppRouter>();

export function TRPCProvider(props: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());
  const [trpcClient] = useState(() =>
    api.createClient({
      links: [
        httpBatchLink({
          //...your other options
          async headers() {
            const headers = new Map<string, string>(); 
            const cookies = await authClient.getCookie(); 
            if (cookies) { 
              headers.set("Cookie", cookies); 
            } 
            return Object.fromEntries(headers); 
          },
        }),
      ],
    }),
  );

  return (
    <api.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {props.children}
      </QueryClientProvider>
    </api.Provider>
  );
}
```

## Options

### Expo Client

**storage**: the SecureStore-compatible storage used to cache session data and cookies. Write coordination is scoped to the provided object, so reuse it when configuring multiple clients that access the same stored data.

```
import { createAuthClient } from "better-auth/react";
import { expoClient } from "@better-auth/expo/client";
import * as SecureStore from "expo-secure-store";

const authClient = createAuthClient({
    baseURL: "http://localhost:8081",
    plugins: [
        expoClient({
            storage: SecureStore,
            // ...
        })
    ],
});
```

**scheme**: scheme is used to deep link back to your app after a user has authenticated using oAuth providers. By default, Better Auth tries to read the scheme from the `app.json` file. If you need to override this, you can pass the scheme option to the client.

```
import { createAuthClient } from "better-auth/react";
import { expoClient } from "@better-auth/expo/client";

const authClient = createAuthClient({
    baseURL: "http://localhost:8081",
    plugins: [
        expoClient({
            scheme: "myapp",
            // ...
        }),
    ],
});
```

**disableCache**: By default, the client will cache the session data in SecureStore. You can disable this behavior by passing the `disableCache` option to the client.

```
import { createAuthClient } from "better-auth/react";
import { expoClient } from "@better-auth/expo/client";

const authClient = createAuthClient({
    baseURL: "http://localhost:8081",
    plugins: [
        expoClient({
            disableCache: true,
            // ...
        }),
    ],
});
```

**cookiePrefix**: Prefix(es) for server cookie names to identify which cookies belong to better-auth. This prevents infinite refetching when third-party cookies are set. Can be a single string or an array of strings to match multiple prefixes. Defaults to `"better-auth"`.

```
import { createAuthClient } from "better-auth/react";
import { expoClient } from "@better-auth/expo/client";
import * as SecureStore from "expo-secure-store";

const authClient = createAuthClient({
    baseURL: "http://localhost:8081",
    plugins: [
        expoClient({
            storage: SecureStore,
            // Single prefix
            cookiePrefix: "better-auth"
        })
    ]
});
```

You can also provide multiple prefixes to match cookies from different authentication systems:

```
const authClient = createAuthClient({
    baseURL: "http://localhost:8081",
    plugins: [
        expoClient({
            storage: SecureStore,
            // Multiple prefixes
            cookiePrefix: ["better-auth", "my-app", "custom-auth"]
        })
    ]
});
```

### Expo Servers

Server plugin options:

**disableOriginOverride**: Override the origin for Expo API routes (default: false). Enable this if you're facing cors origin issues with Expo API routes.
