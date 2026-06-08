# Lab 3 Cheat Sheet — CODEOWNERS + Branch Rulesets

**Exam domains:** D1 · D6 (guardrails & enforcement)

---

## Mental model

> **CODEOWNERS + required reviews via rulesets = ENFORCEMENT**
> **Agent instructions "don't touch `/infra/`" = GUIDANCE (weak)**

CODEOWNERS alone is **not enough** — you must also enable **"Require review from Code Owners"** in a branch ruleset.

---

## Where CODEOWNERS lives

Valid paths (first match wins):
- `CODEOWNERS` (repo root)
- `.github/CODEOWNERS`
- `docs/CODEOWNERS`

---

## Example patterns

```text
*                           @org/core-team          # catch-all
/src/                       @org/core-team          # medium risk
/infra/                     @org/platform-team      # high risk
/.github/workflows/         @org/platform-team      # high risk — can self-elevate perms
/.github/agents/            @org/platform-team      # governance-sensitive
/.github/hooks/             @org/platform-team      # governance-sensitive
/config/production*         @org/platform-team      # critical
```

Multiple lines for same path → **all listed owners** may be required (depends on ruleset config).

---

## Ruleset setup (hands-on checklist)

**Repo → Settings → Rules → Rulesets → New branch ruleset**

| Setting | Enable |
|---------|--------|
| Target branch | `main` |
| Require pull request before merging | ✓ |
| Require review from Code Owners | ✓ |
| Require status checks to pass | ✓ (optional: add `agent-plan-gate`) |
| Block direct pushes | ✓ |
| Include administrators | ✓ |

---

## Autonomy levels by path risk

| Risk | Paths | Autonomy design |
|------|-------|-----------------|
| **Low** | `docs/` | Automerge after required checks |
| **Medium** | `src/`, deps | PR + checks + ≥1 review |
| **High** | `infra/`, `.github/workflows/` | CODEOWNERS + multiple reviews + stricter ruleset |
| **Critical** | prod deploys, secrets, production config | Environment approvals — **agent prepares, human approves** |

---

## CODEOWNERS vs custom agent `applyTo`

| | CODEOWNERS + ruleset | Agent `applyTo` |
|---|---------------------|-----------------|
| Blocks merge? | Yes (with ruleset) | No |
| Auto-requests reviewers? | Yes | No |
| Scopes agent focus? | No | Yes |
| Enforcement? | **Yes** | Focus only |

---

## Exam traps

| Wrong | Correct |
|-------|---------|
| CODEOWNERS file alone enforces review | Must + ruleset "Require review from Code Owners" |
| Tell agent not to edit `/infra/` | CODEOWNERS for `/infra/` + required reviews |
| Agent file prevents workflow changes | CODEOWNERS on `/.github/workflows/` + ruleset |
| Skip review on agent tool expansion | `/.github/agents/` changes = governance-sensitive |

---

## One-liners

- CODEOWNERS routes reviewers; rulesets **enforce** the review requirement.
- Workflow files are high risk — they can grant themselves elevated permissions.
- Critical actions: agent prepares, human approves via **environment approvals**.
