# agentic-core

[![npm version](https://img.shields.io/npm/v/agentic-core)](https://www.npmjs.com/package/agentic-core) [![Downloads/month](https://img.shields.io/npm/dm/agentic-core)](https://www.npmjs.com/package/agentic-core) [![MIT license](https://img.shields.io/badge/license-MIT-blue)](https://opensource.org/license/mit)

[Setup](#run-your-first-summary) · [API and commands](#runtime-surfaces) · [Coverage](#coverage) · [Comparisons](#compare)

agentic-core is an open-source TypeScript toolkit for developers who build Node.js command-line tools and agent packages. It provides brand configuration, stderr logs, dispatch guards, text routing, and process-local usage records. A command-line interface (CLI) and Model Context Protocol (MCP) server expose text statistics and routing.

## Install

```bash
npm install agentic-core
```

Node.js 18 or later is required. TypeScript declarations are included.
The package declares two runtime dependencies: the MCP SDK and Zod.

## Run your first summary

```bash
npx agentic-core summary --text "hello world hello"
```

Success prints JSON with `chars: 17`, `words: 3`, `lines: 1`, and `hello` counted twice in `topWords`.
No credentials, environment variables, or network calls are required.

Save this as `example.mjs`, then run `node example.mjs`:

```js
import { summarizeText, dispatchText, getCostReport } from 'agentic-core';

console.log(summarizeText('hello world hello'));
console.log(await dispatchText('hello', { dryRun: true }));
console.log(await dispatchText('hello')); // Local receipt only.
console.log(getCostReport());
```

The preview prints `dryRun: true`. The local dispatch returns a receipt ID; the report then shows one dispatch and five units.

## How it works

```text
Text -> summary -> character / word / line counts
Text -> optional guards -> dry-run result
                        -> local or HTTP router -> receipt -> usage record
```

## Why it exists

Related tools need shared configuration and dispatch behavior. agentic-core groups those utilities behind a common TypeScript API.
Pino and Consola focus on logging. This package adds routing and guard helpers, with a simpler string logger.

## Compare

Scope checked on 2026-10-14 against the [Pino documentation](https://github.com/pinojs/pino) and [Consola documentation](https://github.com/unjs/consola).
These are alternatives for the logging portion of this package.

| Surface | agentic-core | Pino | Consola |
|---|---|---|---|
| Main scope | Shared tool utilities and routing | JSON logging | Console logging |
| Log output | Level-prefixed strings | JSON records | Reporter-based output |
| Log configuration | Level, quiet flag, sink function | Logger options and transports | Logger options and reporters |

## Coverage

| Surface | Status | Behavior |
|---|---|---|
| Local router | ✓ | Generates a receipt; no external delivery |
| Generic HTTP router | ✓ | POSTs `{ "text": "..." }` to your endpoint |
| Brand configuration | ✓ | Validates required names; shallow-freezes config |
| Logger | ✓ | Four log levels, quiet mode, custom sink |
| Guards | ✓ | Dispatch count cap, UTC quiet hours, substring blocklist |
| Text statistics | ✓ | Characters, whitespace tokens, lines, top words |
| Usage records | ✓ | In-memory entries and totals |

The source defines two router adapters and three guard types. No named SaaS provider adapters exist.
Tests exercise local behavior and mocked HTTP calls. This rewrite does not verify a deployed remote endpoint.

## Common use cases

1. Node.js CLI authors: keep logs on stderr while JSON results use stdout.
2. TypeScript package maintainers: share brand metadata across SDK, CLI, and MCP entry points.
3. Agent developers: preview an HTTP route before calling a service they deploy.
4. Node.js service developers: apply a dispatch cap and UTC quiet window through an SDK guard instance.
5. MCP client authors: inspect text statistics and session usage through the included tools.

## Features

- **Brand metadata:** reuse a validated configuration object.
- **Logging:** send debug, info, warn, and error strings to stderr or a custom sink.
- **Guards:** reject calls before routing when configured limits match.
- **Routing:** select a local receipt adapter or generic HTTP POST adapter.
- **Usage reports:** read or reset current-process records.
- **Text statistics:** count tokens and list frequent normalized words.

## Runtime surfaces

The Node.js SDK exports `defineBrand`, `createLogger`, `createGuards`, `dispatchText`, both router classes, and text-statistics helpers.
It also exports error classes, dry-run helpers, and cost-record functions.

The CLI provides four operational commands:

```bash
npx agentic-core summary --text "hello" --top 3
npx agentic-core dispatch --text "hello" --dry-run
npx agentic-core costs
npx agentic-core mcp
```

`summary` and `dispatch` also read `--file`. Help uses `--help`/`-h`; version uses `--version`/`-V`.
The default CLI router is local. There is no endpoint or guard configuration flag.

The MCP server exposes three tools: `summarize_text`, `dispatch_text`, and `get_cost_report`.
Start it with `npx agentic-core mcp` or `npx agentic-core-mcp`.
The default MCP router is also local.

The Worker source provides `GET /health` and `POST /summarize` with a JSON `text` field.
Worker deployment and runtime compatibility configuration belong to your project; no managed service is included.

## Delivery and behavior limits

- SDK and MCP dispatch default to live execution. Set `dryRun: true`; the CLI uses `--dry-run`.
- The local router generates metadata. It neither stores nor sends the text.
- HTTP routing has a ten-second default timeout and no retries, authentication option, deduplication, or durable queue.
- HTTP success creates a local receipt with a generated ID. The adapter does not parse the response body or verify downstream delivery.
- Guards are optional. CLI defaults configure none; custom SDK or MCP integrations must supply them.
- Caps count completed calls in one guard instance. Concurrent calls can pass the check before the counter updates.
- Quiet hours use UTC. The blocklist matches case-insensitive substrings; it is not a content classifier.
- Cost units are character counts for dispatches, not money. State resets when the process exits.
- A separate `costs` CLI invocation cannot read an earlier dispatch process's records.
- Summary word totals count whitespace tokens. Top-word normalization keeps ASCII letters and digits only.
- `defineBrand` freezes the outer object, not the nested keywords array.

## Integration guides

- Custom HTTP routing: construct `new HttpRouterAdapter(endpoint)` and pass it as the SDK `adapter` option.
- Dispatch guards: construct `createGuards({ caps: { maxDispatches: 10 } })` and pass the instance as `guards`.
- Custom logging: pass `createLogger({ level: 'warn', sink: line => console.error(line) })` as `logger`.
- MCP customization: the server source exports `createMcpServer` and `startMcp` with router, guard, and logger dependencies.

The root SDK export does not expose MCP server factories. Integrate those through your own server build rather than an undocumented package subpath.

## Status

Version 0.1.0 is an early release. APIs can change before 1.0.
Use [GitHub issues](https://github.com/pooriaarab/agentic-core/issues) for support. Public case studies are not yet published in this repository.

## FAQ

### What is agentic-core?

agentic-core is a Node.js toolkit for configuration, logging, guards, and text routing. It includes sample CLI, MCP, and Worker surfaces.

### Which integrations are verified?

agentic-core tests local routing and mocked HTTP behavior. It does not include named vendor integrations or a verified hosted endpoint.

### Does agentic-core provide a managed router?

agentic-core runs in your process. You provide any HTTP destination and operate its service.

### How is agentic-core different from Pino?

agentic-core includes a string logger plus guards and routing helpers. Pino focuses on JSON logging and provides transport APIs.

### How is agentic-core different from Consola?

agentic-core combines basic logging with dispatch utilities. Consola focuses on console logging and reporter-based output.

### Does local dispatch deliver a message?

agentic-core local dispatch creates a receipt with a random ID, character count, and timestamp. It performs no external delivery.

### Are guards enabled by default?

agentic-core checks guards only when you pass an instance. The default CLI and MCP startup do not configure guards.

### Is dispatch a dry run by default?

agentic-core dispatch defaults to live adapter execution. Set `dryRun: true` in the SDK or MCP call before a real dispatch.

### Does the cost report show currency?

agentic-core dispatch records character-count units. It does not calculate prices or reconcile invoices.

## Development gates

Run the repository checks before pushing:

```bash
npm run build && npm run typecheck && npm test
```

Smoke-test the installed CLI from a tarball:

```bash
npm pack
npx --yes --package ./agentic-core-0.1.0.tgz agentic-core summary --text "hello"
```

## Release flow

Open a pull request against `main`. The publish workflow runs from `release` and skips versions already on npm.
A README update alone does not refresh the published npm package.

## Layout

The SDK entry is `src/index.ts`; shared behavior lives in `src/core.ts`.
CLI and MCP sources use `src/cli.ts`, `src/mcp.ts`, and `src/mcp-bin.ts`.
`src/worker.ts` contains the Worker entry. The repository also includes the TypeScript contract and contract documentation.

## License

[MIT](https://opensource.org/license/mit).
