---
url: https://better-auth.com/llms.txt/docs/integrations/nuxt
title: "Nuxt"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Integrate Better Auth with Nuxt.

Before you start, make sure you have a Better Auth instance configured. If you haven't done that yet, check out the [installation](https://better-auth.com/docs/installation).

## Mount the handler

Mount the Better Auth handler to a catch-all Nitro route. Create a file at `server/api/auth/[...all].ts`:

```
import { auth } from "~~/lib/auth"; // import your auth config

export default defineEventHandler((event) => {
    return auth.handler(toWebRequest(event));
});
```

## Create a client

Create an auth client using `better-auth/vue` so `useSession` returns Vue refs:

```
import { createAuthClient } from "better-auth/vue";

export const authClient = createAuthClient();
export const { signIn, signUp, signOut, useSession } = authClient;
```

## Use the session

Use `authClient.useSession(useFetch)` inside a page `setup()` to load the session on the server and hydrate it on the client. Pass Nuxt's `useFetch` so the request is made with the incoming cookies and the payload is reused during hydration:

```
<script setup lang="ts">
import { authClient } from "~~/lib/auth-client";

const { data: session } = await authClient.useSession(useFetch);
</script>

<template>
    <div v-if="session">
        <p>Welcome, {{ session.user.name }}</p>
        <button @click="authClient.signOut()">Sign out</button>
    </div>
    <button v-else @click="authClient.signIn.social({ provider: 'github' })">
        Continue with GitHub
    </button>
</template>
```

For client-only widgets like popovers or menus, call `authClient.useSession()` without an argument. It returns a reactive ref that updates on sign-in and sign-out.

## Protect pages

Create a named route middleware and opt pages into it with `definePageMeta`:

```
import { authClient } from "~~/lib/auth-client";

export default defineNuxtRouteMiddleware(async (to) => {
    const { data: session } = await authClient.useSession(useFetch);
    if (!session.value) {
        return navigateTo({ path: "/login", query: { redirect: to.fullPath } });
    }
});
```

```
<script setup lang="ts">
definePageMeta({ middleware: "auth" });
</script>
```

Rename the file to `app/middleware/auth.global.ts` to run it on every route. Always `return` the `navigateTo(...)` call, since calling it without `return` is a no-op.

## Protect server routes

Call `auth.api.getSession` with the incoming `event.headers` and guard the route with `createError`:

```
import { auth } from "~~/lib/auth";

export default defineEventHandler(async (event) => {
    const session = await auth.api.getSession({ headers: event.headers });
    if (!session?.user) {
        throw createError({ statusCode: 401, statusMessage: "Unauthorized" });
    }
    return { user: session.user };
});
```

If you need to reuse this across many routes, factor the check into your own `server/utils/` helper.

## Use the client during SSR

Aside from `useSession(useFetch)`, `authClient` actions don't forward cookies during SSR by default, so they return as unauthenticated. Two ways to handle this:

**Approach A: call them on the client only.** Wrap the consumer in `<ClientOnly>` or guard with `import.meta.client`:

```
<script setup lang="ts">
import { authClient } from "~~/lib/auth-client";

const accounts = ref<Awaited<ReturnType<typeof authClient.listAccounts>>["data"]>();

onMounted(async () => {
    const { data } = await authClient.listAccounts();
    accounts.value = data;
});
</script>
```

**Approach B: create a request-scoped client that forwards cookies during SSR.** Add a `useAuth` composable and combine it with `useAsyncData`:

```
import { createAuthClient } from "better-auth/vue";

export function useAuth() {
    const url = useRequestURL();
    const headers = import.meta.server ? useRequestHeaders(["cookie"]) : undefined;
    return createAuthClient({
        baseURL: url.origin,
        fetchOptions: { headers },
    });
}
```

```
<script setup lang="ts">
const { data: accounts } = await useAsyncData("accounts", () =>
    useAuth().listAccounts().then((res) => res.data),
);
</script>
```

## Reuse with a Nuxt layer

When you want to reuse the same auth setup across multiple Nuxt apps (for example a marketing site and a dashboard), extract `lib/auth.ts`, `server/`, `app/middleware/`, and `app/composables/` into a [Nuxt layer](https://nuxt.com/docs/4.x/getting-started/layers) and extend it from each app:

```
export default defineNuxtConfig({
    extends: ["./layers/auth"],
});
```
