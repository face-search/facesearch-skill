# FaceSearch REST API examples

Use this reference when the MCP server is not available and you need to call the REST API directly with curl or fetch. The OpenAPI spec is at https://facesearch.net/api/v1/openapi.json.

## Authentication

Every request to `/api/v1/*` requires a Bearer header:

```
Authorization: Bearer fs_live_YOUR_KEY
```

Keys are created from the dashboard and shown exactly once. Never commit them to source control.

## Submit a search

```bash
curl -X POST https://facesearch.net/api/v1/search \
  -H "Authorization: Bearer fs_live_YOUR_KEY" \
  -F "image=@/path/to/face.jpg" \
  -F "consent=true"
```

Returns:

```json
{
  "success": true,
  "data": {
    "jobId": "abc123",
    "status": "pending",
    "deduplicated": false
  }
}
```

Retry-safe by default. If the network flakes and you resubmit the same image, the server derives a content-hash idempotency key and returns the original jobId. To force a fresh search of the exact same image, send an explicit header:

```bash
curl -X POST https://facesearch.net/api/v1/search \
  -H "Authorization: Bearer fs_live_YOUR_KEY" \
  -H "Idempotency-Key: fresh-$(date +%s)" \
  -F "image=@/path/to/face.jpg" \
  -F "consent=true"
```

The response header `X-Idempotency-Key-Source` tells you which path was used (`header` or `content`).

## Poll for completion

```bash
curl https://facesearch.net/api/v1/search/abc123 \
  -H "Authorization: Bearer fs_live_YOUR_KEY"
```

Returns one of three shapes:

- `status: "pending"` or `"processing"`: check back in 3-5 seconds. May include a `progress` number 0-100.
- `status: "completed"`: the `data` object includes a `results` array with matches.
- `status: "failed"`: the `error` field explains why.

## List search history (PRO only)

```bash
curl "https://facesearch.net/api/v1/search?page=1&limit=10&status=completed" \
  -H "Authorization: Bearer fs_live_YOUR_KEY"
```

Query parameters:
- `page` (default 1)
- `limit` (default 10, max 50)
- `status` (optional: pending | processing | completed | failed)
- `from` and `to` (ISO 8601 date-time filters)

## Delete a search

```bash
curl -X DELETE https://facesearch.net/api/v1/search/abc123 \
  -H "Authorization: Bearer fs_live_YOUR_KEY"
```

Returns 204 on success, 404 if not found or not owned by your key, 409 if the job is still processing.

## Check credit balance

```bash
curl https://facesearch.net/api/v1/credits \
  -H "Authorization: Bearer fs_live_YOUR_KEY"
```

Returns:

```json
{
  "success": true,
  "data": {
    "credits": 42,
    "reservedCredits": 0,
    "availableCredits": 42
  }
}
```

## Export a completed search

```bash
curl https://facesearch.net/api/v1/search/abc123/export \
  -H "Authorization: Bearer fs_live_YOUR_KEY"
```

PRO only. Returns the full job + results as JSON, suitable for rendering into a PDF client-side.

## Rate limits and headers

Every response carries rate-limit metadata:

- `X-RateLimit-Remaining`: how many requests you have left in the current window
- `X-RateLimit-Reset`: Unix timestamp when the window resets
- `Retry-After` (on 429 only): seconds to wait before retrying
- `X-Request-Id`: trace ID for debugging — include in support tickets

## Error envelope

Every failed request returns:

```json
{
  "success": false,
  "error": {
    "code": "CODE_IN_SCREAMING_SNAKE",
    "message": "Human-readable explanation"
  }
}
```

Common codes:

| Code | HTTP | Meaning |
|---|---|---|
| `UNAUTHORIZED` | 401 | Bearer missing, invalid, revoked, or expired |
| `INSUFFICIENT_CREDITS` | 402 | Balance below the 3-credit cost |
| `VALIDATION_ERROR` | 400 | Bad request body or query params |
| `INVALID_IMAGE` | 400 | Image rejected by server-side validator |
| `FILE_TOO_LARGE` | 400 | Image exceeds 10 MB |
| `IMAGE_TOO_SMALL` | 400 | Dimensions below 200x200 |
| `IMAGE_TOO_LARGE` | 400 | Dimensions above 4096x4096 |
| `PAYLOAD_TOO_LARGE` | 413 | Request body exceeds 10 MB |
| `RATE_LIMITED` | 429 | Exceeded per-key rate limit; see `Retry-After` |
| `NOT_FOUND` | 404 | Wrong jobId, or not owned by the key |
| `SEARCH_IN_PROGRESS` | 409 | Cannot delete a job that is still processing |
| `SUBSCRIPTION_REQUIRED` | 403 | PRO-only feature requested with free tier |
| `SERVICE_DEGRADED` | 503 | Redis outage; retry after 30 seconds |
| `INTERNAL_ERROR` | 500 | Unexpected server error; include `X-Request-Id` in support tickets |

## CORS

All v1 routes allow `Access-Control-Allow-Origin: *` because auth is Bearer-only (no cookies means no ambient authority to steal). You can call the API from any browser origin.
