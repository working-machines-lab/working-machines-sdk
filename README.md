<p align="center">
  <img src="https://www.workingmachines.dev/sites/composio-dev-70580fcb/root-8a5edab2/brand/working-machines-logo.png" width="112" alt="Working Machines logo" />
</p>

<h1 align="center">Working Machines SDK</h1>

<p align="center">
  The TypeScript SDK for giving AI agents access to real software—without handing them raw credentials.
</p>

<p align="center">
  <a href="https://www.workingmachines.dev">Website</a> ·
  <a href="https://www.workingmachines.dev/docs">Docs</a> ·
  <a href="https://app.workingmachines.dev">Console</a>
</p>

> Status: public SDK launch draft. The package name and repository URL below are placeholders until the Working Machines package is published.

## What it is

Working Machines is an execution layer for AI agents. Your agent asks to use Gmail, GitHub, Slack, Stripe, HubSpot, or thousands of other apps; Working Machines selects an authorized connection and executes the action server-side.

Your code receives a typed result. It does not receive provider OAuth tokens, API keys, or a broad browser session.

```text
Your app or agent
        │
        ▼
Working Machines SDK
        │  signed request + chosen connection
        ▼
Working Machines
        │  isolated credentials + policy + audit trail
        ▼
Gmail · GitHub · Slack · Stripe · 1,400+ apps
```

## Why use it

- One action interface across thousands of apps.
- Credentials stay in Working Machines; they are never passed into your model or application process.
- OAuth, API-key connections, scopes, retries, and provider differences are handled once.
- Discover actions and input schemas at runtime for dynamic agent workflows.
- Use a named connection when an account has more than one identity.
- Get consistent errors, execution IDs, and audit-friendly results.

## Quickstart

Create an API key in the [Working Machines console](https://app.workingmachines.dev/access), then install the SDK when published:

```sh
npm install @working-machines/sdk
```

```ts
import { WorkingMachines } from "@working-machines/sdk";

const wm = new WorkingMachines({
  apiKey: process.env.WORKING_MACHINES_API_KEY!,
});

const result = await wm.execute("github.get_current_user", {});
console.log(result);
```

Every call runs through `https://app.workingmachines.dev/v1` by default.

## Use an app action

Use the direct action id when an agent decides what to run:

```ts
await wm.execute("gmail.search_threads", {
  query: "from:customer newer_than:7d",
});
```

Or use the provider namespace for concise application code:

```ts
await wm.gmail.search_threads({
  query: "from:customer newer_than:7d",
});
```

When you have multiple connected accounts, select one deliberately:

```ts
await wm.github.create_issue(
  {
    owner: "acme",
    repo: "product",
    title: "Follow up with design",
  },
  { connectionName: "work" },
);
```

## Discover tools at runtime

The catalog makes it practical to build an agent UI or let an agent find an action before it acts:

```ts
const providers = await wm.catalog.providers({ q: "customer support" });
const actions = await wm.catalog.actions("slack");
const action = await wm.catalog.action("slack.send_message");

console.log(action.inputSchema);
```

For an endpoint that is not yet modeled as an action, use a provider proxy through the same selected connection:

```ts
const response = await wm.proxy("github", {
  endpoint: "/user/repos",
  method: "GET",
});
```

## Build safe agent workflows

Working Machines is designed around a simple boundary: models can decide *what* to request; Working Machines controls *whether*, *where*, and *how* it executes.

```text
Agent proposes an action
        │
        ▼
Action + connection + scopes are evaluated
        │
        ▼
Working Machines executes with isolated credentials
        │
        ▼
Your agent receives the allowed result
```

For actions that send, publish, modify, or delete data, keep user intent explicit in your application. Treat provider actions as real external side effects.

## Configuration

```ts
new WorkingMachines({
  apiKey: process.env.WORKING_MACHINES_API_KEY!,
  baseUrl: "https://app.workingmachines.dev/v1", // default
  connectionName: "work",                         // optional default account
  timeoutMs: 30_000,
  maxRetries: 2,
});
```

Per-call options override the client default:

```ts
const work = wm.using({ connectionName: "work" });
await work.slack.send_message(
  { channel: "#product", text: "Release is live." },
  { timeoutMs: 10_000 },
);
```

## Self-hosted runtime

For a local or self-hosted Working Machines runtime, use the self-hosted client:

```ts
import { WorkingMachinesSelfHosted } from "@working-machines/sdk";

const wm = new WorkingMachinesSelfHosted({
  baseUrl: "http://localhost:3000",
  runtimeToken: process.env.WORKING_MACHINES_RUNTIME_TOKEN,
});

await wm.execute("hackernews.get_top_stories", {});
```

## Development

```sh
bun install
bun run check
```

The check runs linting, strict TypeScript checks, ESM/CJS/declaration builds, unit tests with coverage, and published-package type-resolution tests.

## License

MIT © 2026 Working Machines — see [LICENSE](./LICENSE).
