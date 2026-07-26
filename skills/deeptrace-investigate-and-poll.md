---
name: Trigger a Deeptrace investigation and poll for the root cause
description: Start an asynchronous Deeptrace investigation of a production issue, then poll until it completes and read the root-cause analysis.
api: openapi/deeptrace-api-openapi.yml
operations: [triggerInvestigation, getInvestigation]
---

# Trigger a Deeptrace investigation and poll for the root cause

Deeptrace is an AI SRE. You describe a production symptom; it investigates across
logs, metrics, and code and returns a root-cause analysis. Investigations are
asynchronous: you trigger one, get an id back immediately, then poll for the result.

## Auth
- Base URL: `https://api.deeptrace.com`
- Send your API key (`dt_` prefix) as `X-API-Key: <key>` **or** `Authorization: Bearer <key>`.
- Create keys at https://app.deeptrace.com/api-keys.

## Steps

1. **Trigger the investigation** — `triggerInvestigation`
   `POST /api/v1/investigate` with JSON body `{"query": "<describe the issue>"}`.
   The response returns `investigation_id` and `status: "processing"`.

2. **Poll for results** — `getInvestigation`
   `GET /api/v1/investigations/{investigation_id}`.
   While running, `status` is `processing` and `alert_summary` / `investigation_text`
   are `null`. Poll every few seconds; investigations usually finish in a couple of
   minutes.

3. **Read the result** — when `status` is `completed`, read `alert_summary` (one-line
   root cause) and `investigation_text` (full analysis). For a citation-rich web view,
   link to `https://app.deeptrace.com/investigation/{investigation_id}`.

## Rules
- Do not tight-loop; back off a few seconds between polls.
- Handle `401` (missing/invalid key) and `403` (expired/revoked key) by re-checking the
  API key rather than retrying blindly. `422` means the request body failed validation.
- There is no idempotency key — avoid re-POSTing the same query on transient errors
  before confirming the first call did not already create an investigation.
