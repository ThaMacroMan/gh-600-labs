# GH-600 Study Guide — GitHub Certified: Agentic AI Developer (beta)

**Exam:** GH-600 · 120 minutes · proctored · pass mark **700/1000** · maintained by GitHub, delivered via Pearson VUE/Microsoft.
**Beta note:** results released ~8 weeks after the beta period closes. Mostly GA features; some commonly-used Preview features may appear.
**Sandbox to practice the UI:** https://ghcertdemo.starttest.com/

This guide is organized by the six exam domains. Each section ends with **exam traps** — the distinctions the test likes to probe.

---

## Mental model for the whole exam

Everything maps to one idea: **GitHub is the control plane.** Agents (primarily **GitHub Copilot** — the cloud/coding agent and custom agents) *propose* work; **GitHub's native mechanisms** (branches, pull requests, required checks, CODEOWNERS, rulesets/branch protection, environments, workflow permissions, hooks) *gate and audit* it. When a question gives you a scenario, ask: **"Which GitHub-native control enforces this?"** The answer is almost never "tell the agent in its instructions" — instructions are *guidance*; allowlists, gates, checks, and permissions are *enforcement*.

> **The single most repeated principle:** *Treat "instructions not to do X" as guidance; treat tool allowlists, required checks, permissions, and gates as enforcement.*

---

## Domain 1 — Prepare agent architecture & SDLC processes (15–20%)

### 1.1 Integrate agents into the SDLC
- Identify **discrete, well-scoped steps** an agent should perform (e.g., "open a PR that fixes a failing test"), not open-ended goals.
- Define **inputs, outputs, and success criteria** for every agent task. Success criteria must be **verifiable** (checks pass, scans clean), not vague ("make it better").
- **Anti-patterns to recognize and mitigate:** unbounded scope, mixing planning with execution, no success criteria, agent merging its own work, bypassing review, no inspectable artifacts, over-broad tool/permission grants.

### 1.2 Separate planning, reasoning, and execution
Reliable systems split three things so reviewers can validate intent *before* impact:

| Phase | What it is | Where it lives in GitHub |
|---|---|---|
| **Planning** | What will be done and why | PR description, issue comment, or `pull_request_template.md` |
| **Execution** | Concrete changes | Commits on a branch |
| **Validation** | Evidence the outcome met success criteria | Checks, scans, artifacts, review outcomes |

**Two workflow shapes:**
- **Plan-first PR** — agent opens a PR containing *only the plan*; humans approve the plan, then the agent implements. **Use for high-risk work** (infra, auth, `.github/workflows/`, production). Reduces *early* exposure.
- **Plan + execution PR** — plan (in description) + code commits in the same PR. **Use for low/medium-risk, fast-iteration work.** Code appears before the plan is validated, but still can't merge without approval.

> Key insight: the choice is **not** *whether* work is reviewed (it always is) — it's *when code is allowed to be generated relative to human validation*.

**Enforce planning boundaries (enforcement, not vibes):**
1. **Capability boundary** — planning/review agents get **read-only tools**.
2. **Explicit handoff** — execution happens only after plan approval, via a deliberate transition to an implementation agent.
3. **Tool gating in orchestrators** — run planning with tools disabled; enable tools only after the plan is accepted.
4. **"Plan mode"** — interfaces that generate a plan artifact and pause before applying changes.

### 1.3 Observability & control for autonomous agents
- Plan the **degree of autonomy** explicitly with guardrails (see the risk table in Domain 5/6).
- Agents must produce **inspectable artifacts inside standard tooling** (PRs, commits, workflow runs, checks, uploaded artifacts).
- Design **human intervention that doesn't slow delivery** — gate only the actions that materially reduce risk.

**Exam traps (D1):**
- "Tell the agent in its instructions not to touch `/infra`" is a *weak* answer. Correct = CODEOWNERS + required reviews + rulesets on that path.
- Success criteria must be *verifiable/automated*, not subjective.
- Plan-first vs plan+execution → pick based on **risk/reversibility**, not speed alone.

---

