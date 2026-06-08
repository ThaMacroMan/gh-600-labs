# gh-600-labs Repo Setup

## Step 1: Clone your new repo

```bash
git clone https://github.com/YOUR_USERNAME/gh-600-labs
cd gh-600-labs
```

## Step 2: Copy lab files into the repo

Create the folder structure and copy these files from your local `labs/lab-files/` folder:

```
lab-files/lab1/.github/agents/security-reviewer.agent.md  →  .github/agents/security-reviewer.agent.md
lab-files/lab1/.github/agents/planner.agent.md             →  .github/agents/planner.agent.md
lab-files/lab2-workflow/.github/workflows/nightly-agent.yml →  .github/workflows/nightly-agent.yml
lab-files/lab3/CODEOWNERS                                  →  CODEOWNERS
lab-files/lab4/.github/pull_request_template.md            →  .github/pull_request_template.md
lab-files/lab5/.github/hooks/block-dangerous-tools.json    →  .github/hooks/block-dangerous-tools.json
lab-files/lab5/.github/hooks/HOOKS_EXPLAINER.md            →  .github/hooks/HOOKS_EXPLAINER.md
```

Or with bash (run from inside your cloned repo, adjust the source path to wherever your labs folder lives):

```bash
LABS="/path/to/GH-600 Agentuc AI Developer/labs/lab-files"

mkdir -p .github/agents .github/workflows .github/hooks

cp "$LABS/lab1/.github/agents/security-reviewer.agent.md" .github/agents/
cp "$LABS/lab1/.github/agents/planner.agent.md"           .github/agents/
cp "$LABS/lab2-workflow/.github/workflows/nightly-agent.yml" .github/workflows/
cp "$LABS/lab3/CODEOWNERS"                                CODEOWNERS
cp "$LABS/lab4/.github/pull_request_template.md"          .github/pull_request_template.md
cp "$LABS/lab5/.github/hooks/block-dangerous-tools.json"  .github/hooks/
cp "$LABS/lab5/.github/hooks/HOOKS_EXPLAINER.md"          .github/hooks/
```

## Step 3: Commit and push

```bash
git add .
git commit -m "Add GH-600 lab files"
git push
```

## Step 4: Enable Copilot on the repo (for Lab 1)

Repo → **Settings** → **Copilot** → Cloud agent → enable for this repo

## Step 5: Set up branch protection (Lab 3)

Repo → **Settings** → **Rules** → **Rulesets** → New branch ruleset:

- Target branch: `main`
- Require a pull request before merging ✓
- Require status checks to pass ✓ → add check name: `agent-plan-gate`
- Require review from Code Owners ✓
- Include administrators ✓
- Save

Then test it: try pushing directly to `main` — it should be blocked.

## Step 6: Do Lab 6 in VS Code (MCP server)

Open the cloned repo in VS Code and follow `lab-files/lab6/MCP_CONFIG_GUIDE.md`.

---

## Final folder structure in your repo

```
gh-600-labs/
├── README.md
├── CODEOWNERS                              ← Lab 3
└── .github/
    ├── agents/
    │   ├── security-reviewer.agent.md      ← Lab 1
    │   └── planner.agent.md                ← Lab 1
    ├── workflows/
    │   └── nightly-agent.yml               ← Lab 2
    ├── hooks/
    │   ├── block-dangerous-tools.json      ← Lab 5
    │   └── HOOKS_EXPLAINER.md              ← Lab 5
    └── pull_request_template.md            ← Lab 4
```
