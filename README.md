# quorum-claude-plugin

Claude Code plugin that adds the `/quorum:brainstorm` slash command —
submits a topic to a multi-agent deliberation against a
[quorum](https://github.com/peeramid-labs/quorum-rs) orchestrator and
streams the converged answer back into Claude Code.

---

## Install (read carefully — the install lines are typed INSIDE Claude Code, not in a terminal)

### Step 1 — open Claude Code

In your terminal:

```bash
claude
```

You should now see the Claude Code prompt — the `>` cursor where you
normally type questions.

### Step 2 — register this repo as a plugin marketplace

At the **Claude Code prompt** (not your shell), type:

```
/plugin marketplace add peeramid-labs/quorum-claude-plugin
```

The leading `/` is what makes Claude Code intercept this — it's a
built-in slash command, not a message to the model.

### Step 3 — install the plugin from that marketplace

Still at the **Claude Code prompt**:

```
/plugin install quorum@quorum-marketplace
```

`quorum` is the plugin name (defined in `.claude-plugin/plugin.json`
in this repo); `quorum-marketplace` is the marketplace name (defined
in `.claude-plugin/marketplace.json`).

You'll see Claude Code confirm `Plugin "quorum" installed`. No restart
needed.

### Step 4 — use it

Still at the **Claude Code prompt**:

```
/quorum:brainstorm Should we adopt elliptic-curve identifiers for agent attestation?
```

(or whatever topic you want the agent team to deliberate on)

The plugin runs prereq checks, dispatches the deliberation via the
local `quorum` CLI, streams agent activity as it arrives, and surfaces
the converged verdict.

---

## Why two slash commands (not one)?

Claude Code doesn't install plugins from raw GitHub URLs. It installs
them from a **marketplace** — a JSON registry that lists one or more
plugins. This repo ships both the marketplace registry and the plugin
itself, so step 2 (`marketplace add`) and step 3 (`plugin install`) point
at the same repo. Two commands; one repo.

`claude plugin install peeramid-labs/quorum-claude-plugin` typed at a
**terminal** will NOT work — that's not a real shell command. The
install machinery is inside Claude Code; the slash-command flow above
is the only supported install path today.

---

## Prerequisites

The skill expects, in your shell environment:

| Requirement | How to install / set |
|---|---|
| `quorum` CLI ≥ 0.6.0 | `cargo install --locked --git https://github.com/peeramid-labs/quorum-rs --branch dev quorum-rs` |
| `QUORUM_DEMO_TOKEN` env var | `export QUORUM_DEMO_TOKEN=<token from the demo handout>` |
| `nsed.yaml` in pwd | `curl -fsSL https://raw.githubusercontent.com/peeramid-labs/nsed/demo/zoom-for-ai/examples/demo/nsed.yaml > nsed.yaml` |

**Note on the `quorum` install:** the crates.io `quorum-cli` package
belongs to a different project and is NOT what this skill expects.
The git-install above pulls v0.6.0+ from the canonical repo's default
branch (`peeramid-labs/quorum-rs`), which ships the `quorum` binary as
part of the `quorum-rs` crate. Once a release tag publishes
`quorum-rs` to crates.io, the simpler `cargo install quorum-rs` will
work too. Rust toolchain required (`rustup install stable` if you
don't have one).

If any prerequisite is missing the skill surfaces the exact fix and
refuses to proceed — no silent failures.

---

## Uninstall

At the Claude Code prompt:

```
/plugin uninstall quorum
/plugin marketplace remove quorum-marketplace
```

---

## What the plugin does internally

1. Verifies the four prerequisites above
2. Runs `quorum run --room <room>` against the orchestrator named in
   `nsed.yaml`
3. Streams agent activity (`-> CortexA propose`, `<- CortexA proposed`,
   etc.) back to the user as it arrives
4. Reads `verdict.md` from the output dir and presents it, with a
   footer pointing at `quorum trace <job_id>` for per-round drill-down

The full skill prompt is at `skills/brainstorm/SKILL.md` in this repo.

---

## Source

Authored in the [nsed](https://github.com/peeramid-labs/nsed) repo at
`examples/demo/plugin/`; this repository is a distribution mirror.
File issues / PRs against the source repo.