## Domain 2 — Implement tool use & environment interaction (20–25%) — *highest weight*

### 2.1 Select & configure agent tools
- Identify the **minimum required tools** for the task; prefer **allowlists** over wildcards.
- Configure **tool permissions** per agent: **read-only toolsets for planning/review agents; implementation tools only for execution agents.**
- Changes to tool allowlists are a **governance-sensitive change** — review them like a high-risk dependency.

### 2.2 Custom agents (`.github/agents/*.agent.md`)
A custom agent file refines scope *within* a repository:
```
applyTo:
  - '**/*.js'
  - 'src/auth/**'
tools:
  - read_file
  - search_files
```
- `applyTo` → which files/dirs the agent focuses on.
- `tools` → which actions it can perform.
- `instructions` → how it behaves.
- **Custom agents DO NOT control** branch creation, PR behavior, or execution isolation — those are enforced by the **execution context**, not the agent file.

### 2.3 Model Context Protocol (MCP)
**MCP = a standard way for AI clients to connect to tools/services via MCP servers.** It gives agents a consistent model to discover tools, send structured requests, receive structured results, and reuse the pattern across systems.

- **MCP server** — exposes tools; sits between client and the real system. Can run **local** (your machine, access to local resources, usually no external auth) or **remote** (hosted, accessed over network, easier to share). The **GitHub MCP server** connects clients to repos, issues, PRs.
  - Remote is the **recommended** default in supported IDEs. GitHub Enterprise Server = local only; GitHub Enterprise Cloud with data residency = both.
  - Add in VS Code: Copilot icon → Copilot Chat → **Agent mode** → Tools icon → **Configure tools** → **Add MCP server** → choose **HTTP**, enter URL (e.g., `https://api.githubcopilot.com/mcp/`), pick scope, **Authenticate**.
  - Local example endpoint: `http://localhost:3000`.
- **MCP registry** — a **catalog of MCP servers** for discovery. Default = **GitHub MCP Registry**; orgs/enterprises can host a **custom registry** (fork/self-host the open-source registry, run via Docker, or build your own; or use **Azure API Center** as a managed registry).
  - Custom registry must follow **MCP registry v0.1 spec**, expose `GET /v0.1/servers`, `GET /v0.1/servers/{serverName}/versions/latest`, `GET /v0.1/servers/{serverName}/versions/{version}`, and include CORS headers (`Access-Control-Allow-Origin: *`, methods `GET, OPTIONS`, headers `Authorization, Content-Type`).
  - Azure API Center: enable anonymous access; enter the **base URL only** (do *not* append `/v0.1/servers`).
- **Allow list** — policy controlling **which MCP servers are permitted**, enforced at **org/enterprise** level (tied to the Copilot seat).
  - Enterprise: enterprise → **AI controls** → **MCP** → set "MCP servers in Copilot" Enabled → set Registry URL → **Restrict MCP access to registry servers**: **Allow all** vs **Registry only**.
  - Org: Org **Settings → Copilot → Policies** → enable MCP → optional Registry URL → Allow all / Registry only.
  - **Registry only** ⇒ even **local** servers must be listed in the registry with **exactly matching server IDs**. Policy applies **immediately** to all developers.

**How the three fit together:** server *exposes* tools → registry makes them *discoverable/trustable* → allow list decides what is *permitted*.

### 2.4 Integrate agents within dev environments / execution context
**Execution context** = the constraints defining *where* an agent operates and *what* it can access: **repository, branch, workflow, permissions.**
- **Repository scope** = first boundary; agent only reads/writes the repo it's enabled in (Settings → Copilot → Cloud agent → enable per repo). No cross-repo access unless granted.
- **Branch-based isolation** = agents never work on `main` directly; they create a branch, change it, open a PR to a base branch. For Copilot cloud agent you can select a **base branch** when delegating. Branch behavior is enforced by the **execution context/system**, not the custom agent.
- **Workflow boundaries** = execution happens inside GitHub Actions workflows = clean, repeatable, logged containers; also how agents run in **CI**.
- **Permission boundaries** = workflows get permissions via tokens (`GITHUB_TOKEN`). Always set explicitly and minimize.

