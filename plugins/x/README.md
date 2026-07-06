# x

X (Twitter) for Claude Code: a remote X MCP server (search, users, bookmarks, trends, news, writes) plus X's official API skill.

## What's inside

- **MCP server `x`** — remote HTTP server at `xmcp.braydenmay.com/mcp`: a Cloudflare Worker OAuth proxy in front of X's hosted MCP (`api.x.com/mcp`). On first use, authenticate with `/mcp` — you log in with your own X account and the server acts with your permissions. Tokens are stored encrypted per-grant on the Worker and auto-refresh.
- **Skill `using-x`** — maps the MCP tools and bundles [X's official skill.md](https://docs.x.com/skill.md) as reference (search syntax, rate limits, auth concepts).

## Note

The Worker is backed by my personal X developer app — API usage is billed to it (pay-per-use). If you're not me, deploy your own instance ([brayd3nmay/mcp](https://github.com/brayd3nmay/mcp) → `x/`) and point `.mcp.json` at it.
