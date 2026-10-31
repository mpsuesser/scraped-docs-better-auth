---
url: https://better-auth.com/llms.txt/docs/integrations/electron
title: "Electron"
description: ""
access_date: 2026-10-31T21:45:27.385Z
current_date: 2026-10-31T21:45:27.385Z
---

Integrate Better Auth with Electron.

Electron is a popular framework for building cross-platform desktop applications using web technologies.

Better Auth can be integrated into Electron apps to provide secure authentication flows, leveraging the system browser.

## Installation

### Configure a Better Auth front- & back-end

Before integrating with Electron, ensure you have a Better Auth server and client set up.

To get started, check out our [installation](https://better-auth.com/docs/installation) guide for setting up Better Auth.

### Install the required packages

Install the Better Auth server package and the Electron integration package in your server, Electron app, and web client projects.

**In each project, run:**

#### npm

```
npm install better-auth @better-auth/electron
```

#### pnpm

#### yarn

#### bun

### Add the Electron plugin to your Better Auth server

Add the Electron plugin to your Better Auth server.

```
import { betterAuth } from "better-auth";
import { electron } from "@better-auth/electron";

export const auth = betterAuth({
  plugins: [electron()],
  emailAndPassword: {
    enabled: true,
  },
  social: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
  },
});
```

### Add the proxy plugin to your web client

On your frontend, add the proxy plugin to handle redirects back into the Electron app.

```
import { createAuthClient } from "better-auth/client";
import { electronProxyClient } from "@better-auth/electron/proxy";

export const authClient = createAuthClient({
  baseURL: "http://localhost:8081",
  plugins: [
    electronProxyClient({
      protocol: {
        scheme: "com.example.app"
      },
    }),
  ],
});
```

### Initialize the Electron client

```
import { createAuthClient } from "better-auth/client";
import { electronClient } from "@better-auth/electron/client";

export const authClient = createAuthClient({
  baseURL: "http://localhost:8081", // Base URL of your Better Auth frontend
  plugins: [
    electronClient({
      signInURL: "https://app.example.com/sign-in", // The URL to redirect to for authentication
      protocol: {
        scheme: "com.example.app" // The custom protocol scheme registered by your Electron app
      },
      storage: {
        getItem: async (key) => {
          // retrieve entry from storage
        },
        setItem: async (key, value) => {
          // set entry in storage
        },
      },
    }),
  ],
});
```

If you'd rather not implement your own storage solution, we offer a default option:

1. Make sure to install the `conf` package:

#### npm

```
npm install conf
```

#### pnpm

#### yarn

#### bun

2. Then, you can use it as follows:

```
import { storage } from "@better-auth/electron/storage";

electronClient({
  storage: storage(),
});
```

### Scheme and Trusted Origins

The Electron plugin uses deep links to redirect users back to your app after authentication. To enable this, you need to add your app's protocol scheme to the `trustedOrigins` on your **Better Auth server**.

First, make sure you have a custom protocol scheme registered in your build configuration.

#### electron-forge

```
module.exports = {
  packagerConfig: {
    protocols: [{
      name: "MyApp Protocol",
      schemes: ["com.example.app"],
    }],
  },
  // ...other config options
};
```

#### electron-builder

Then, update your Better Auth config to include the scheme in `trustedOrigins`:

```
export const auth = betterAuth({
  trustedOrigins: ["com.example.app:/"],
});
```

### Configure the BrowserWindow

Ensure that your `BrowserWindow` is configured with `nodeIntegration` set to `false` and `contextIsolation` set to `true`, to ensure NodeJS APIs aren't exposed to any JavaScript process running in the browser.

```
import { BrowserWindow } from "electron";
import { join } from "node:path";

const win = new BrowserWindow({
  webPreferences: {
    preload: join(__dirname, "preload.mjs"),
    nodeIntegration: false,
    contextIsolation: true,
  },
});
```

### Setup the Main Process

In your Electron main process, use the `setupMain()` method from the Electron auth client to handle necessary configurations.

This will:

- Register the protocol handler for deep links.
- Register the [user image proxy](#user-image-proxy) protocol.
- Set up content security policies.
- Set up IPC bridges for communication between main and renderer processes.

```
import { authClient } from "./lib/auth-client";

authClient.setupMain();
```

Note that this must be called before the app is ready.

### Setup the Renderer Process

In your preload script, use the `setupRenderer()` method from the Electron auth client to expose safe IPC bridges to the renderer process.

```
import { setupRenderer } from "@better-auth/electron/preload";

setupRenderer();
```

Note that this must also be called before the app is ready.

## Usage

### Handling Authorization in the Browser

In order to redirect users back to your Electron application, you need to call the `ensureElectronRedirect()` method on your web sign-in callback page. Also make sure to preserve any PKCE and state parameters during the sign-in initiation, by passing them via `fetchOptions.query`.

The following example uses React, but the same logic applies to any framework:

```
import { useEffect, use } from "react";
import { authClient } from "../auth-client";

function SignIn({
  searchParams,
}: {
  searchParams: Promise<{
    client_id?: string | undefined;
    state?: string | undefined;
    code_challenge?: string | undefined;
  }>;
}) {
  const query = use(searchParams);

  useEffect(() => {
    const id = authClient.ensureElectronRedirect();
    return () => {
      clearTimeout(id);
    }
  }, []);

  return (
    <button
      onClick={() =>
        authClient.signIn.social({
          provider: "google",
          fetchOptions: { query }, // preserve PKCE/state
        })
      }
    >
      Sign in with Google
    </button>
  );
}
```

To handle already signed-in users, you can use the `electron.transferUser` method on the client and `electronTransferUser` on the server to transfer the user to the Electron app. [See Example](https://github.com/better-auth/better-auth/blob/main/demo/nextjs/app/\(auth\)/sign-in/_components/electron.tsx)

### Handling Authentication in Electron

In your Electron renderer process, you can use the IPC bridges exposed by the preload script.

```
function Auth() {
  useEffect(() => {
    const unsubscribeAuthenticated =
      window.onAuthenticated((user) => {
        console.log("Authenticated user:", user);
      });
    const unsubscribeAuthError =
      window.onAuthError((ctx) => {
        toast.error(\`Authentication error: ${ctx.message}\`);
      });

    return () => {
      unsubscribeAuthenticated();
      unsubscribeAuthError();
    };
  }, []);

  return (
    <>
      <button onClick={() => window.requestAuth()}>
        Sign in with Browser
      </button>
      <button
        onClick={() =>
          window.requestAuth({
            provider: "google", // sign in with social provider, redirecting directly to the provider
          })
        }
      >
        Sign in with Google
      </button>
    </>
  );
}
```

In the main process, you can call from the auth client directly:

```
import { authClient } from "./lib/auth-client";

authClient.requestAuth();
```

### Sign Out

To sign out the user inside the renderer process, you can use the `signOut` bridge in the renderer process:

```
<button onClick={() => window.signOut()}>Sign out</button>
```

### Subscribing to User Updates

You can listen for user changes via the `onUserUpdated` bridge in the renderer process:

```
useEffect(() => {
  const unsubscribe = window.onUserUpdated((user) => {
    console.log("User updated:", user);
  });
  return () => unsubscribe();
}, []);
```

### Handling Errors

Listen for authentication errors via the `onAuthError` bridge in the renderer process. The bridge will receive error context forwarded from fetch hooks.

```
useEffect(() => {
  const unsubscribe = window.onAuthError((ctx) => {
    console.error("Authentication error:", ctx);
  });
  return () => unsubscribe();
}, []);
```

### Manual Token Exchange

In some environments, deep link redirects may not work reliably (e.g. certain Linux desktop environments, sandboxed browsers, or when the Electron app isn't registered as the default handler for the protocol scheme). As a fallback, users can manually copy the authorization code from the web UI and paste it into the Electron app.

**Front-end:**

After authentication completes, the authorization code is available via the `electron.getAuthorizationCode()` method on the proxy client. You can display this code to the user so they can copy it:

```
import { useEffect } from "react";
import { authClient } from "../auth-client";

function Providers({ children }) {
  useEffect(() => {
    const authorizationCode = authClient.electron.getAuthorizationCode();
    if (authorizationCode) {
      // Display the code to the user
      console.log("Authorization code:", authorizationCode);
    }
  }, []);

  return <>{children}</>;
}
```

The authorization code is also returned after a successful sign-in initiated from the Electron app or when using the `transferUser` method.

```
const { data } = await authClient.electron.transferUser({
  fetchOptions: {
    query: params,
  },
});

if (data?.electron_authorization_code) {
  // Display the code to the user
}
```

**Electron:**

In the Electron renderer, provide an input for the user to paste the authorization code, then call `window.authenticate()` to exchange it for a session:

```
function ManualCodeEntry() {
  return (
    <input
      type="text"
      placeholder="Paste code here"
      maxLength={32}
      onChange={(e) => {
        if (e.target.value.length === 32) {
          // Important: requestAuth() must have been called before
          window.authenticate({ token: e.target.value }); 
        }
      }}
    />
  );
}
```

The `authenticate` bridge sends the authorization code to the main process, which exchanges it for a session using the stored PKCE code verifier and state. On success, the `onAuthenticated` bridge is triggered.

### User Image Proxy

To avoid CSP issues, the Electron plugin securely proxies user avatar images through a custom `user-image://` protocol.

You can use the image URL directly in your renderer:

```
<img src={user.image} alt="Avatar" />
{/* or <img src="user-image://<user-id>" /> */}
```

To configure or disable the proxy, see the [`userImageProxy`](#userimageproxy) option.

### Creating IPC bridges

You should create IPC bridges to extend the functionality exposed to your renderer process. This ensures a minimal, safe API surface.

First, create an IPC handler in the main process that uses the `authClient` to perform the desired action:

```
import { authClient } from "./lib/auth-client";
import { ipcMain } from "electron";

ipcMain.handle("myBridge", async (_event, data) => {
  const cookie = authClient.getCookie();
  return await authClient.someEndpoint({
    data,
    fetchOptions: {
      headers: { cookie },
    },
  });
});
```

Next, expose the bridge in your preload script using `contextBridge`:

```
import { contextBridge, ipcRenderer } from "electron";

contextBridge.exposeInMainWorld("myBridge", (data: Record<string, any>) => {
  return ipcRenderer.invoke("myBridge", data);
});
```

Now you can call your custom bridge from anywhere in the renderer process:

```
useEffect(() => {
  window
    .myBridge({
      foo: "bar",
    })
    .then((res) => {
      console.log("Bridge response:", res);
    });
}, []);
```

For more details, check out Electron's [Inter-Process Communication](https://www.electronjs.org/docs/latest/tutorial/ipc) tutorial.

## Options

### Server plugin

#### codeExpiresIn?

The duration, in seconds, for which the authorization code is valid. (defaults to `300`)

Note that the authorization code will be refreshed during active endpoint usage.

```
electron({
  codeExpiresIn: 300, // 5 minutes
});
```

#### redirectCookieExpiresIn?

The duration, in seconds, for which the redirect cookie remains valid. (defaults to `120`)

```
electron({
  redirectCookieExpiresIn: 120, // 2 minutes
});
```

The redirect cookie name is derived by the `clientID`.

#### cookiePrefix?

The prefix to use for cookies set by the plugin. (defaults to `better-auth`)

```
electron({
  cookiePrefix: "better-auth",
});
```

#### clientID?

The client id to use for identifying the Electron client during authorization. (defaults to `electron`)

Make sure this matches the clientID provided to both the proxy and electron client plugin.

```
electron({
  clientID: "electron",
});
```

### Proxy client

#### protocol

The protocol scheme to use for deep linking in Electron.

Should follow the [reverse domain name notation](https://datatracker.ietf.org/doc/html/rfc8252#section-7.1) to ensure uniqueness.

Make sure this matches the protocol scheme provided to the Electron client and `trustedOrigins`.

```
electronProxyClient({
  protocol: "com.example.app",
});
```

```
electronProxyClient({
  protocol: {
    scheme: "com.example.app",
  },
});
```

#### callbackPath?

The callback path to use for authentication redirects. (defaults to `/auth/callback`)

Make sure this matches the path provided to the electron client plugin.

```
electronProxyClient({
  callbackPath: "/auth/callback",
});
```

#### clientID?

The client id to use for identifying the Electron client during authorization. (defaults to `electron`)

Make sure this matches the clientID provided to both the server and electron client plugin.

```
electronProxyClient({
  clientID: "electron",
});
```

#### cookiePrefix?

The prefix to use for cookies set by the plugin. (defaults to `better-auth`)

```
electronProxyClient({
  cookiePrefix: "better-auth",
});
```

### Client plugin

#### signInURL

The URL to redirect to for authentication.

```
electronClient({
  signInURL: "http://localhost:3000/sign-in",
});
```

#### protocol

The protocol scheme to use for deep linking in Electron.

Should follow the [reverse domain name notation](https://datatracker.ietf.org/doc/html/rfc8252#section-7.1) to ensure uniqueness.

Make sure this matches the protocol scheme provided to the proxy client and `trustedOrigins`.

```
electronProxyClient({
  protocol: "com.example.app",
});
```

```
electronProxyClient({
  protocol: {
    scheme: "com.example.app",
  },
});
```

#### callbackPath?

The callback path to use for authentication redirects. (defaults to `/auth/callback`)

Make sure this matches the path provided to the proxy client plugin.

```
electronClient({
  callbackPath: "/auth/callback",
});
```

#### storage

Storage solution to use to store session and cookie data.

By default a storage file is generated in the `userData` directory. The name is derived by the project name.

```
electronClient({
  storage: {
    getItem: (key) => {
      // get entry from storage
    },
    setItem: (key, value) => {
      // set entry in storage
    },
  },
});
```

#### storagePrefix?

Prefix for local storage keys. (defaults to `better-auth`)

```
electronClient({
  storagePrefix: "better-auth",
});
```

#### cookiePrefix?

Prefix(es) for server cookie names to filter. (defaults to `better-auth`)

This is used to identify which cookies belong to better-auth to prevent infinite refetching when third-party cookies are set.

```
electronClient({
  cookiePrefix: "better-auth",
});
```

```
electronClient({
  cookiePrefix: ["better-auth", "my-app"],
});
```

#### channelPrefix?

Channel prefix for IPC bridges. (defaults to `better-auth`)

```
electronClient({
  channelPrefix: "myapp",
});
```

#### clientID?

The client id to use for identifying the Electron client during authorization. (defaults to `electron`)

Make sure this matches the clientID provided to both the server and proxy client plugin.

```
electronClient({
  clientID: "electron",
});
```

#### sanitizeUser?

A function to sanitize the user object before it is sent to the renderer process. Use this to strip sensitive fields.

```
electronClient({
  sanitizeUser: (user) => {
    const { sensitiveField, ...rest } = user;
    return rest;
  },
});
```

#### userImageProxy?

Configuration for the user image proxy. See [User Image Proxy](#user-image-proxy) for details.

```
electronClient({
  userImageProxy: {
    enabled: true, // default: true
    maxSize: 1024 * 1024 * 5, // default: 5MB
  },
});
```

To disable the proxy entirely:

```
electronClient({
  userImageProxy: {
    enabled: false,
  },
});
```

#### disableCache?

Whether to disable caching the session data locally. (defaults to `false`)

```
electronClient({
  disableCache: true,
});
```
