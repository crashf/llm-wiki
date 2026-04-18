---
title: N8N Design Patterns
type: concept
tags: [n8n, automation, api, troubleshooting]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

Key patterns and lessons learned from building n8n workflows programmatically via REST API. The most critical discovery is a compatibility bug with If nodes that causes UI rendering failures when using typeVersion 2.x.

## Key Points

- Use REST API with `X-N8N-API-KEY` header for all workflow operations
- Never edit n8n SQLite database directly — corrupts workflow data
- If nodes MUST use typeVersion 1 with `conditions.boolean` format
- Keep credentials in n8n, never hardcode secrets in nodes
- Use Manual Trigger for initial testing

## Connections

- Related to [[n8n-maintenance-automation]] — patterns applied in the maintenance workflow
- Related to [[llm-wiki-pattern]] — wiki pattern used to document these discoveries

## Details

### If Node typeVersion Bug (CRITICAL)

When building If nodes via API, **always use typeVersion 1** with the `conditions.boolean` format. typeVersion 2.x nodes (2, 2.1, 2.2) save correctly to the database but render as **empty/blank** in the n8n UI — conditions appear missing even though they exist in the database.

#### ✅ Correct format (typeVersion 1 — always use this):

```json
{
  "type": "n8n-nodes-base.if",
  "typeVersion": 1,
  "parameters": {
    "conditions": {
      "boolean": [
        {
          "value1": "={{ $json.someField }}",
          "value2": true
        }
      ]
    }
  }
}
```

#### ❌ Broken format (typeVersion 2.2 — do NOT use via API):

```json
{
  "typeVersion": 2.2,
  "parameters": {
    "conditions": {
      "options": { "version": 2, ... },
      "conditions": [{ "leftValue": "...", "rightValue": true, "operator": {...} }]
    }
  }
}
```

### API Workflow Creation Pattern

1. read_file workflow JSON from file
2. Prepare payload (name, nodes, connections, settings)
3. POST to `/api/v1/workflows` with header `X-N8N-API-KEY`
4. Do NOT include `active: false` in payload — causes "read-only" error

Example:
```bash
KEY=$(cat ~/.openclaw/workspace/.secrets/n8n-api-key.txt)
curl -s -X POST \
  -H "X-N8N-API-KEY: $KEY" \
  -H "Content-Type: application/json" \
  -d @workflow.json \
  "https://n8n.pund-it.ca/api/v1/workflows"
```

### Safe Patterns

- Create new workflows as **inactive** by default
- Use **Manual Trigger** for initial testing
- Keep credentials in n8n (do not hardcode secrets in nodes)
- Keep workflow names short and purpose-driven
- Export JSON for versioning when requested

### ConnectWise Integration Patterns

- Use existing credential: `Connectwise API Httpauth`
- CW v4_6 requires **ClientId** header; ensure **Send Headers = ON** and `ClientId: <value>` is set
- If `$env.CWM_BASE_URL` is blocked, hardcode the base URL
- For CW nodes, clone config from a known-good HTTP Request node (method-first layout) to keep auth/options consistent