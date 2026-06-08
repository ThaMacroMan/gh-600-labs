# Lab 6 Cheat Sheet — MCP (Model Context Protocol)

**Exam domain:** D2 (tool use — highest weight ~20–25%)

---

## Mental model (memorize this triangle)

```
MCP Server    →  exposes tools
Registry      →  makes servers discoverable / trustable
Allow List    →  decides which servers are PERMITTED
```

Three **distinct** things — exam will mix them up in wrong answers.

---

## MCP server

- Standard way for AI clients to connect to tools/services
- **Remote** = recommended default in supported IDEs
  - Example: `https://api.githubcopilot.com/mcp/`
- **Local** = runs on your machine (e.g. `http://localhost:3000`)
- **GitHub Enterprise Server** = local only
- **GHEC with data residency** = both local and remote

Add in VS Code: Copilot Chat → Agent mode → Tools → Configure tools → Add MCP server → HTTP → Authenticate

---

## MCP registry

Catalog of MCP servers for discovery.

| Default | Custom options |
|---------|----------------|
| GitHub MCP Registry | Fork/self-host (Docker), build your own, **Azure API Center** |

**Required endpoints (exam):**
```
GET /v0.1/servers
GET /v0.1/servers/{serverName}/versions/latest
GET /v0.1/servers/{serverName}/versions/{version}
```

**Required CORS headers:**
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type
```

**Azure API Center:** enable anonymous access; enter **base URL only** — do NOT append `/v0.1/servers`

---

## Allow list (org/enterprise policy)

**Where:** Org Settings → Copilot → Policies → MCP (or Enterprise → AI controls → MCP)

| Policy | Behavior |
|--------|----------|
| **Allow all** | Any MCP server permitted |
| **Registry only** | Only servers in configured registry permitted |

**"Registry only" rules (high-yield):**
- Even **local** servers must be in registry with **exactly matching server ID**
- Policy applies **immediately** to all Copilot seat holders
- Correct answer for "only vetted MCP servers"

---

## Custom agents + MCP

In agent profile `tools`:
```yaml
tools: ['read', 'search', 'some-mcp-server/tool-1', 'some-mcp-server/*']
```

- Expanding MCP tools = **governance-sensitive** — review like high-risk dependency
- `/.github/agents/` changes should have CODEOWNERS + platform review

---

## Server vs registry vs allow list

| Question | Answer |
|----------|--------|
| What exposes tools? | MCP **server** |
| What makes servers discoverable? | **Registry** |
| What controls permission to use? | **Allow list** (org/enterprise) |
| Registry only + local server? | Must be in registry with exact server ID |
| Recommended server type? | **Remote** |

---

## Exam traps

| Wrong | Correct |
|-------|---------|
| Registry and allow list are the same | Server exposes, registry discovers, allow list permits |
| Local servers exempt under "Registry only" | Must be listed with exact matching ID |
| Append `/v0.1/servers` to Azure API Center URL | Base URL only |
| Adding MCP tools is low risk | Treat as high-risk dependency change |
| MCP server controls branch behavior | Execution context — not MCP or agent file |

---

## One-liners

- Server exposes, registry discovers, allow list permits.
- Registry only ⇒ local servers need exact IDs in registry.
- Remote MCP = recommended default; GHES = local only.
- Tool/MCP expansion = governance-sensitive — least privilege always.
