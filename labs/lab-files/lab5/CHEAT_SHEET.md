# Lab 5 Cheat Sheet — Hooks (`.github/hooks/`)

**Exam domain:** D6 (guardrails & accountability)

---

## Mental model

> **Hooks enforce rules regardless of the model's reasoning.**
> **Instructions = guidance. Hooks = enforcement.**

This is one of the most-tested GH-600 concepts.

---

## Hook config location

`.github/hooks/*.json` — runs custom commands at defined lifecycle points.

---

## Three trigger types (memorize all)

| Trigger | When it fires | Use for |
|---------|--------------|---------|
| **`pre-tool-use`** | Before tool executes | **Block** unsafe actions |
| **`post-tool-use`** | After tool executes | Audit / log tool usage |
| **`error`** | When tool fails | Escalation, alerting |

---

## How blocking works

```bash
# BLOCKS the tool call:
echo 'BLOCKED: policy violation'; exit 1

# ALLOWS the tool call:
exit 0
```

Non-zero exit = action blocked. Model reasoning is **irrelevant**.

---

## This lab's hooks

| Hook | Trigger | What it does |
|------|---------|--------------|
| `block-dangerous-tools` | `pre-tool-use` | Blocks `delete_file`, `delete_directory`, `drop_table` |
| `block-production-writes` | `pre-tool-use` | Blocks writes to `config/production` paths |
| `audit-all-tool-calls` | `post-tool-use` | Logs every tool call for traceability |
| `escalate-on-tool-error` | `error` | Logs failure + posts PR comment for human review |

---

## Enforcement stack (layered)

```
Instructions/system prompt  →  agent SHOULD not do X     (guidance)
Tool allowlist in agent     →  agent lacks capability    (capability boundary)
pre-tool-use hook           →  agent CANNOT do X         (enforcement)
CODEOWNERS + ruleset        →  merge blocked without review
Environment approval        →  critical actions gated    (agent prepares, human approves)
```

---

## Hooks vs other controls

| Control | Blocks at | Independent of model? |
|---------|-----------|----------------------|
| Instructions | — | No |
| Tool allowlist | Capability | Partially |
| **pre-tool-use hook** | Action time | **Yes** |
| CODEOWNERS | Merge time | Yes |
| Environment approval | Deploy time | Yes |

---

## Exam question patterns

| Question | Answer |
|----------|--------|
| Prevent `delete` tool regardless of model reasoning? | `pre-tool-use` hook, check `$TOOL`, `exit 1` |
| "Never delete files" in instructions vs hook? | Instructions = guidance; hook = enforcement |
| Log every tool call after execution? | `post-tool-use` |
| Capture failures + notify on PR? | `error` trigger |

---

## Exam traps

| Wrong | Correct |
|-------|---------|
| Rely on instructions to prevent destructive tools | `pre-tool-use` hook + tool allowlist |
| Hook runs after action to block it | `pre-tool-use` blocks **before** execution |
| Hooks replace CODEOWNERS | Layered defense — different enforcement points |

---

## One-liners

- pre = block/validate, post = audit, error = escalate.
- `exit 1` = blocked. Model can't override.
- Hooks + allowlists + gates = defense in depth.
