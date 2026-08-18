---
url: https://better-auth.com/llms.txt/docs/concepts/cli
title: "Cli"
description: ""
access_date: 2026-08-18T00:08:46.984Z
current_date: 2026-08-18T00:08:46.984Z
---

# CLI

Learn about the Better Auth CLI commands for generating and migrating database schemas, creating initial admins, initializing projects, generating secret keys, and gathering diagnostic info.



Better Auth comes with a built-in CLI to help you manage the database schemas, create initial admin users, initialize your project, generate a secret key for your application, and gather diagnostic information about your setup.

## ## Generate
The `generate` command creates the schema required by Better Auth. If you're using a database adapter like Prisma or Drizzle, this command will generate the right schema for your ORM. If you're using the built-in Kysely adapter, it will generate an SQL file you can run directly on your database.




#### npm

```bash title="Terminal"
npx auth@latest generate
```

#### pnpm

```bash title="Terminal"
pnpm dlx auth@latest generate
```

#### yarn

```bash title="Terminal"
yarn dlx auth@latest generate
```

#### bun

```bash title="Terminal"
bun x auth@latest generate
```


## ### Options
* `--output` - Where to save the generated schema. For Prisma, it will be saved in prisma/schema.prisma. For Drizzle, it goes to schema.ts in your project root. For Kysely, it's an SQL file saved as schema.sql in your project root.
* `--config&#x60; - The path to your Better Auth config file. By default, the CLI will search for an auth.ts file in &#x2A;*./*&#x2A;, **./utils*&#x2A;, **./lib**, or any of these directories under the `src` directory.
* `--yes` - Skip the confirmation prompt and generate the schema directly.

## ## Migrate
The migrate command applies the Better Auth schema directly to your database. This is available if you're using the built-in Kysely adapter. For other adapters, you'll need to apply the schema using your ORM's migration tool.




#### npm

```bash title="Terminal"
npx auth@latest migrate
```

#### pnpm

```bash title="Terminal"
pnpm dlx auth@latest migrate
```

#### yarn

```bash title="Terminal"
yarn dlx auth@latest migrate
```

#### bun

```bash title="Terminal"
bun x auth@latest migrate
```


## ### Options
* `--config&#x60; - The path to your Better Auth config file. By default, the CLI will search for an auth.ts file in &#x2A;*./*&#x2A;, **./utils*&#x2A;, **./lib**, or any of these directories under the `src` directory.
* `--yes` - Skip the confirmation prompt and apply the schema directly.

