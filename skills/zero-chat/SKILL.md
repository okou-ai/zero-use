---
name: zero-chat
description: Use the Zero API (api.vm0.ai) to send messages to AI chat threads, retrieve thread details, and list messages. Triggers when the user asks to send messages to Zero, interact with the Zero chat API, or manage chat threads programmatically via API.
---

# Zero Chat API

Use this skill to interact with the Zero API at `https://api.vm0.ai`.

## Authentication

All requests require a Bearer token. Read it from the environment:

```bash
echo $ZERO_API_KEY
```

Pass it as an HTTP header:

```
Authorization: Bearer $ZERO_API_KEY
```

If `$ZERO_API_KEY` is not set, ask the user to set it before proceeding.

## API Endpoints

Base URL: `https://api.vm0.ai`

---

### Send a Message

Creates a new thread and sends the first message, or appends to an existing thread.

```bash
# New thread
curl -X POST https://api.vm0.ai/api/v1/chat-threads/messages \
  -H "Authorization: Bearer $ZERO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Your message here"}'

# Append to existing thread
curl -X POST https://api.vm0.ai/api/v1/chat-threads/messages \
  -H "Authorization: Bearer $ZERO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Follow-up message", "threadId": "<uuid>"}'
```

**Request body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `prompt` | string | Yes | The message content (min 1 character) |
| `threadId` | string (UUID) | No | Omit to start a new thread |

**Response:** `201 Created`

---

### Get Thread Details

```bash
curl https://api.vm0.ai/api/v1/chat-threads/<threadId> \
  -H "Authorization: Bearer $ZERO_API_KEY"
```

**Response:** `200 OK` — thread metadata

---

### List Thread Messages

```bash
# Latest 50 messages
curl "https://api.vm0.ai/api/v1/chat-threads/<threadId>/messages" \
  -H "Authorization: Bearer $ZERO_API_KEY"

# Paginate — messages after a specific message ID
curl "https://api.vm0.ai/api/v1/chat-threads/<threadId>/messages?sinceId=<messageId>&limit=20" \
  -H "Authorization: Bearer $ZERO_API_KEY"
```

**Query parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `limit` | integer (1–100) | 50 | Max number of messages to return |
| `sinceId` | string (UUID) | — | Return only messages after this message ID |

**Response:** `200 OK` — array of messages

---

## Error Codes

| Status | Meaning |
|--------|---------|
| 401 | Missing or invalid API key — check `$ZERO_API_KEY` |
| 403 | Valid key but access denied to this resource |
| 404 | Thread or message not found |
| 400 | Bad request — check your request body |

## Usage Patterns

**Start a conversation and capture the thread ID:**

```bash
RESPONSE=$(curl -s -X POST https://api.vm0.ai/api/v1/chat-threads/messages \
  -H "Authorization: Bearer $ZERO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Hello"}')

THREAD_ID=$(echo $RESPONSE | jq -r '.threadId')
```

**Poll for new messages using sinceId:**

```bash
LAST_ID="<last-seen-message-id>"
curl -s "https://api.vm0.ai/api/v1/chat-threads/$THREAD_ID/messages?sinceId=$LAST_ID" \
  -H "Authorization: Bearer $ZERO_API_KEY" | jq '.messages'
```
