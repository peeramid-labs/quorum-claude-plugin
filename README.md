# quorum-claude-plugin

Claude Code plugin that adds `/quorum:brainstorm <topic>` — a slash
command that submits a topic to a multi-agent deliberation against a
[quorum](https://github.com/peeramid-labs/quorum-rs) orchestrator and
streams the converged answer back into Claude.

## Install

From within Claude Code (these are slash commands typed into the
Claude Code prompt, not shell commands):

```
/plugin marketplace add peeramid-labs/quorum-claude-plugin
/plugin install quorum@quorum-marketplace
```

Then invoke the skill:

```
/quorum:brainstorm Should we adopt elliptic-curve identifiers for agent attestation?
```

### Why two steps?

Claude Code only installs plugins from a *marketplace* (a JSON
registry that lists one or more plugins). This repo ships a
`marketplace.json` so the marketplace + plugin live together — adding
the marketplace also makes the `quorum` plugin available for install.

## Prerequisites

The skill expects, in your shell environment:

| Requirement | How to install / set |
|---|---|
| `quorum` CLI ≥ 0.6.0 | `cargo install --locked --git https://github.com/peeramid-labs/quorum-rs --branch dev quorum-cli` |
| `QUORUM_DEMO_TOKEN` env var | `export QUORUM_DEMO_TOKEN=<token from the demo handout>` |
| `nsed.yaml` in pwd | `curl -fsSL https://raw.githubusercontent.com/peeramid-labs/nsed/demo/zoom-for-ai/examples/demo/nsed.yaml > nsed.yaml` |

**Note on the `quorum` install:** crates.io currently has only the
stale 0.3.0 (missing `--output-dir` and current dispatch behavior). The
git-install above pulls 0.6.0 from the canonical repo's default
branch. Rust toolchain required (`rustup install stable` if you
don't have one).

If any prerequisite is missing the skill surfaces the exact fix and
refuses to proceed — no silent failures.

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
