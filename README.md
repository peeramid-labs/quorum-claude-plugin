# quorum-claude-plugin

Claude Code plugin that adds `/quorum:brainstorm <topic>` — a slash
command that submits a topic to a multi-agent deliberation against a
[quorum](https://github.com/peeramid-labs/quorum-rs) orchestrator and
streams the converged answer back into Claude.

## Install

```bash
claude plugin install peeramid-labs/quorum-claude-plugin
```

Then in Claude Code:

```
/quorum:brainstorm Should we adopt elliptic-curve identifiers for agent attestation?
```

## Prerequisites

The skill expects, in your shell environment:

| Requirement | Check |
|---|---|
| `quorum` CLI (≥ 0.6.0) | `cargo install quorum-cli` |
| `QUORUM_DEMO_TOKEN` env var | `export QUORUM_DEMO_TOKEN=<your token>` |
| `nsed.yaml` in pwd | see [demo template](https://github.com/peeramid-labs/nsed/blob/dev/examples/demo/nsed.yaml) |

If any are missing the skill surfaces the exact fix and refuses to
proceed — no silent failures.

## What it does

1. Verifies prerequisites
2. Runs `quorum run --room <room>` against the orchestrator named in
   `nsed.yaml`
3. Streams agent activity (`-> CortexA propose`, `<- CortexA proposed`,
   etc.) back to the user
4. Surfaces the converged `verdict.md` plus a footer pointing at the
   per-round trace

## Source

Authored in the [nsed](https://github.com/peeramid-labs/nsed) repo
at `examples/demo/plugin/`; this repository is a distribution mirror
for `claude plugin install`.
