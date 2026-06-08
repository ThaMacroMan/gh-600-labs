# Lab 4 Cheat Sheet — PR Template (Plan / Evidence / Review)

**Exam domains:** D1 (planning/validation) · D6 (governance)

---

## Mental model

PRs are **architectural control points** — not just collaboration. The template structures three SDLC phases into every agent PR.

---

## Three sections = three phases

| Section | Phase | What goes here |
|---------|-------|----------------|
| **## Plan** | Planning | Goal, files changed, steps, risks/reversibility |
| **## Evidence** | Validation | Verifiable checkboxes — tests, scans, coverage |
| **## Review checklist** | Human gate | Intent, scope, secrets, irreversible changes |

---

## Where each phase lives in GitHub

| Phase | GitHub-native location |
|-------|------------------------|
| Planning | PR description (`## Plan`) |
| Execution | Commits on the branch |
| Validation | Checks, scans, artifacts, review outcomes (`## Evidence`) |

---

## Success criteria rules

**Verifiable (exam-correct):**
- All required status checks pass
- CodeQL shows no new high-severity alerts
- Secret scanning clean
- Tests pass

**Not verifiable (exam trap):**
- "Improve code quality"
- "Make it cleaner"
- "Reviewer is happy"

---

## Plan-first vs plan+execution (template context)

| Pattern | Template usage |
|---------|----------------|
| **Plan-first PR** | PR contains **only** the plan; code comes in a **second** PR after approval |
| **Plan+execution PR** | Same PR has plan in description **and** code commits |

Choose by **risk/reversibility** — high-risk paths (`.github/workflows/`, infra, auth) → plan-first.

---

## Making the plan mandatory (enforcement)

Template alone = **guidance** (agent can leave sections empty).

To **enforce**:
1. Add a **Plan Gate** workflow that fails if `## Plan` is missing/empty
2. Mark that check as **required** in branch ruleset / branch protection

> Template = guidance. Required check = enforcement.

---

## Review checklist purpose

Gate **only what materially reduces risk** — preserve velocity:
- Intent matches task brief
- Scope is appropriate
- `.github/` changes have platform review (per CODEOWNERS)
- No hardcoded secrets
- Irreversible changes flagged and approved

---

## Exam traps

| Wrong | Correct |
|-------|---------|
| PR template alone guarantees a plan | Add required status check (Plan Gate) + ruleset |
| Subjective success criteria | Automated, verifiable signals |
| Skip plan for high-risk workflow changes | Plan-first for infra/auth/workflows/prod |
| Validation lives in agent memory | Checks, scans, artifacts in GitHub |

---

## One-liners

- Planning → PR description. Execution → commits. Validation → checks/scans/artifacts.
- "Looks better" is not a success criterion.
- Template = guidance; required check = enforcement.
