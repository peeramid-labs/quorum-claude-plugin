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
