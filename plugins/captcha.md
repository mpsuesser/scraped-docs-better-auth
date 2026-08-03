---
url: https://better-auth.com/llms.txt/docs/plugins/captcha
title: "Captcha"
description: ""
access_date: 2026-08-03T19:43:07.705Z
current_date: 2026-08-03T19:43:07.705Z
---

Captcha plugin

The **Captcha Plugin** integrates bot protection into your Better Auth system by adding captcha verification for key endpoints. This plugin ensures that only human users can perform actions like signing up, signing in, or resetting passwords. The following providers are currently supported:

- [Google reCAPTCHA](https://developers.google.com/recaptcha)
- [Cloudflare Turnstile](https://www.cloudflare.com/application-services/products/turnstile/)
- [hCaptcha](https://www.hcaptcha.com/)
- [CaptchaFox](https://captchafox.com/)

## Installation

### Add the plugin to your auth config

```
import { betterAuth } from "better-auth";
import { captcha } from "better-auth/plugins"; 

export const auth = betterAuth({
    plugins: [
        captcha({ 
            provider: "cloudflare-turnstile", // or google-recaptcha, hcaptcha, captchafox
            secretKey: process.env.TURNSTILE_SECRET_KEY!, 
        }), 
    ],
});
```

### Add the captcha token to your request headers

Add the captcha token to your request headers for all protected endpoints. This example shows how to include it in a `signIn` request:

```
import { authClient } from "@/lib/auth-client"

await authClient.signIn.email({
    email: "user@example.com",
    password: "secure-password",
    fetchOptions: { 
        headers: { 
            "x-captcha-response": turnstileToken, 
        }, 
    }, 
});
```

- To implement Cloudflare Turnstile on the client side, follow the official [Cloudflare Turnstile documentation](https://developers.cloudflare.com/turnstile/) or use a library like [react-turnstile](https://www.npmjs.com/package/@marsidev/react-turnstile).
- To implement Google reCAPTCHA on the client side, follow the official [Google reCAPTCHA documentation](https://developers.google.com/recaptcha/intro) or use libraries like [react-google-recaptcha](https://www.npmjs.com/package/react-google-recaptcha) (v2) and [react-google-recaptcha-v3](https://www.npmjs.com/package/react-google-recaptcha-v3) (v3).
- To implement hCaptcha on the client side, follow the official [hCaptcha documentation](https://docs.hcaptcha.com/#add-the-hcaptcha-widget-to-your-webpage) or use libraries like [@hcaptcha/react-hcaptcha](https://www.npmjs.com/package/@hcaptcha/react-hcaptcha)
- To implement CaptchaFox on the client side, follow the official [CaptchaFox documentation](https://docs.captchafox.com/getting-started) or use libraries like [@captchafox/react](https://www.npmjs.com/package/@captchafox/react)

## How it works

The plugin acts as a middleware: it intercepts all `POST` requests to configured endpoints (see `endpoints` in the [Plugin Options](#plugin-options) section).

it validates the captcha token on the server, by calling the captcha provider's `/siteverify`.

- if the token is missing, gets rejected by the captcha provider, or if the `/siteverify` endpoint is unavailable, the plugin returns an error and interrupts the request.
- if the token is accepted by the captcha provider, the middleware returns `undefined`, meaning the request is allowed to proceed.

## Plugin Options

- **`provider` (required)**: your captcha provider.
- **`secretKey` (required)**: your provider's secret key used for the server-side validation.
- `endpoints` (optional): replaces the default array of paths where captcha verification is enforced. If set, only the specified paths will be protected. Default is `["/sign-up/email", "/sign-in/email", "/request-password-reset"]`.
- `minScore` (optional - only *Google ReCAPTCHA v3*): minimum score threshold. Default is `0.5`.
- `siteKey` (optional - only *hCaptcha* and *CaptchaFox*): prevents tokens issued on one sitekey from being redeemed elsewhere.
- `siteVerifyURLOverride` (optional): overrides endpoint URL for the captcha verification request.
