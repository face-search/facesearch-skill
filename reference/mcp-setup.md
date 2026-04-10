# FaceSearch MCP setup

Install the FaceSearch MCP server so Claude can call face-search tools directly from chat. One-time setup; takes 60 seconds.

## Get an API key

1. Sign in at https://facesearch.net
2. Dashboard → API keys → Create new key
3. Copy the `fs_live_...` value. It is shown exactly once — you cannot retrieve it later.
4. Optionally set a daily or monthly quota on the key to cap leakage if it's ever lost.

## Add to Claude Code

Edit `~/.claude.json` (or `.mcp.json` in your project) and add:

```json
{
  "mcpServers": {
    "facesearch": {
      "url": "https://facesearch.net/api/mcp/http",
      "headers": {
        "Authorization": "Bearer fs_live_YOUR_KEY_HERE"
      }
    }
  }
}
```

Restart Claude Code. The `facesearch` server should appear in `/mcp` output with five tools available.

## Add to Cursor

Edit `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "facesearch": {
      "url": "https://facesearch.net/api/mcp/http",
      "headers": {
        "Authorization": "Bearer fs_live_YOUR_KEY_HERE"
      }
    }
  }
}
```

## Test the connection

Ask your assistant: "Call facesearch:get_credits for me". You should see your current credit balance. If you get an auth error, double-check the Bearer token in the config file.

## Troubleshooting

- **"Missing or invalid API key"**: The Bearer token is wrong, the key was revoked, or the header format is off. Create a fresh key in the dashboard.
- **"RATE_LIMITED"**: You exceeded your per-key rate limit. Wait the number of seconds specified in `retryAfter` and try again. Consider raising the limit on the key in the dashboard.
- **"INSUFFICIENT_CREDITS"**: Balance is below the 3-credit cost of a search. Top up via the dashboard.
- **"SERVICE_DEGRADED"**: Redis outage on our side. Wait 30 seconds, then retry. If it persists, check status.facesearch.net.
- **Tool not found**: Your MCP client is not fully-qualified. Always reference tools as `facesearch:submit_face_search`, not just `submit_face_search`.

## Rotating a key

Dashboard → API keys → Create new → update your `.mcp.json` → revoke the old key. The old key stops working within one request.
