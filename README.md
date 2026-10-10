# Agentic core

Shared core for CLI tools: brand config, logs, and guards.

One core serves three faces: a terminal command, an npm import, and an MCP server. Each face returns the same shape.

## Install

```sh
npm install agentic-core
```

## Use it

Run the command:

```sh
npx agentic-core summary --text "hello world hello"
```

Import the library:

```js
import { summarizeText } from "agentic-core";

console.log(summarizeText("hello world hello"));
```

Start the MCP server:

```sh
npx agentic-core mcp
```

## Three usecases

1. You define one brand config and reuse it in each tool you ship.
2. You send logs to stderr with a level filter and a quiet flag.
3. You guard a dispatch with caps, quiet hours, and a blocklist.

## What you get

- Brand config with one call.
- Logs that stay on stderr, out of result output.
- Guards: caps, quiet hours, and a blocklist.
- Routes with dry run previews and cost reports.
- The same API on the CLI, the library, and MCP.

## Why the core has no deps

Pino leads for services, consola suits CLIs, and c12 suits file config. This core pins a small frozen API, so it ships with no deps. It stays light and runs on edge workers. Packs add pino or c12 when they need transports or file config.

## Gates

Run all three before you push:

```sh
npm run build && npm run typecheck && npm test
```

Then run the real installed CLI once from a tarball:

```sh
npm pack
npx --yes --package ./pooriaarab-agentic-core-0.1.0.tgz agentic-core summary --text "hello"
```

## Release flow

Push to `main` for review. Merge to `release` to publish. The publish workflow ships only when the version is new.

## Layout

- `src/index.ts`: the library face.
- `src/cli.ts`: the terminal face.
- `src/mcp.ts`: the MCP server. `src/mcp-bin.ts` starts it.
- `src/worker.ts`: the edge worker face.
- `src/core.ts`: the shared core.
- `agentic-core.d.ts`: the frozen contract.
- `docs/agentic-core.md`: the contract in plain words.

## License

MIT. See `LICENSE`.
