# Lab 1 Cheat Sheet — Custom Agents (`.github/agents/*.agent.md`)

**Exam domains:** D1 (architecture/SDLC) · D2 (tool use)

---

## Mental model

Custom agent files **scope capability and focus** within a repo. They do **not** control branch behavior, PR creation, or `GITHUB_TOKEN` permissions — that's the **execution context**.

> **Instructions = guidance. Tool allowlists = enforcement.**

---

## Agent file anatomy

| Field | Enforces? | Purpose |
|-------|-----------|---------|
| `name` | No | Display name / invocation |
| `description` | No | Purpose for humans + agent picker |
| `applyTo` | Partially | Which files/dirs the agent **focuses on** |
| `tools` | **Yes** | **Capability boundary** — hard allowlist |
| `instructions` | No | Behavior guidance only |

---

## Tool aliases (know the concepts)

| Alias | Purpose |
|-------|---------|
| `read` / `read_file` | Read files |
| `search` / `search_files` | Search files/text |
| `edit` / `Write` | Create/modify files |
| `execute` / `shell` | Run terminal commands |
| `agent` | Invoke another custom agent |

- **Omit `tools`** → ALL tools enabled (over-broad — exam trap)
- **`tools: []`** → no tools (forced read-only)
- **Explicit allowlist** → always prefer minimum required tools

---

## Agent roles in this lab

| Agent | Tools | Output |
|-------|-------|--------|
| **planner** | read-only | Structured plan (Goal, Steps, Success criteria, Risks) |
| **security-reviewer** | read-only | CRITICAL / WARNING / INFO report |
| **implementer** (concept) | read + **edit** | Commits on branch after plan approval |

---

## Plan-first vs plan+execution

| Pattern | When | What happens |
|---------|------|--------------|
| **Plan-first** | High risk (infra, auth, `.github/workflows/`, prod) | Plan only → human approves → then implementer runs |
| **Plan+execution** | Low/medium risk | Plan in PR description + code in same PR |

Choice is based on **risk/reversibility**, not speed alone.

---

## Multi-agent handoff

```
Human task → PLANNER (read-only) → human approves plan
          → IMPLEMENTER (edit tools) → branch + PR
          → SECURITY-REVIEWER (read-only) → review report
          → human merges (checks + reviews pass)
```

---

## Where phases live in GitHub

| Phase | Location |
|-------|----------|
| **Planning** | PR description, issue comment |
| **Execution** | Commits on branch |
| **Validation** | Checks, scans, artifacts, review outcomes |

Success criteria must be **verifiable/automated** — not "make it cleaner."

---

## Exam traps

| Wrong | Correct |
|-------|---------|
| Give planner write tools to draft code | Planner = read-only; separate implementer after approval |
| Tell agent "don't touch `/infra/`" | CODEOWNERS + ruleset + required reviews |
| Agent file prevents pushes to `main` | Branch rulesets / execution context |
| Omit `tools` for simplicity | Explicit minimum allowlist |
| `applyTo` blocks PRs or branch behavior | `applyTo` = focus only; enforcement = GitHub controls |

---

## One-liners

- Read-only tools for planners/reviewers; write tools only for executors.
- `applyTo` + `tools` scope **focus/capability**, not branch/PR behavior.
- Durable plan artifact = PR description — not agent memory or chat history.
