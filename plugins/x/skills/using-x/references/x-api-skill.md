---
name: X
description: Use when building applications that access X's public conversation data, including posts, users, trends, spaces, direct messages, and lists. Reach for this skill when agents need to search posts, retrieve user data, manage streams, authenticate requests, handle rate limits, or integrate X data into applications.
metadata:
    mintlify-proj: x
    version: "1.0"
---

# X API Skill

## Product summary

The X API provides programmatic access to X's public conversation through modern REST endpoints. Agents use it to search posts, retrieve user profiles, access trends, manage lists, send direct messages, and analyze engagement metrics. The API uses pay-per-usage pricing with no subscriptions. Key entry points: Bearer Token authentication (app-only), OAuth 1.0a and 2.0 for user context, and REST endpoints at `https://api.x.com/2/`. Official SDKs available for Python and TypeScript handle authentication, pagination, and rate limiting automatically. Primary docs: https://docs.x.com/x-api/introduction

---

## When to use

Reach for this skill when:

- **Searching posts**: User wants to find posts by keyword, hashtag, user, date range, or engagement metrics (recent 7 days or full archive)
- **User lookups**: Agent needs to retrieve user profiles, follower counts, verification status, or user metrics
- **Real-time streaming**: Building applications that need near real-time post delivery with filtered rules
- **Post management**: Creating, editing, or deleting posts programmatically
- **Engagement operations**: Liking, reposting, bookmarking, or managing replies
- **User relationships**: Following, blocking, muting, or managing lists
- **Direct messages**: Sending or retrieving private messages
- **Authentication**: Setting up OAuth flows or managing API credentials
- **Rate limit handling**: Implementing retry logic and monitoring usage
- **Data enrichment**: Requesting specific fields and expansions to customize response payloads

---

## Quick reference

### Authentication methods

| Method | Use case | Credentials |
|:-------|:---------|:------------|
| **Bearer Token (App-only)** | Public data, no user context | API Key + Secret → Bearer Token |
| **OAuth 1.0a User Context** | User-specific actions, private data | API Key + Secret + User Access Token + Secret |
| **OAuth 2.0 Authorization Code** | Third-party apps, user sign-in | Client ID + Secret + Authorization Code |
| **Basic Auth** | Enterprise APIs only | Email + Password (HTTPS only) |

### Essential endpoints

| Endpoint | Method | Purpose |
|:---------|:-------|:---------|
| `/2/tweets/search/recent` | GET | Search posts from last 7 days |
| `/2/tweets/search/all` | GET | Search full post archive (paid) |
| `/2/tweets/search/stream` | GET | Real-time filtered stream |
| `/2/tweets/search/stream/rules` | POST/GET | Manage stream filter rules |
| `/2/tweets/{id}` | GET | Retrieve single post |
| `/2/tweets` | POST | Create post |
| `/2/users/by/username/{username}` | GET | Look up user by username |
| `/2/users/{id}` | GET | Look up user by ID |

### Field and expansion parameters

| Parameter | Purpose | Example |
|:----------|:--------|:---------|
| `tweet.fields` | Request post fields | `created_at,public_metrics,lang` |
| `user.fields` | Request user fields | `created_at,description,public_metrics` |
| `expansions` | Include related objects | `author_id,referenced_tweets.id` |
| `max_results` | Results per page | `10` to `100` (endpoint-specific) |
| `pagination_token` | Navigate pages | Token from previous response |

### Rate limit headers

Every response includes:
- `x-rate-limit-limit`: Max requests in window
- `x-rate-limit-remaining`: Requests left
- `x-rate-limit-reset`: Unix timestamp when window resets

### Common HTTP status codes

| Code | Meaning | Action |
|:-----|:--------|:-------|
| 200 | Success | Parse response |
| 400 | Bad Request | Check query syntax, required params |
| 401 | Unauthorized | Verify Bearer Token or OAuth credentials |
| 403 | Forbidden | App lacks access; check Developer Console |
| 404 | Not Found | Post/user deleted or doesn't exist |
| 429 | Rate Limited | Wait until `x-rate-limit-reset`, implement backoff |
| 5xx | Server Error | Retry with exponential backoff |

---

## Decision guidance

### When to use search vs. stream

| Scenario | Use search | Use stream |
|:---------|:-----------|:-----------|
| Historical data analysis | ✓ | — |
| Real-time monitoring | — | ✓ |
| One-time lookup | ✓ | — |
| Continuous listening | — | ✓ |
| Specific date range | ✓ | — |
| Full archive needed | ✓ (paid) | — |
| Low latency required | — | ✓ |

### When to use fields vs. expansions

| Need | Use fields | Use expansions |
|:-----|:-----------|:---------------|
| More data on primary object | ✓ | — |
| Related object details | — | ✓ |
| Post metrics (likes, reposts) | ✓ | — |
| Author info with post | — | ✓ |
| Reduce response size | ✓ | — |

### Bearer Token vs. OAuth 1.0a

| Requirement | Bearer Token | OAuth 1.0a |
|:------------|:-------------|:-----------|
| Public data only | ✓ | ✓ |
| User-specific actions | — | ✓ |
| Simpler setup | ✓ | — |
| Third-party apps | — | ✓ |
| Production apps | ✓ | ✓ |

---

## Workflow

### Making your first request

