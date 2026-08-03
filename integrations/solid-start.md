---
url: https://better-auth.com/llms.txt/docs/integrations/solid-start
title: "Solid Start"
description: ""
access_date: 2026-08-03T19:38:28.543Z
current_date: 2026-08-03T19:38:28.543Z
---

# SolidStart Integration

Integrate Better Auth with SolidStart.



Before you start, make sure you have a Better Auth instance configured. If you haven't done that yet, check out the [installation](/docs/installation).

Mount the handler [#mount-the-handler]

We need to mount the handler to SolidStart server. Put the following code in your `[...auth].ts` file inside `/routes/api/auth` folder.

```ts title="[...auth].ts"
import { auth } from "~/lib/auth";
import { toSolidStartHandler } from "better-auth/solid-start";

export const { GET, POST } = toSolidStartHandler(auth);
```