> **Using PostgreSQL with a non-default schema?**
> 
> The migrate command automatically detects your configured `search_path` and creates tables in the correct schema. See the [PostgreSQL adapter documentation](/docs/adapters/postgresql#use-a-non-default-schema) for configuration details.

## ## Create Admin
The `create-admin` command creates an initial admin user through your configured Better Auth instance. It requires the Admin plugin and a persistent database, and it uses the same server-side `auth.api.createUser` path as the Admin plugin so passwords are hashed and database hooks still run.




#### npm

```bash title="Terminal"
npx auth@latest create-admin --email admin@example.com --name "Admin" --role admin
```

#### pnpm

```bash title="Terminal"
pnpm dlx auth@latest create-admin --email admin@example.com --name "Admin" --role admin
```

#### yarn

```bash title="Terminal"
yarn dlx auth@latest create-admin --email admin@example.com --name "Admin" --role admin
```

#### bun

```bash title="Terminal"
bun x auth@latest create-admin --email admin@example.com --name "Admin" --role admin
```


If users already exist, the command asks for confirmation. Use `--force` or `--yes` to skip that prompt.

## ### Options
* `--email` - The email address for the admin user.
* `--password` - The password for the admin user. If omitted, the CLI will prompt for it.
* `--name` - The name for the admin user. Defaults to `Admin`.
* `--role` - The role to assign. Defaults to `admin`.
* `--data` - Additional user fields as a JSON object.
* `--no-email-verified` - Create the admin user with an unverified email. By default, the CLI marks the admin email as verified.
* `--config` - The path to your Better Auth config file.
* `--force` - Create the admin user even when users already exist.
* `--yes` - Skip the existing-users confirmation prompt.

## ## Init
The `init` command allows you to initialize Better Auth in your project.




#### npm

```bash title="Terminal"
npx auth@latest init
```

#### pnpm

```bash title="Terminal"
pnpm dlx auth@latest init
```

#### yarn

```bash title="Terminal"
yarn dlx auth@latest init
```

#### bun

```bash title="Terminal"
bun x auth@latest init
```


## ### Options
* `--name` - The name of your application. (defaults to the `name` property in your `package.json`).
* `--framework` - The framework your codebase is using. Currently, the only supported framework is `Next.js`.
* `--plugins` - The plugins you want to use. You can specify multiple plugins by separating them with a comma.
* `--database` - The database you want to use. Currently, the only supported database is `SQLite`.
* `--package-manager` - The package manager you want to use. Currently, the only supported package managers are `npm`, `pnpm`, `yarn`, `bun` (defaults to the manager you used to initialize the CLI).

## ## Upgrade
The `upgrade` command updates older `better-auth` dependencies and official `@better-auth/*` packages that participate in the synchronized release train to the version of the CLI being run. It compares the minimum version allowed by each `package.json` specifier; specifiers whose minimum is the same or newer are left unchanged. Independently versioned packages, such as `@better-auth/utils`, are not changed.




#### npm

```bash title="Terminal"
npx auth@latest upgrade
```

#### pnpm

```bash title="Terminal"
pnpm dlx auth@latest upgrade
```

#### yarn

```bash title="Terminal"
yarn dlx auth@latest upgrade
```

#### bun

```bash title="Terminal"
bun x auth@latest upgrade
```


## ### Options
* `--cwd` - The project directory containing the `package.json` to update. Defaults to the current directory.
* `--yes` - Skip the confirmation prompt and install the updates directly.

## ## Info
The `info` command provides diagnostic information about your Better Auth setup and environment. Useful for debugging and sharing when seeking support.




#### npm

```bash title="Terminal"
npx auth@latest info
```

#### pnpm

```bash title="Terminal"
pnpm dlx auth@latest info
```

#### yarn

```bash title="Terminal"
yarn dlx auth@latest info
```

#### bun

```bash title="Terminal"
bun x auth@latest info
```


## ### Output
The command displays:

* **System**: OS, CPU, memory, Node.js version
* **Package Manager**: Detected manager and version
* **Better Auth**: Version and configuration (sensitive data auto-redacted)
* **Frameworks**: Detected frameworks (Next.js, React, Vue, etc.)
* **Databases**: Database clients and ORMs (Prisma, Drizzle, etc.)

## ### Options
* `--config` - Path to your Better Auth config file
* `--json` - Output as JSON for sharing or programmatic use

## ### Examples
```bash
# Basic usage
npx auth@latest info

# Custom config path
npx auth@latest info --config ./config/auth.ts

# JSON output
npx auth@latest info --json > auth-info.json
```

Sensitive data like secrets, API keys, and database URLs are automatically replaced with `[REDACTED]` for safe sharing.

## ## Secret
The CLI also provides a way to generate a secret key for your Better Auth instance.




#### npm

```bash title="Terminal"
npx auth@latest secret
```

#### pnpm

```bash title="Terminal"
pnpm dlx auth@latest secret
```

#### yarn

```bash title="Terminal"
yarn dlx auth@latest secret
```

#### bun

```bash title="Terminal"
bun x auth@latest secret
```


## ## Common Issues
**Error: Cannot find module X**

The CLI resolves most imports for you: `tsconfig.json` path aliases (including SvelteKit's `$lib`) and stubbed framework virtual modules (`$env/*`, `$app/*`, `cloudflare:workers`, Vite assets like `?raw`). For SvelteKit, run `svelte-kit sync` first so `.svelte-kit/tsconfig.json` exists.

A few module types can't load outside their bundler (e.g. `.svelte` components or `import.meta.glob`). Keep those out of your config file's import graph.
