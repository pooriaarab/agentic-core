---
name: agentic-core
description: Shared core for CLI tools: brand config, logs, and guards. Keywords: shared-types, config, guards, agentic-suite, claude-code, codex, cli, mcp.
---

# agentic-core

Shared core for CLI tools: brand config, logs, and guards.

## Use the SDK

```js
import { summarizeText, dispatchText } from "@pooriaarab/agentic-core";

summarizeText("some text");
await dispatchText("some text", { dryRun: true });
```

## Use the CLI

```sh
agentic-core summary --text "some text"
agentic-core dispatch --text "some text" --dry-run
```

## Use the MCP server

Start the server with `agentic-core mcp`. Call `summarize_text`
for stats, `dispatch_text` for routes, `get_cost_report` for costs.

## Rules

- Preview each route with dry run first.
- Report guard denials and adapter errors as failures.
- Read the cost report at the end.