**Invoke an agent in CI (pattern to recognize):**
```yml
# .github/workflows/agent-task.yml
on:
  workflow_dispatch:
  schedule:
    - cron: '0 9 * * *'
permissions:
  contents: read
jobs:
  agent-task:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '18' }
      - env:
          COPILOT_GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: npx @github/copilot-cli -p "Summarize recent changes" --no-ask-user
      # use a custom agent:
      - run: npx @github/copilot-cli --agent security-reviewer -p "Review for vulns" --no-ask-user
```

### 2.5 Safe execution paths & robust error handling
- **Error handling:** fail fast (`continue-on-error: false`), log meaningful errors, prevent partial/inconsistent changes.
- **Retries:** for *transient* failures (network/flaky) — rerun steps or loop with backoff:
  ```
  for i in 1 2 3; do npx @github/copilot-cli -p "Run task" && break; sleep 5; done
  ```
- **Rollbacks:** branch isolation + PR review = natural rollback; also close/discard PR or revert commits if merged.
- **Escalation paths:** require review, auto-assign reviewers, notify maintainers when the agent is stuck/uncertain.
- **Traceability/accountability:** workflow logs, commit history, PR discussions; clear commit messages, scoped branches, review everything.

**Exam traps (D2):**
- **Retries** = transient errors only; **escalation** = repeated/persistent failures or uncertainty. Don't retry forever.
- `GITHUB_TOKEN`-triggered actions usually **don't** spawn new workflow runs (loop protection); **GitHub App tokens / PATs can** — design to avoid recursion.
- Custom agent file ≠ branch/PR control. Allowlist (capability) ≠ branch scope (execution context).
- "Registry only" forces local servers into the registry with exact IDs.

---

## Domain 3 — Manage memory, state, and execution (10–15%)

### 3.1 Memory strategies — choose the right type
- **Short-term (working) memory** — within a single task/session; transient context.
- **Long-term memory** — persists across sessions (e.g., persistent instructions, learned conventions).
- **External memory** — stored outside the agent (files, issues, artifacts, datastores) and retrieved as needed.
- **Scope memory to task-relevant information** — don't carry irrelevant context; it causes drift and cost.
- Define **expiration, pruning, and reset rules** so stale context is discarded.

### 3.2 Persist state & manage context drift
- Capture **task progress and decisions as durable artifacts** (PR description, commits, uploaded workflow artifacts, issue comments) so work can **resume without repeating steps or diverging** from prior decisions.
- **Context drift** = the agent gradually loses alignment with the original goal/decisions during long runs. Detect it (compare current actions vs the recorded plan) and correct it (re-anchor to the durable plan artifact).

### 3.3 Continuity across tools/environments
- **Share state** through durable, inspectable artifacts rather than hidden in-process memory.
- **Prevent conflicting context** (two sources disagreeing) and **stale context** (outdated info) — single source of truth, expiration rules.
- Make evidence **traceable to a specific workflow run and commit** ("which run produced this, against what code state?").

**Exam traps (D3):**
- Durable artifacts (PRs/commits/uploaded artifacts) are the GitHub-native way to persist state — *not* relying on the model "remembering."
- Drift correction = re-anchor to recorded plan; resumption must not repeat completed steps.

---

## Domain 4 — Perform evaluation, error analysis, and tuning (15–20%)

### 4.1 Success criteria & evaluation signals
- Specify **expected outcomes and operational constraints** up front.
- Use both **qualitative** (review feedback, readability) and **quantitative** (tests passed, coverage, scan findings, latency) signals.
- **Align evaluation criteria with development intent** — measure what the task was actually meant to achieve.
- Generate signals with **automated scanning tools** (CodeQL/code scanning, secret scanning, dependency review, test/CI checks).

