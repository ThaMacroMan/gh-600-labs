# Lab 2 Cheat Sheet — Agent in CI (GitHub Actions)

**Exam domain:** D2 (tool use & execution context)

---

## Mental model

Lab 1 **defines** agents (`.github/agents/`). Lab 2 **runs** them in CI via `copilot-cli` inside a workflow — with least privilege, loop-safe tokens, and durable artifacts.

---

## Workflow pattern (this lab)

```yaml
on:
  schedule:
    - cron: '0 9 * * *'    # nightly automation
  workflow_dispatch:        # manual trigger for testing

permissions:
  contents: read            # checkout + read repo
  pull-requests: write      # open/update PRs — NOT contents: write

env:
  COPILOT_GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

run: |
  npx @github/copilot-cli \
    --agent security-reviewer \
    -p "Your prompt here" \
    --no-ask-user
```

---

## Key concepts

| Topic | Remember |
|-------|----------|
| **`schedule` + `workflow_dispatch`** | Automate nightly + manual test runs |
| **Explicit `permissions`** | Always declare minimum — never `write-all` |
| **`GITHUB_TOKEN`** | Auto-injected, scoped to repo, **loop-safe** (won't spawn new workflow runs) |
| **PAT / App token** | Can trigger new runs — use only when cross-repo or re-trigger needed |
| **`--no-ask-user`** | Non-interactive — required for CI (no human to answer prompts) |
| **`--agent name`** | Invokes custom agent from `.github/agents/name.agent.md` |
| **Artifacts + `if: always()`** | Upload output even on failure — traceable to `github.run_id` |

---

## Permissions cheat sheet

| Need | Grant |
|------|-------|
| Read repo | `contents: read` |
| Open/update PRs | `pull-requests: write` |
| Push commits to branch | `contents: write` (only if needed) |
| Everything | **Never** `write-all` on exam |

---

## GITHUB_TOKEN vs PAT

| | `GITHUB_TOKEN` | PAT / App token |
|---|----------------|-----------------|
| Stored in repo? | No — injected at runtime | Yes — in secrets |
| Loop prevention | Yes — generally won't re-trigger workflows | Can trigger new runs |
| Scope | This repo + your `permissions:` block | Whatever scopes you grant |
| Default choice | ✓ Yes | Only when needed |

---

## Artifacts (Domain 3 tie-in)

- Makes agent output **inspectable** and **durable**
- Name with `github.run_id` → traceable to specific run + commit state
- `if: always()` → preserve logs on failure (critical for root-cause analysis)

---

## Error handling in CI

| Failure type | Response |
|--------------|----------|
| Transient (network timeout) | Bounded retries (e.g. 3 attempts + sleep) |
| Same check fails twice | **Escalate** to human with evidence — don't retry forever |
| Persistent / uncertain | Escalation, not infinite retry |

---

## Exam traps

| Wrong | Correct |
|-------|---------|
| `permissions: write-all` | Explicit minimal permissions |
| No `permissions` block | Always declare explicitly |
| PAT in README or committed YAML | `secrets.GITHUB_TOKEN` at runtime |
| Skip `--no-ask-user` in CI | Required — workflow would hang |
| Artifacts only on success | `if: always()` — failures need logs too |
| `--agent` creates agent at runtime | Loads existing `.github/agents/` profile |

---

## One-liners

- `schedule` cron + minimal `permissions` = nightly agent with least privilege.
- `GITHUB_TOKEN` = loop-safe; PATs can recurse.
- `--no-ask-user` = CI-safe, non-interactive.
- Artifacts trace evidence to run + commit.
