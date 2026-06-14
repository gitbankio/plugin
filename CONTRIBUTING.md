# Contributing to gitbankio/plugin

Thank you for your interest in contributing.

## What lives here

This repo contains exactly one deliverable: `plugin.md`. It is the canonical Gitbank AI plugin file -- a structured markdown document that teaches any AI assistant how to interact with the Gitbank API in relayer mode (no wallet required).

## How to contribute

**Improving the plugin file**

- Open an issue describing what is missing, unclear, or incorrect
- Submit a pull request against `plugin.md`
- Changes must remain AI-readable: clear section headers, consistent parameter tables, working example sessions

**Adding platform-specific guidance to the README**

- If you successfully load the plugin into an AI not listed in the README, open a PR adding setup instructions for that platform
- Keep instructions concise: upload file, prompt format, any platform-specific caveats

## Rules

- English only
- No wallet connection instructions -- this plugin is relayer mode only
- No references to internal tooling
- Keep the plugin file self-contained -- it must work without any external context

## Testing your changes

After editing `plugin.md`, load it into at least one AI assistant and verify:

1. Balance check works: `"Check my vault. GitHub: <testuser>"`
2. Prepare flow works: `"Swap 1 USDC to WETH. GitHub: <testuser>"` -- AI should call the prepare endpoint and show the confirm instructions

## License

Apache 2.0
