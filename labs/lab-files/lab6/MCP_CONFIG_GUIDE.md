# Lab 6 — MCP Server Configuration
## Domain 2: Implement Tool Use & Environment Interaction

## The mental model (memorize this)

```
MCP Server    →  exposes tools
Registry      →  makes servers discoverable / trustable
Allow List    →  decides which servers are PERMITTED
```

These are three distinct things. The exam will mix them up in wrong answers.

---

## Hands-on: Add the GitHub Remote MCP Server in VS Code

1. Open VS Code with GitHub Copilot extension installed and signed in
2. Click the **Copilot icon** in the sidebar
3. Open **Copilot Chat** → switch to **Agent mode** (dropdown at top of chat)
4. Click the **Tools icon** (wrench/hammer)
5. Click **Configure tools** → **Add MCP server**
6. Choose **HTTP** (this is a remote server)
7. Enter the URL: `https://api.githubcopilot.com/mcp/`
8. Choose scope: **Workspace** (scopes to this repo) or **User** (all repos)
9. Click **Authenticate**
10. Observe the tools the GitHub MCP server exposes (repos, issues, PRs, etc.)

> **Why remote, not local?**  
> Remote = recommended default for most users. Easier to share, no local process needed.  
> Local = runs on your machine, accesses local resources, no external auth usually needed.  
> GitHub Enterprise Server = **local only**  
> GitHub Enterprise Cloud with data residency = supports both

---

## Org-level Allow List (conceptual — requires org admin)

### Where to configure:
**Org:** Settings → Copilot → Policies → enable MCP → set Registry URL → choose policy

### Policy options:
| Setting | Behavior |
|---------|----------|
| Allow all | Developers can use any MCP server |
| Registry only | Only servers listed in the configured registry are permitted |

### "Registry only" — what you must know:
- Even **local** MCP servers must be listed in the registry
- The registry entry must have an **exactly matching server ID**
- Policy applies **immediately** to all developers with Copilot seats
- This is the correct answer when the exam asks "how do you ensure developers only use vetted MCP servers?"

---

## Custom MCP Registry

If your org wants to host its own registry (not use the GitHub default):

**Options:**
1. Fork/self-host the open-source GitHub MCP registry (run via Docker)
2. Build your own
3. Use **Azure API Center** as a managed registry

**Required endpoints** (know these for the exam):
```
GET  /v0.1/servers
GET  /v0.1/servers/{serverName}/versions/latest
GET  /v0.1/servers/{serverName}/versions/{version}
```

**Required CORS headers:**
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type
```

**Azure API Center:** enable anonymous access; enter **base URL only** (do NOT append `/v0.1/servers`)

---

## Enterprise allow list location

Enterprise: enterprise settings → **AI controls** → **MCP** → enable → set Registry URL → choose "Allow all" or "Registry only"

---

## Exam question patterns

**Q: An org wants developers to only use approved MCP servers. Which setting?**  
A: "Registry only" with approved servers listed in the configured registry

**Q: With "Registry only" enforced, what's required for a local MCP server to work?**  
A: It must be listed in the registry with an **exactly matching server ID**

**Q: Which is the recommended MCP server type for most users in supported IDEs?**  
A: Remote

**Q: What does a registry do vs an allow list?**  
A: Registry = discovery/trust (catalog of servers). Allow list = permission (which are permitted to be used).

**Q: GitHub Enterprise Server supports which MCP server type?**  
A: Local only. GHEC with data residency supports both.

**Q: Adding or expanding MCP tools for an agent should be treated as?**  
A: A high-risk dependency change that increases blast radius — review it like you'd review a security-sensitive dependency.
