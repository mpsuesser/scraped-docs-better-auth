---
url: https://better-auth.com/llms.txt/docs/examples/next-js
title: "Next Js"
description: ""
access_date: 2026-08-28T22:16:12.077Z
current_date: 2026-08-28T22:16:12.077Z
---

# Next.js Example (/docs/examples/next-js)

Better Auth Next.js example.



This is an example of how to use Better Auth with Next.

**Implements the following features:**
Email & Password . Social Sign-in . Passkeys . Email Verification . Password Reset . Two Factor Authentication . Profile Update . Session Management . Organization, Members and Roles

See [Demo](https://demo.better-auth.com)

[View example on GitHub](https://github.com/better-auth/better-auth/tree/main/demo/nextjs)

[Better Auth Next.js Example](https://stackblitz.com/github/better-auth/better-auth/tree/main/demo/nextjs?codemirror=1&fontsize=14&hidenavigation=1&runonclick=1&hidedevtools=1)

## ## How to run
1. Clone the code sandbox (or the repo) and open it in your code editor
2. Move .env.example to .env and provide necessary variables
3. Run the following commands
   ```bash
   pnpm install
   pnpm dev
   ```
4. Open the browser and navigate to `http://localhost:3000`

## ### SSO Login Example
For this example, we utilize DummyIDP. Initiate the login from the [DummyIDP login](https://dummyidp.com/apps/app_01k16v4vb5yytywqjjvv2b3435/login), click "Proceed", and from here it will direct you to user's dashboard.

## ### SCIM Sync Example
For this example, we utilize DummyIDP. Head out to the [IDP dashboard](https://dummyidp.com/apps/app_01k16v4vb5yytywqjjvv2b3435) and try to add, update or remove users, then go to the admin page or directly to your database and watch the synchronization work.