### 4.2 Analyze failures & find root cause
Use **logs, plans, traces, outputs, and workflow artifacts** to identify failures, then classify the **root cause**:
- **Reasoning errors** — flawed plan/logic, wrong approach.
- **Tool misuse** — wrong tool, wrong args, missing/over-broad tool access.
- **Context/environment issues** — stale/conflicting context, missing permissions, environment constraints, missing dependencies.

### 4.3 Tune behavior based on results
Levers, in order of "enforcement strength":
- **Revise instructions/workflows/constraints** (custom instructions, agent files, required checks).
- **Refine memory usage** (scope, prune, reset).
- **Refine tool usage and tool access** (tighten or adjust the allowlist).

**Exam traps (D4):**
- Match the **fix to the root cause**: tool misuse → adjust tool access/instructions; context issue → fix memory/permissions/env; reasoning error → revise plan/instructions.
- Evaluation signals should be **automated and tied to intent**, surfaced as GitHub artifacts (check runs, scan results).

---

## Domain 5 — Orchestrate multi-agent coordination (15–20%)

### 5.1 Operate & manage multi-agent workflows
- Apply an **orchestration pattern** to coordinate agents (e.g., planner→implementer→reviewer handoffs; supervisor/sub-agent delegation).
- **Isolate agents for parallel execution** — separate branches/worktrees/PRs so they don't collide.
- **Detect & resolve conflicts:** overlapping code changes, duplicated effort, contradictory outputs. Resolution often = isolation + clear ownership + merge/review gates.

### 5.2 Observability for multi-agent behavior
- Produce **artifacts suitable for review and audit** for each agent.
- **Document key decisions, handoffs, and outcomes across agents.**
- Enable **post-hoc analysis** — reconstruct what each agent did and why from artifacts/logs.

### 5.3 Detect & respond to multi-agent failures
- Identify **failed, partial, or stalled** executions.
- Respond to **degraded behavior or coordination breakdown**.
- **Recovery patterns:** rollback and **human-in-the-loop**.

### 5.4 Lifecycle of agents in multi-agent workflows
- **Add** agents without disrupting active workflows.
- **Update/reconfigure/replace** an agent without breaking the flow.
- **Retire** agents while **preserving auditability and continuity**.

**Delegation boundary recap (from agent profile config):**
- **Capability boundary** = which tools are allowed (prefer allowlists).
- **Visibility boundary** = whether the agent is user-selectable in interactive UI.
- **Delegation boundary** = which subagents can be invoked and how handoffs occur.

**Exam traps (D5):**
- Parallel agents → **isolation** (separate branches) is the primary conflict-avoidance mechanism.
- Recovery for coordination failure = rollback + human-in-the-loop, not "let it retry."
- Retiring an agent must **preserve audit trail** — don't delete history.

---

## Domain 6 — Implement guardrails and accountability (10–15%)

### 6.1 Define autonomy levels (right-size human intervention by risk)
Classify actions by **operational, security, and compliance risk**, then assign autonomy that maximizes speed while staying compliant with org security + Responsible AI standards:

| Task type | Example paths | Risk | Autonomy design |
|---|---|---|---|
| Docs/formatting | `docs/` | **Low** | automerge after required checks (and reviews) pass |
| App code, deps | `src/`, dependency bumps | **Medium** | PR + checks + ≥1 review |
| Infra, workflows | `infra/`, `.github/workflows/` | **High** | CODEOWNERS + multiple reviews + stricter rulesets |
| Prod deploys, secrets/settings | production, secrets | **Critical** | environment approvals; agent **prepares but can't execute** |

### 6.2 Guardrails & human-in-the-loop (enforcement mechanisms)
- **Identify the subset of actions that require human judgment** and gate exactly those.
- **Block actions that violate security/compliance/Responsible AI policy.**
- **Least privilege:** scope permissions and execution contexts.
  ```yml
  permissions:
    contents: read
    pull-requests: write   # read repo, update PR context; no broad write
  ```
- **Require explicit authorization / controlled paths** for **irreversible or compliance-sensitive** changes (environment approvals on protected deploys/secrets).
- **Preserve velocity** — don't add approvals that don't materially reduce risk.

### 6.3 PR governance controls (the enforcement toolkit)
- **PR template** (`.github/pull_request_template.md`) requires a structured **Plan / Evidence / Review checklist** on every agent PR.
- **Required status checks** turn "include a plan" into a guarantee — e.g., a **Plan Gate** workflow that fails if the plan artifact is missing; an admin marks it **required** via rulesets/branch protection.
- **CODEOWNERS** auto-routes sensitive paths to the right reviewers:
  ```text
  /security/            @security-team
  /.github/workflows/   @platform-team
  /infra/               @platform-team
  *                     @core-team
  ```
- **Environments with required reviewers** pause a job until approval — the agent prepares, a human approves prod.

### 6.4 Hooks (enforcement independent of the model)
Config files (e.g., `.github/hooks/`) that run custom commands at defined points:
```json
{
  "name": "block-high-risk-command",
  "trigger": "pre-tool-use",
  "run": "if [[ \"$TOOL\" == \"delete\" ]]; then echo 'Blocked'; exit 1; fi"
}
```
- **Pre-action** = validate/block unsafe actions before they run.
- **Post-action** = log usage/outputs for audit.
- **Error** = capture failures, trigger escalation/alerting.
- Key point: hooks enforce rules **regardless of the model's reasoning** — guidance in instructions is not enforcement; hooks are.

### 6.5 Secrets & GitHub Agentic Workflows defense-in-depth
- **Never** put secrets in instructions files, committed config, or workflow YAML in plaintext. Use protected secret boundaries injected at runtime; scope by environment; pass only to components that need them. The agent's runtime has its **own** secret boundary — it does **not** auto-inherit repo CI secrets.
- **GitHub Agentic Workflows** ship with: **read-only tokens by default**, **safe outputs** (agent proposes; a separate gated step decides), **zero secrets in the agent process**, **sandboxed/containerized execution**, **network isolation + allowlisted outbound**, **threat detection scanning proposed outputs before writes**. Copilot cloud agent also uses a **configurable firewall** to limit external access.

**Exam traps (D6):**
- "Agent prepares but can't execute" = the hallmark of **Critical** autonomy (environment approvals).
- Hooks/allowlists/required checks = enforcement; instructions = guidance. The exam loves this contrast.
- Least privilege default = restrict, elevate only where needed.

---

## High-yield one-liners (last-minute review)
- GitHub = control plane; agents propose, GitHub gates.
- PRs are **architectural control points**, not just collaboration.
- Read-only tools for planners/reviewers; write tools only for executors.
- `applyTo` + `tools` scope a custom agent's *focus/capability*, **not** branch/PR behavior.
- Repository scope → branch isolation → workflow → permissions = layered execution context.
- MCP: server exposes, registry discovers, allow list permits. Registry-only ⇒ local servers need exact IDs in registry.
- Remote MCP server is the recommended default; GHES = local only.
- `GITHUB_TOKEN` actions don't re-trigger workflows; PATs/App tokens can.
- Retry transient; escalate persistent. Same check fails twice → escalate with what failed/attempted/evidence/next step.
- Durable artifacts persist state and combat drift; trace evidence to run + commit.
- Match tuning to root cause: reasoning / tool misuse / context-env.
- Parallel agents need isolation; recovery = rollback + human-in-the-loop; retiring preserves audit.
- Critical actions: agent prepares, human approves via environments.
- Secrets never in repo content; agent runtime has its own secret boundary.

---

## Sources
- [Study guide for Exam GH-600](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-600)
- [GitHub Certified: Agentic AI Developer (beta) — certification page](https://learn.microsoft.com/en-us/credentials/certifications/agentic-ai-developer/)
- [Designing Agent Architecture and SDLC Integration (module)](https://learn.microsoft.com/en-us/training/modules/design-agent-architecture-integration/)
- [Tooling, MCP, and Agent Execution Environments (module)](https://learn.microsoft.com/en-us/training/modules/agent-tooling-mcp-execution-environments/)
- [Foundations of Agentic AI in GitHub (module)](https://learn.microsoft.com/en-us/training/modules/foundations-agentic-ai/)