1. **Get credentials**: Navigate to [console.x.com](https://console.x.com), create an app, copy the Bearer Token from "Keys and tokens"
2. **Choose endpoint**: Start with user lookup (`/2/users/by/username/{username}`) or post lookup (`/2/tweets/{id}`)
3. **Build request**: Use cURL, Postman, or SDK with Bearer Token in Authorization header
4. **Parse response**: Check HTTP status, extract `data` field, handle `errors` array if present
5. **Add fields**: Append `?tweet.fields=created_at,public_metrics` to request additional data
6. **Handle pagination**: Store `pagination_token` from response, use in next request for more results
7. **Monitor rate limits**: Check response headers, implement exponential backoff on 429 errors

### Searching posts

1. **Construct query**: Use operators like `from:username`, `#hashtag`, `lang:en`, `-is:retweet`
2. **Choose endpoint**: Recent search (7 days, all users) or full-archive (paid, all history)
3. **Set max_results**: Default 10, max 100 for recent search, max 500 for full-archive
4. **Request fields**: Add `tweet.fields=created_at,public_metrics,author_id` for engagement data
5. **Add expansions**: Use `expansions=author_id` to include author details in `includes.users`
6. **Paginate**: Store `next_token` from response, pass in next request to get more results
7. **Verify results**: Check `data` array length, handle partial errors in `errors` array

### Setting up filtered stream

1. **Add rules**: POST to `/2/tweets/search/stream/rules` with rule value (e.g., `{"add": [{"value": "from:xdevelopers"}]}`)
2. **Verify rules**: GET `/2/tweets/search/stream/rules` to confirm rules are active
3. **Connect stream**: GET `/2/tweets/search/stream` with Bearer Token, keep connection open
4. **Handle disconnections**: Implement reconnect logic with exponential backoff
5. **Process posts**: Parse JSON objects as they arrive, each line is a complete post object
6. **Update rules**: POST new rules or delete old ones without stopping stream
7. **Monitor capacity**: Check rule count (max 1,000), rule length (max 1,024 chars)

### Authenticating with OAuth 1.0a

1. **Get credentials**: API Key, API Secret, User Access Token, User Access Token Secret from Developer Console
2. **Build signature**: Combine credentials with request method, URL, and parameters
3. **Add header**: Include `Authorization: OAuth oauth_consumer_key="...", oauth_token="...", oauth_signature="..."` etc.
4. **Use library**: Prefer official SDK or OAuth library to avoid signature errors
5. **Verify scopes**: Ensure app has required permissions for the endpoint
6. **Handle token refresh**: Tokens don't expire but can be revoked; implement error handling

---

## Common gotchas

- **Missing Bearer Token**: 401 errors mean token is invalid, regenerated, or not in Authorization header. Always use format `Bearer YOUR_TOKEN`
- **Forgetting fields parameter**: Default response returns only `id` and `text` for posts. Add `tweet.fields=created_at,public_metrics` to get engagement data
- **Confusing expansions with fields**: Expansions include related objects (author, media); fields request additional data on primary object. Use both together
- **Rate limit exhaustion**: 429 errors require waiting until `x-rate-limit-reset` timestamp. Implement exponential backoff (wait 1s, 2s, 4s, etc.)
- **Stream rules not matching**: Rules are case-sensitive and use exact phrase matching. Use quotes for phrases: `"machine learning"` not `machine learning`
- **Pagination token expiration**: Tokens expire after 30 seconds of inactivity. Store and use immediately
- **Protected account posts**: Posts from protected accounts only visible with user context auth. Public endpoints return 403 for protected content
- **Deleted posts return 404**: Don't retry; handle gracefully in error handling
- **Query length limits**: Recent search max 512 chars, full-archive max 1,024 chars (4,096 for Enterprise). Simplify queries if hitting limits
- **Partial errors in 200 responses**: Check `errors` array even when HTTP status is 200. Some resources may fail while others succeed
- **Stream buffer overflow**: If client doesn't consume fast enough, connection drops. Implement async processing or increase buffer size
- **OAuth 1.0a signature errors**: Use official SDK or library; manual signature calculation is error-prone. Check timestamp is within 5 minutes of server time

---

## Verification checklist

Before submitting work with X API integration:

- [ ] **Authentication**: Verify Bearer Token or OAuth credentials are valid and in correct header format
- [ ] **Endpoint**: Confirm endpoint URL is correct and method (GET/POST/DELETE) matches documentation
- [ ] **Required parameters**: Check all required query parameters or request body fields are present
- [ ] **Fields and expansions**: Verify fields parameter includes needed data; expansions include related objects
- [ ] **Rate limits**: Implement retry logic for 429 errors; check `x-rate-limit-remaining` proactively
- [ ] **Error handling**: Check for both HTTP status codes and `errors` array in response body
- [ ] **Pagination**: Store and use `pagination_token` for multi-page results; don't assume all results in first request
- [ ] **Data validation**: Verify response structure matches expected schema; handle null/missing fields
- [ ] **Credentials security**: Never commit API keys, tokens, or secrets; use environment variables
- [ ] **Stream handling**: For filtered stream, implement reconnection logic and handle disconnections gracefully
- [ ] **Testing**: Test with Postman or SDK before deploying; verify with small datasets first

---

## Resources

**Comprehensive navigation**: https://docs.x.com/llms.txt — Full page-by-page listing for agent navigation

**Critical documentation**:
1. [Make Your First Request](https://docs.x.com/make-your-first-request) — Quick start with cURL and SDKs
2. [Authentication Overview](https://docs.x.com/fundamentals/authentication/overview) — All auth methods and setup
3. [Search Posts Introduction](https://docs.x.com/x-api/posts/search/introduction) — Search operators and endpoints

---

> For additional documentation and navigation, see: https://docs.x.com/llms.txt