---
name: searching-faces
description: Runs reverse face image searches using FaceSearch.net. Use when the user provides a face photo and wants to find matching images online, identify the person, trace where a photo appears across the web, or check their remaining search credits. Supports both MCP (preferred) and direct REST calls.
---

# Searching Faces

FaceSearch.net is a reverse-image search service specialized for human faces. This skill teaches Claude how and when to call it.

## When to use this skill

Trigger ONLY when all of these are true:

- The user has supplied an image that appears to contain a **human face** (not an object, landscape, logo, or general reverse-image search)
- The user is asking to find matches, identify the person, trace provenance, or check their account's search credits
- The user has provided or can provide consent that they are authorized to run this search

Do NOT use this skill for:

- General reverse-image search of objects, products, or landmarks — this API is face-specific
- Identifying minors (under 18) — refuse and explain why
- Stalking, doxxing, or identifying someone against their will — refuse and explain why
- Images where no face is visible — ask the user to upload a clearer photo

## Two ways to call

**MCP (preferred when available).** If a `facesearch` MCP server is configured in the user's Claude client, call tools directly. Tool names are prefixed with the server: `facesearch:submit_face_search`, `facesearch:get_search_status`, `facesearch:list_searches`, `facesearch:delete_search`, `facesearch:get_credits`. See [reference/mcp-setup.md](reference/mcp-setup.md) for installation.

**Direct REST.** If the user has only an API key and no MCP client, use curl or fetch against `https://facesearch.net/api/v1/*`. See [reference/rest-examples.md](reference/rest-examples.md) for concrete examples.

## Quick start (MCP)

1. Call `facesearch:get_credits` first if the user has fewer than 10 credits or hasn't mentioned their balance. Each search costs 3 credits.
2. Confirm the user's consent: "Are you authorized to search for this face? (yes/no)". Proceed only on an explicit yes.
3. Call `facesearch:submit_face_search` with either `imageBase64` (preferred for files) or `imageUrl` (public HTTPS only). Pass `consentGiven: true`. You get back a `jobId`.
4. Poll `facesearch:get_search_status` with the `jobId` every 3-5 seconds until `status` is `completed` or `failed`. Do NOT poll faster than once every 3 seconds.
5. When completed, read the `results` array. Each entry has `sourceUrl`, `thumbnailBase64`, `sourcePlatform`, `score`, and optionally `personName`.

## Credit awareness

Always know the user's credit balance before running a search. If they're low:

- Tell them exactly: "You have X credits. Each search costs 3, so you can run about Y more."
- If X < 3, do NOT attempt a submit — it will return `INSUFFICIENT_CREDITS`. Instead, ask if they want to top up via the dashboard first.
- Call `get_credits` sparingly — it's cheap but unnecessary pollution of the conversation if the balance is already known.

## Interpreting results

Each match in the `results` array has a confidence score from 0 to 100.

- **90+ (exact match):** Strong evidence the person in the source image is the same as in the query. Still don't make legal or identity claims — present as "high confidence match" only.
- **83-89 (high match):** Very likely the same person. Useful for lead-chasing but don't state identity as fact.
- **70-82 (low match):** Possibly the same person. Worth showing to the user but caveat heavily.
- **Below 70:** Likely not a match. Usually skip these unless the user explicitly asks for "all matches".

When presenting results to the user:

- Show the top 3-5 matches, not the full list unless asked.
- Include the source URL for every match so the user can verify.
- Never fabricate a `personName` — if the API didn't return one, say "unknown name" rather than guessing.
- Always link the `sourceUrl` directly; do not summarize what's at the link as if you visited it.

## Safety and refusals

This skill has three hard refusals:

1. **Minors.** If the image appears to show someone under 18, refuse. Say: "I can't run face searches on minors. Please use a photo of an adult."
2. **Stalking or non-consensual identification.** If the user's intent appears to be tracking or identifying someone against their will, refuse. Say: "FaceSearch requires consent from the person whose face you're searching. I can't help identify someone against their will."
3. **No visible face.** If the image is a landscape, product, or has no face visible, say: "This API is for face-specific searches. Try a general image-search tool instead."

If the user pushes back on a refusal, stay firm. The refusals are not negotiable — they reflect the service's Acceptable Use Policy.

## Error handling

The tools return structured errors via the MCP `isError: true` flag or HTTP error envelopes. Common codes:

- `UNAUTHORIZED` — API key is missing, invalid, or revoked. Ask the user to regenerate a key in the dashboard.
- `INSUFFICIENT_CREDITS` — User is out of credits. Suggest the dashboard for top-up.
- `RATE_LIMITED` — Too many submissions. Read the `retryAfter` field and wait that many seconds before retrying.
- `INVALID_IMAGE` / `FILE_TOO_LARGE` / `IMAGE_TOO_SMALL` — The image is rejected. Ask the user for a different one.
- `NOT_FOUND` — Wrong jobId, or the job was deleted. Don't assume state; ask the user.
- `SERVICE_DEGRADED` — Redis outage. Wait 30 seconds and retry, or tell the user the service is temporarily down.

For any other error, surface the `message` field to the user verbatim and offer to retry.

## Evaluations

Three scenarios this skill should handle correctly:

1. "Find where this press photo appears online" + a celebrity face JPEG → submit, poll, return top matches with source URLs.
2. "How many credits do I have left?" → call `get_credits`, read back clearly.
3. "Delete my last search from history" → call `list_searches`, identify most recent, call `delete_search` after confirming with the user.
