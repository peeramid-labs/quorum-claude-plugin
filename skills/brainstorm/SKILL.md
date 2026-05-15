---
name: brainstorm
description: "Submit a topic to a quorum of AI agents for collaborative deliberation. Multiple agents propose independently, evaluate each other's proposals, and converge on a quorum-backed verdict. Use when the user types `/brainstorm <topic>` or asks to 'ask the team', 'get a second opinion', or 'run this past the quorum'."
---

# brainstorm

Run a multi-agent deliberation against the configured quorum orchestrator. The user types `/brainstorm <topic>` (or asks for a "team opinion" / "second opinion") and you orchestrate the run, stream activity back, and surface the verdict.

## How this works

`quorum run` submits a task to a remote orchestrator. The orchestrator dispatches the task to a heterogeneous agent team — each agent independently proposes a solution, then evaluates peer proposals, and the protocol converges on a quorum-backed outcome. Convergence detection ends the round when agreement is high enough, so you don't pay for tokens past the point where the answer stabilises.

The user's machine is the **client** for this deliberation; the agents themselves run in the cloud. You're driving `quorum run` on the user's behalf and showing them the result.

## Steps to perform

### 1. Verify prerequisites

Run these checks in order. If any fail, surface the exact fix to the user and stop — do not proceed to step 2.

| Check | Command | If missing |
|---|---|---|
| `quorum` CLI installed | `command -v quorum` | Tell user: `cargo install quorum-cli` |
| `QUORUM_DEMO_TOKEN` env var set | `test -n "$QUORUM_DEMO_TOKEN"` | Ask user to `export QUORUM_DEMO_TOKEN=<token-from-demo-handout>` |
| `nsed.yaml` in working dir | `test -f nsed.yaml` | Fetch the demo template (it specifies the cloud agents the demo room dispatches to): `curl -fsSL https://raw.githubusercontent.com/peeramid-labs/nsed/main/examples/demo/nsed.yaml > nsed.yaml`. The generic `quorum init` writes a workspace without the `agents:` list and isn't suitable for the demo room — use this template. |
| Orchestrator reachable | `curl -fsSm 5 https://api.peeramid.xyz/health` | Network issue; tell user to check connectivity |

### 2. Submit the task

Pass the user's topic verbatim. Capture verdict + dispatch into a fresh per-invocation directory (so subsequent runs don't trip `quorum run`'s refuse-to-overwrite-`verdict.md` guard):

```bash
OUT="/tmp/quorum-runs/$(date +%Y%m%d-%H%M%S)-$$"
mkdir -p "$OUT"
quorum run --room demo --output-dir "$OUT" "${ARGUMENTS}"
```

(The `-$$` suffix appends the shell PID. Second-granular timestamps alone can collide on rapid back-to-back invocations; the PID makes each invocation's directory unique even within the same second. `$$` is portable; `date +%N` for nanoseconds isn't — BSD `date` on macOS doesn't support it.)

(Alternative: pass `--force-output` to allow overwriting a single fixed
directory. Per-invocation dirs are preferred because they preserve
prior artifacts and make `quorum trace`-style follow-ups easier.)

Stream the stdout/stderr back to the user **as it arrives** — `quorum run` prints lines like `Connected to orchestrator`, `Job submitted: <id>`, `  -> <agent_id> propose...`, `  <- <agent_id> proposed`, `  <- <evaluator_id> evaluated`, and `  !! ...` (timeouts/errors). Each of those is part of the show; relay them to the user with no buffering — the live tick is the demo's value.

### 3. Surface the verdict

When `quorum run` exits, the output dir (`$OUT` per step 2) contains:
- `verdict.md` — the winning agent's proposal text (this *is* the answer; format varies by which agent won and what prompt set they used)
- `dispatch.json` — the request that was sent (agents, policy, room, task)
- `prompt.txt` — the prompt that was sent

Read `verdict.md` and present it to the user verbatim, then add a short footer using this template:

```
_Verdict from <JOB_ID> · full artifacts at <OUTPUT_DIR>/ · for per-round drill-down: `quorum trace <JOB_ID>`_
```

Substitute the placeholders with values from the run you actually executed:
- `<JOB_ID>` — printed to stdout by `quorum run`; capture it from the streamed output (look for a line containing `job_id` or `Job:`).
- `<OUTPUT_DIR>` — whatever path you passed to `--output-dir` in step 2 (the example above used `/tmp/quorum-runs`; substitute whatever you actually used).

Do NOT try to call `quorum trace` from within the skill — it's a follow-up step the user runs themselves if they want per-round inspection.

Do not try to extract "stances" or "dissent" from `verdict.md` — it's the winning proposal text, not a structured trace. Stance / per-agent disagreement data lives in the deliberation trace served by `quorum trace`, not in this file.

### 4. Failure modes

| Symptom | What it means | What to do |
|---|---|---|
| `quorum run` exits non-zero with "401" / "403" | Token rejected by orchestrator | Surface the message; ask user to verify `QUORUM_DEMO_TOKEN` |
| Hangs > 90s without a verdict | Agent fleet stalled or orchestrator overloaded | After 90s, kill the run, tell user the team didn't converge, suggest re-running |
| `connection refused` / DNS error | Cloud orch unreachable | Tell user to check network, then `curl https://api.peeramid.xyz/health` |
| Empty `verdict.md` | Convergence not reached | Read `trace.json` instead — show the proposals + evaluations even without a verdict |

## What NOT to do

- **Don't** synthesise the answer yourself. The whole point of `/brainstorm` is that the *team of remote agents* produced the answer, not you. Pass through their verdict; don't paraphrase.
- **Don't** retry silently on failure. If the run fails, surface the failure with the exact error so the user understands what broke.
- **Don't** read or modify `nsed.yaml` — it's the user's workspace config, not something to edit per-task.

## Why this matters (for the demo arc)

Single LLMs make calls; a quorum of LLMs makes well-grounded decisions. Disagreement between proposers is *signal* — and gets surfaced in the verdict. The agents are heterogeneous (different models, different prompts), so the consensus emerges from genuinely independent perspectives, not just N copies of the same chain-of-thought. Token-efficient by design: convergence detection short-circuits the round the moment agreement is high enough.

ARGUMENTS: <topic to brainstorm>
