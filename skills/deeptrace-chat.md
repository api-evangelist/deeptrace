---
name: Ask Deeptrace about production and continue the conversation
description: Send a message to Deeptrace's AI chat, continue multi-turn by reusing chat_id, and retrieve or list past chats.
api: openapi/deeptrace-api-openapi.yml
operations: [sendChatMessage, getChat, listChats]
---

# Ask Deeptrace about production and continue the conversation

Deeptrace exposes an AI chat over your production systems. Send a message, get an
AI-powered response, and continue the thread by passing the returned `chat_id`.

## Auth
- Base URL: `https://api.deeptrace.com`
- Send `Authorization: Bearer <your_api_key>` (the same `dt_` Deeptrace API key).

## Steps

1. **Send a message** — `sendChatMessage`
   `POST /api/v1/chat` with body `{"messages": [{"role": "user", "content": "<question>"}]}`.
   The response includes `chat_id` and `response`. Optionally set `model` (default
   `claude-opus-4-6`) or `system_prompt`. Add `?stream=true` to receive Server-Sent
   Events instead of a single JSON response.

2. **Continue the conversation** — call `sendChatMessage` again with the same
   `chat_id` in the body plus the new user message.

3. **Retrieve a chat** — `getChat`
   `GET /api/v1/chats/{chat_id}` returns the full history: `messages` (content blocks),
   `data_sources`, `investigation_results`, and `citation_map`.

4. **List chats** — `listChats`
   `GET /api/chat?page=1&count=20` for a paginated list; `count` max is 100. Set
   `my_chats=true` to filter to your own chats.

## Rules
- Reuse `chat_id` for multi-turn context instead of resending prior turns as new chats.
- Paginate with `page`/`count`; do not request `count` above 100.
- Handle `401`/`403` as auth failures (check the key), `422` as a validation error on
  the message array.
