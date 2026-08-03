---
url: https://better-auth.com/llms.txt/docs/guides/browser-extension-guide
title: "Browser Extension Guide"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

A step-by-step guide to creating a browser extension with Better Auth.

In this guide, we'll walk you through the steps of creating a browser extension using [Plasmo](https://docs.plasmo.com/) with Better Auth for authentication.

If you would like to view a completed example, you can check out the [browser extension example](https://github.com/better-auth/examples/tree/main/browser-extension-example).

## Setup & Installations

Initialize a new Plasmo project with TailwindCSS and a src directory.

```
pnpm create plasmo --with-tailwindcss --with-src
```

Then, install the Better Auth package.

```
pnpm add better-auth
```

To start the Plasmo development server, run the following command.

```
pnpm dev
```

## Configure tsconfig

Configure the `tsconfig.json` file to include `strict` mode.

For this demo, we have also changed the import alias from `~` to `@` and set it to the `src` directory.

```
{
    "compilerOptions": {
        "paths": {
            "@/_": [
                "./src/_"
            ]
        },
        "strict": true,
        "baseUrl": "."
    }
}
```

## Create the client auth instance

Create a new file at `src/auth/auth-client.ts` and add the following code.

```
import { createAuthClient } from "better-auth/react"

export const authClient = createAuthClient({
    baseURL: "http://localhost:3000" /* Base URL of your Better Auth backend. */,
    plugins: [],
});
```

## Configure the manifest

We must ensure the extension knows the URL to the Better Auth backend.

Head to your package.json file, and add the following code.

```
{
    //...
    "manifest": {
        "host_permissions": [
            "https://URL_TO_YOUR_BACKEND" // localhost works too (e.g. http://localhost:3000)
        ]
    }
}
```

## You're now ready!

You have now set up Better Auth for your browser extension.

Add your desired UI and create your dream extension!

To learn more about the client Better Auth API, check out the [client documentation](https://better-auth.com/docs/concepts/client).

Here's a quick example 😎

```
import { authClient } from "./auth/auth-client"

function IndexPopup() {
    const {data, isPending, error} = authClient.useSession();
    if(isPending){
        return <>Loading...</>
    }
    if(error){
        return <>Error: {error.message}</>
    }
    if(data){
        return <>Signed in as {data.user.name}</>
    }
}

export default IndexPopup;
```

## Bundle your extension

To get a production build, run the following command.

```
pnpm build
```

![](https://docs.plasmo.com/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fdeveloper_mode.76f090f7.png&w=1920&q=75)

Click on "Load Unpacked" and navigate to your extension's `build/chrome-mv3-dev` (or `build/chrome-mv3-prod`) directory.

To see your popup, click on the puzzle piece icon on the Chrome toolbar, and click on your extension.

Learn more about [bundling your extension here.](https://docs.plasmo.com/framework#loading-the-extension-in-chrome)

## Configure the server auth instance

First, we will need your extension URL.

An extension URL formed like this: `chrome-extension://YOUR_EXTENSION_ID`.

![](https://better-auth.com/extension-id.png)

Head to your server's auth file, and make sure that your extension's URL is added to the `trustedOrigins` list.

```
import { betterAuth } from "better-auth"
import { auth } from "@/auth/auth"

export const auth = betterAuth({
    trustedOrigins: ["chrome-extension://YOUR_EXTENSION_ID"],
})
```

If you're developing multiple extensions or need to support different browser extensions with different IDs, you can use wildcard patterns:

```
export const auth = betterAuth({
    trustedOrigins: [
        // Support a specific extension ID
        "chrome-extension://YOUR_EXTENSION_ID",
        
        // Or support multiple extensions with wildcard (less secure)
        "chrome-extension://*"
    ],
})
```

## That's it!

Everything is set up! You can now start developing your extension. 🎉

## Wrapping Up

Congratulations! You've successfully created a browser extension using Better Auth and Plasmo. We highly recommend you visit the [Plasmo documentation](https://docs.plasmo.com/) to learn more about the framework.

If you would like to view a completed example, you can check out the [browser extension example](https://github.com/better-auth/examples/tree/main/browser-extension-example).

If you have any questions, feel free to open an issue on our [GitHub repo](https://github.com/better-auth/better-auth/issues), or join our [Discord server](https://discord.gg/better-auth) for support.
