# Lab 5 — Hooks Explainer
## Domain 6: Implement Guardrails and Accountability

## The core principle

> **Hooks enforce rules regardless of the model's reasoning.**
> Instructions/system prompts are guidance. Hooks are enforcement.

This is one of the most-tested concepts on GH-600.

---

## Three trigger types (know all three)

| Trigger | When it fires | Use for |
|---------|--------------|---------|
| `pre-tool-use` | Before a tool executes | Block unsafe actions |
| `post-tool-use` | After a tool executes | Audit/log tool usage |
| `error` | When a tool fails | Escalation, alerting |

---

## How blocking works

A hook blocks an action by **exiting non-zero** (`exit 1`).

```bash
# This BLOCKS the tool call:
echo 'BLOCKED: Destructive tool call denied'; exit 1

# This ALLOWS the tool call:
exit 0  # or just don't exit non-zero
```

The model's reasoning is irrelevant — if the hook exits non-zero, the action is blocked.

---

## Exam question patterns

**Q: How do you prevent an agent from calling a `delete` tool regardless of what the model reasons?**  
A: A `pre-tool-use` hook that checks `$TOOL == "delete"` and exits non-zero.

**Q: What's the difference between telling an agent "never delete files" vs a hook?**  
A: Instructions are guidance (model can ignore/misinterpret). Hook is enforcement (fires independently).

**Q: A hook fires after every tool call and logs the output. What trigger type is this?**  
A: `post-tool-use`

**Q: You want to capture tool failures and auto-post a comment on the PR. What trigger?**  
A: `error`

---

## Connection to autonomy levels

Hooks are the **enforcement layer** for Responsible AI standards:

```
Instructions/system prompt    →  agent SHOULD not do X      (guidance)
pre-tool-use hook             →  agent CANNOT do X          (enforcement)
```

For critical actions (production, secrets), hooks + environment approvals = the enforcement stack.
