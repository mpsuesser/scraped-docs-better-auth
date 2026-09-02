---
url: https://better-auth.com/llms.txt/docs/ai-resources/mcp
title: "Mcp"
description: ""
access_date: 2026-09-02T12:04:16.047Z
current_date: 2026-09-02T12:04:16.047Z
---

Connect Better Auth documentation to MCP-capable clients via the remote documentation MCP server.

Better Auth hosts a **remote MCP server** that exposes documentation search, examples, and setup help to any MCP-capable client (Cursor, Codex, Claude Code, Open Code, and others).

**Endpoint:** `https://mcp.better-auth.com/mcp`

This is separate from the [MCP plugin](https://better-auth.com/docs/plugins/mcp), which adds MCP *provider authentication* to your app. The server above is for **consuming** Better Auth docs inside your editor or agent.

## Using the CLI

Run the [Better Auth CLI](https://better-auth.com/docs/concepts/cli) and pick your client:

```
npx auth@latest mcp
```

With no flags, the command lists supported targets. You can pass a target directly:

#### Cursor

```
npx auth@latest mcp --cursor
```

This opens Cursor with a deeplink so the server is added to your MCP configuration.

You can also use the one-click control on this site:
