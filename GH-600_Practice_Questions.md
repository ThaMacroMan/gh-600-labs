# GH-600 Practice Questions

40 exam-style questions, weighted to the official domain percentages. Answers and explanations are at the bottom — cover them while you work. Scenario-style, single best answer unless noted.

**Domain weighting in this set:** D1 (7) · D2 (10) · D3 (5) · D4 (6) · D5 (7) · D6 (5).

---

## Domain 1 — Prepare agent architecture & SDLC processes

**1.** A team wants to ensure an agent never modifies files under `infra/` without sign-off from the platform team. Which approach *enforces* this?
A. Add "do not edit infra/" to the agent's custom instructions
B. Configure CODEOWNERS for `/infra/` plus required reviews via a ruleset
C. Ask reviewers to watch for infra changes during review
D. Put a comment in each infra file warning the agent

**2.** Which is the best example of a *verifiable* success criterion for an agent task?
A. "Improve the code quality of the module"
B. "Make the function cleaner and more readable"
C. "All required checks pass and code scanning reports no new high-severity alerts"
D. "Refactor until the reviewer is happy"

**3.** A task touches production authentication logic and is hard to reverse. Which workflow shape is preferred?
A. Plan + execution in the same PR for speed
B. Plan-first PR — approve the plan before any code is generated
C. Let the agent commit directly to a release branch
D. Skip planning; rely on post-merge monitoring

**4.** Which is an agent anti-pattern the exam expects you to mitigate?
A. Producing a structured plan in the PR description
B. Scoping the task to a single failing test
C. Allowing the agent to merge its own PR without review
D. Uploading test results as workflow artifacts

**5.** In the plan/execution/validation split, where does *validation* surface in GitHub? (choose the best)
A. In the agent's hidden chain-of-thought
B. In checks, scans, artifacts, and review outcomes
C. Only in the commit messages
D. In the repository description

**6.** A reviewer says: "I can only see the final diff; I can't tell what the agent intended." What design change fixes this?
A. Give the agent more tools
B. Separate planning from execution so intent is reviewable before impact
C. Increase the agent's autonomy level
D. Remove required checks to speed review

**7.** What is the correct way to enforce that a planning agent cannot modify files during planning?
A. Add a polite instruction asking it not to write files
B. Restrict the planning agent to read-only tools
C. Trust the model to follow the system prompt
D. Review the diff afterward

---

## Domain 2 — Implement tool use & environment interaction

**8.** An organization wants developers to use *only* MCP servers it has vetted. Which policy setting achieves this?
A. "Allow all" with a registry URL configured
B. Disable Copilot entirely
C. "Registry only" with the approved servers listed in the configured registry
D. Add the servers to each developer's local config manually

**9.** With "Registry only" enforced, what is required for a *local* MCP server to be usable?
A. Nothing — local servers are always exempt
B. It must be listed in the registry with an exactly matching server ID
C. It must be converted to a remote server
D. The developer must be an org admin

**10.** In a `.github/agents/reviewer.agent.md` file, what do `applyTo` and `tools` control?
A. Which branch the agent commits to and how PRs are opened
B. The files/dirs the agent focuses on and the actions it can perform
C. Whether the workflow is triggered on push
D. The GITHUB_TOKEN permissions for the repo

**11.** Which statement about branch behavior for a Copilot cloud agent is correct?
A. The custom agent file defines branch creation and PR behavior
B. The agent edits `main` directly, then a check reverts bad changes
C. Branch-based scope is enforced by the execution context, not the agent file
D. Agents cannot create branches at all

**12.** You want an agent task to run nightly in CI with least privilege. Which snippet is correct?
A. `permissions: write-all` on a `pull_request` trigger
B. `schedule: - cron: '0 9 * * *'` with `permissions: { contents: read }`
C. No `permissions` block and a PAT stored in the repo README
D. `on: push` with `contents: write` and secrets printed to logs

**13.** Which best describes how MCP servers, registries, and allow lists relate?
A. They are three names for the same feature
B. Server exposes tools; registry makes servers discoverable; allow list decides which are permitted
C. The registry blocks servers; the allow list exposes tools
D. Allow lists run locally; registries run in CI

**14.** A workflow is meant for PRs but also has `workflow_dispatch`. How do you stop PR-only steps from misfiring on manual runs?
A. Delete the `workflow_dispatch` trigger
B. Add a job-level condition `if: github.event_name == 'pull_request'`
C. Run everything and ignore failures
D. Use `continue-on-error: true` everywhere

**15.** An agent step fails intermittently due to network timeouts. The *most appropriate* first response is:
A. Immediately escalate to a human on the first failure
B. Implement bounded retries (e.g., 3 attempts with a short sleep)
C. Disable error handling so it continues
D. Merge anyway and monitor production

**16.** The same required check fails twice on an agent's PR. According to the safe-iteration policy, what should happen?
A. Keep retrying indefinitely until it passes
B. Escalate to a human with what failed, what was attempted, the evidence, and a suggested next step
C. Force-merge with admin override
D. Delete the branch and start over silently

**17.** Why does GitHub generally *not* spawn new workflow runs from actions performed with `GITHUB_TOKEN`?
A. To save Actions minutes only
B. To help prevent unintended recursive/loop executions
C. Because `GITHUB_TOKEN` cannot trigger anything ever
D. It's a bug, not a feature

**18.** A planning/review agent and an implementation agent share one repo. Which tool configuration matches best practice?
A. Both get full read/write tools
B. Planner = read-only tools; implementer = execution/write tools
C. Planner = write tools; implementer = read-only
D. Neither gets any tools

**19.** Which is true about remote vs local MCP servers in supported IDEs?
A. Local is the recommended default for most users
B. Remote is positioned as the recommended setup; GitHub Enterprise Server supports local configuration
C. GitHub Enterprise Server supports only remote servers
D. Local servers always require external authentication

**20.** Adding or expanding MCP tools for an agent should be treated as:
A. A trivial config tweak needing no review
B. A high-risk dependency change that increases blast radius and warrants review
C. Something only end users decide, never governed
D. Equivalent to changing a commit message

---

## Domain 3 — Manage memory, state, and execution

**21.** An agent must resume a long task after interruption without redoing finished steps. What enables this?
A. Relying on the model's built-in memory of the session
B. Capturing progress and decisions as durable artifacts (PR description, commits, uploaded artifacts)
C. Restarting from scratch each time
D. Storing state only in the agent's short-term context

**22.** During an extended run, the agent's actions start diverging from the approved plan. This is:
A. Tool misuse
B. Context drift — detect and re-anchor to the recorded plan
C. A permissions error
D. Normal and requires no action

**23.** Which memory type is best for information that must persist across multiple sessions?
A. Short-term/working memory
B. Long-term (or external) memory
C. No memory
D. The runner's `/tmp` directory

**24.** Two sources give the agent contradictory configuration values. The risk you're managing is:
A. Stale context
B. Conflicting context — establish a single source of truth
C. Retry storms
D. Over-privileged tokens

**25.** To make agent evidence auditable, each artifact should be traceable to:
A. A specific workflow run and the commit/code state that produced it
B. The reviewer's name only
C. The agent's model version only
D. Nothing — artifacts are self-explanatory

---

## Domain 4 — Perform evaluation, error analysis, and tuning

**26.** An agent called the wrong tool with malformed arguments. The root-cause class is:
A. Reasoning error
B. Tool misuse
C. Context/environment issue
D. Network failure

**27.** Which produces *automated, quantitative* evaluation signals aligned with code quality/security?
A. A reviewer's gut feeling
B. Code scanning (CodeQL), tests/CI checks, dependency review
C. The PR title
D. The number of commits

**28.** An agent repeatedly fails because a required dependency isn't available in its environment. Root cause:
A. Reasoning error
B. Context/environment issue
C. Tool misuse
D. Insufficient autonomy

**29.** After analysis shows the agent's *plan/approach* was flawed, the right tuning lever is:
A. Add more secrets
B. Revise the instructions/workflow/constraints (and plan)
C. Broaden the tool allowlist
D. Increase retries

**30.** Which artifacts are most useful for root-cause analysis of an agent failure?
A. Only the final merged code
B. Logs, plans, traces, outputs, and workflow artifacts
C. The repo star count
D. Slack reactions

**31.** Evaluation criteria should primarily be aligned with:
A. The agent's preferences
B. Development intent — what the task was meant to achieve
C. The longest possible runtime
D. Maximizing tool calls

---

## Domain 5 — Orchestrate multi-agent coordination

**32.** Two agents work in parallel on the same repo and produce overlapping edits. The primary mechanism to prevent collisions is:
A. Running both on `main`
B. Agent isolation — separate branches/worktrees and PRs
C. Disabling required checks
D. Giving both write-all permissions

**33.** A coordinated multi-agent workflow stalls partway. The recommended recovery pattern is:
A. Let it retry forever
B. Rollback and human-in-the-loop
C. Auto-merge partial work
D. Delete all artifacts and restart silently

**34.** You need to retire an agent from a multi-agent workflow. What must be preserved?
A. Nothing
B. Auditability and workflow continuity
C. Only the agent's secrets
D. The agent's short-term memory

**35.** For post-hoc analysis of multi-agent behavior, the workflow must:
A. Hide handoffs to reduce noise
B. Produce review/audit-suitable artifacts documenting decisions, handoffs, and outcomes
C. Store everything only in chat
D. Avoid logging to save space

**36.** In agent profile configuration, the **delegation boundary** controls:
A. Which tools are allowed
B. Whether the agent is user-selectable in the UI
C. Which subagents can be invoked and how handoffs occur
D. The branch the agent targets

**37.** You add a new agent to an active multi-agent workflow. The goal is to do so:
A. By pausing and rebuilding the entire workflow
B. Without disrupting the active workflow, preserving continuity
C. By deleting the existing agents first
D. Only by merging directly to main

**38.** Three agents emit contradictory outputs for the same task. This is an example of:
A. A conflict to detect and resolve (contradictory outputs)
B. Healthy redundancy needing no action
C. A permissions misconfiguration
D. A retry policy

---

## Domain 6 — Implement guardrails and accountability

**39.** A task deploys to production and accesses protected secrets. The correct autonomy design is:
A. Automerge after checks
B. PR + one review
C. Environment approvals — agent prepares the change but a human approves execution
D. Let the agent deploy directly with `write-all`

**40.** Which mechanism enforces a rule *regardless of the model's reasoning*, e.g., blocking a `delete` tool call before it runs?
A. A line in the system prompt
B. A `pre-tool-use` hook that inspects the action and exits non-zero to block it
C. A polite PR comment
D. A README note

---

# Answer Key & Explanations

**1. B** — CODEOWNERS + required reviews via rulesets *enforce* sign-off. Instructions/comments are guidance, not enforcement. *(D1)*

**2. C** — Verifiable = objective/automated (checks pass, no new high-severity scan alerts). A/B/D are subjective. *(D1)*

**3. B** — High-risk, hard-to-reverse work → **plan-first** so intent is validated before any code exists, reducing early exposure. *(D1)*

**4. C** — Self-merging without review removes the primary safety mechanism. The others are good practices. *(D1)*

**5. B** — Validation = checks, scans, artifacts, review outcomes. Planning lives in the PR description/issue; execution in commits. *(D1)*

**6. B** — Separating planning from execution makes intent reviewable *before* impact. *(D1)*

**7. B** — Read-only tools are *enforcement*; instructions/trust are not. *(D1/D2)*

**8. C** — "Registry only" restricts developers to servers in the configured registry. "Allow all" permits any server. *(D2)*

**9. B** — Under "Registry only," even local servers must be listed in the registry with exactly matching server IDs. *(D2)*

**10. B** — `applyTo` = focus (files/dirs); `tools` = permitted actions. They don't control branch/PR behavior or token permissions. *(D2)*

**11. C** — Branch-based scope is enforced by the execution context/system, not the custom agent file. Agents branch + PR; they don't touch `main` directly. *(D2)*

**12. B** — Scheduled cron trigger + minimal `contents: read` = least privilege. `write-all`, plaintext PATs, and logging secrets are wrong. *(D2)*

**13. B** — Server exposes, registry discovers/trusts, allow list permits. *(D2)*

**14. B** — A job-level `if: github.event_name == 'pull_request'` (defensive gating) ensures PR-only steps run only in PR context. *(D2)*

**15. B** — Transient/network failures → bounded retries first. Escalate on *persistent* failure, not the first one. *(D2)*

**16. B** — Safe-iteration policy: after the same check fails twice, escalate with what failed, what was attempted, the evidence, and the suggested next step (prevents infinite loops). *(D2)*

**17. B** — `GITHUB_TOKEN` actions generally don't trigger new runs, preventing recursion/loops. PATs and GitHub App tokens *can* trigger runs. *(D2)*

**18. B** — Read-only for planners/reviewers; write/execution tools only for implementers. *(D2)*

**19. B** — Remote is the recommended default; GHES supports local configuration; GHEC with data residency supports both. Local servers typically need no external auth. *(D2)*

**20. B** — Expanding MCP tools increases blast radius → review like a high-risk dependency; tool-allowlist changes are governance-sensitive. *(D2)*

**21. B** — Durable artifacts (PR description, commits, uploaded artifacts) let work resume without repeating steps. Don't rely on model memory. *(D3)*

**22. B** — Divergence from the approved plan during a long run = context drift; correct by re-anchoring to the recorded plan artifact. *(D3)*

**23. B** — Long-term/external memory persists across sessions; short-term is transient. *(D3)*

**24. B** — Contradictory sources = conflicting context; fix with a single source of truth. (Outdated = stale context.) *(D3)*

**25. A** — Evidence must trace to a specific workflow run and commit/code state for audit/debug. *(D3)*

**26. B** — Wrong tool / malformed args = tool misuse. *(D4)*

**27. B** — CodeQL/code scanning, CI checks, dependency review = automated quantitative signals. *(D4)*

**28. B** — Missing dependency in the runtime = context/environment issue. *(D4)*

**29. B** — Flawed approach → revise instructions/workflow/constraints (and the plan). More retries/secrets/tools won't fix bad reasoning. *(D4)*

**30. B** — Logs, plans, traces, outputs, and workflow artifacts drive root-cause analysis. *(D4)*

**31. B** — Align evaluation with development intent. *(D4)*

**32. B** — Agent isolation (separate branches/worktrees/PRs) prevents overlapping-change collisions. *(D5)*

**33. B** — Stalled/degraded multi-agent coordination → rollback + human-in-the-loop. *(D5)*

**34. B** — Retiring an agent must preserve auditability and workflow continuity. *(D5)*

**35. B** — Post-hoc analysis needs audit-suitable artifacts documenting decisions, handoffs, and outcomes. *(D5)*

**36. C** — Delegation boundary = which subagents can be invoked and how handoffs occur. (Capability = tools; visibility = UI selectability.) *(D5)*

**37. B** — Add agents without disrupting active workflows, preserving continuity. *(D5)*

**38. A** — Contradictory outputs across agents = a conflict to detect and resolve. *(D5)*

**39. C** — Critical/irreversible + secrets → environment approvals; the agent prepares, a human approves execution. *(D6)*

**40. B** — A `pre-tool-use` hook enforces the rule independent of the model's reasoning; prompt/comment/README are guidance only. *(D6)*

---

## Score guide
- **36–40 (90%+):** exam-ready across domains.
- **30–35 (75–89%):** solid; review the domains where you missed.
- **<30 (<75%):** re-read the study guide sections for missed domains and redo those questions.

The real exam needs **700/1000 (70%)** to pass and includes interactive components, so aim comfortably above 75% here.
