## Plan
<!-- 
DOMAIN 1 EXAM CONCEPT: The "planning" phase lives HERE — in the PR description.
Planning (PR desc) → Execution (commits) → Validation (checks/scans/reviews)

For HIGH RISK tasks: this plan should exist in a PLAN-FIRST PR 
(code is generated only AFTER this plan is approved)

For LOW/MEDIUM tasks: plan + code in the same PR is acceptable
-->

**Goal:** <!-- One sentence: what is this change trying to achieve? -->

**Files changed and why:**
- `path/to/file.ts` — 

**Steps the agent took:**
1. 
2. 

**Risks / reversibility:** <!-- How hard is this to undo if something goes wrong? -->

---

## Evidence
<!--
DOMAIN 1 EXAM CONCEPT: The "validation" phase lives HERE — verifiable, automated.
Success criteria must be VERIFIABLE (checks pass, scans clean), not subjective.
"The code looks better" is NOT a valid success criterion.
-->

- [ ] All required status checks pass
- [ ] Code scanning (CodeQL) shows no new high-severity alerts  
- [ ] Secret scanning shows no new alerts
- [ ] Tests pass (coverage stable or improved)
- [ ] Dependency review shows no new critical vulnerabilities

---

## Review checklist
<!--
DOMAIN 6 EXAM CONCEPT: Human review is focused on what matters.
Gate only the actions that materially reduce risk (preserve velocity).
-->

- [ ] Intent is clear and matches the task brief
- [ ] Scope is appropriate (no unintended changes)
- [ ] Changes to `.github/` paths have platform team review (per CODEOWNERS)
- [ ] No hardcoded secrets or credentials
- [ ] Irreversible changes have been flagged and approved

---

<!-- 
─────────────────────────────────────────────────────────────
EXAM NOTES on this template:

Q: Where does "planning" surface in GitHub?
A: PR description (here, in the ## Plan section)

Q: Where does "execution" surface?
A: Commits on the branch

Q: Where does "validation" surface?  
A: Checks, scans, artifacts, review outcomes (the ## Evidence section)

Q: How do you make the plan a REQUIREMENT, not just optional?
A: Required status check (e.g., a "Plan Gate" workflow that fails if ## Plan is empty)
   Mark it required via rulesets/branch protection — admin configures this

Q: What's the difference between plan-first PR and plan+execution PR?
A: Plan-first: code exists ONLY after plan is approved (high risk, hard to reverse)
   Plan+execution: both in one PR (lower risk, faster iteration)
   Choice is based on RISK/REVERSIBILITY, not speed preference
─────────────────────────────────────────────────────────────
-->
