---
name: x
description: Use when searching X (Twitter) posts, looking up users or their timelines, managing bookmarks, fetching trends or news, or answering any question that needs live X data. Also use when debugging X MCP auth (401, token refresh) or choosing between MCP tools and raw X API calls.
---

# X MCP

This plugin connects a remote X MCP server (an OAuth proxy in front of `api.x.com/mcp`). **Prefer the MCP tools over raw `curl`/REST calls** — they run with the user's OAuth context and handle auth automatically.

## Tools

| Task | Tool |
|------|------|
| Search posts (full archive) | `search_posts_all` |
| Post volume counts (last 7 days) | `get_posts_counts_recent` |
| Fetch post(s) by ID | `get_posts_by_id`, `get_posts_by_ids` |
| Who liked / reposted / quoted a post | `get_posts_liking_users`, `get_posts_reposted_by`, `get_posts_quoted_posts` |
| Current user | `get_users_me` |
| User by handle / ID | `get_users_by_username`, `get_users_by_usernames`, `get_users_by_id` |
| Search users | `search_users` |
| A user's posts / home timeline / mentions | `get_users_posts`, `get_users_timeline`, `get_users_mentions` |
| Bookmarks: list / add / remove | `get_users_bookmarks`, `create_users_bookmark`, `delete_users_bookmark` |
| Bookmark folders | `get_users_bookmark_folders`, `get_users_bookmarks_by_folder_id`, `create_users_bookmark_folder` |
| News | `search_news`, `get_news` |
| Trends for a location | `get_trends_by_woeid` (WOEID, e.g. 1 = worldwide, 23424977 = US) |

## Usage notes

- Search tools accept the standard X search query syntax (`from:user`, `#tag`, `-is:retweet`, `lang:en`, boolean operators). See the reference below for full syntax, fields/expansions, and rate-limit guidance.
- Requests cost real money (pay-per-use) and writes have strict rate limits — batch lookups with the plural tools (`get_posts_by_ids`, `get_users_by_usernames`) and don't poll in loops.
- Auth is a one-time OAuth login when the server is first connected (`/mcp` command in Claude Code). On persistent `401`s or auth errors, tell the user to reconnect via `/mcp` — don't retry.

## Reference

Full X API capability guide (auth methods, search operators, fields/expansions, rate limits, workflows): read `references/x-api-skill.md` in this skill's directory. It documents the raw REST API — use it for query syntax and concepts, but call the MCP tools above rather than the REST endpoints it shows.
