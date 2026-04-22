---
name: zero-use
description: Use the Zero API (api.vm0.ai) to send messages to AI chat threads, retrieve thread details, and list messages. Triggers when the user asks to send messages to Zero, interact with the Zero chat API, or manage chat threads programmatically via API.
---

# Zero Chat API

Use this skill to interact with the Zero API at `https://api.vm0.ai`.

Use `skills/zero-use/zero-curl` instead of `curl` — it handles authentication automatically.

## API Endpoints

Base URL: `https://api.vm0.ai`

---

### Send a Message

Creates a new thread and sends the first message, or appends to an existing thread.

```bash
# New thread
skills/zero-use/zero-curl -X POST https://api.vm0.ai/api/v1/chat-threads/messages \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Your message here"}'

# Append to existing thread
skills/zero-use/zero-curl -X POST https://api.vm0.ai/api/v1/chat-threads/messages \
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
skills/zero-use/zero-curl https://api.vm0.ai/api/v1/chat-threads/<threadId>
```

**Response:** `200 OK` — thread metadata

---

### List Thread Messages

```bash
# Latest 50 messages
skills/zero-use/zero-curl "https://api.vm0.ai/api/v1/chat-threads/<threadId>/messages"

# Paginate — messages after a specific message ID
skills/zero-use/zero-curl "https://api.vm0.ai/api/v1/chat-threads/<threadId>/messages?sinceId=<messageId>&limit=20"
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
| 401 | Missing or invalid API key |
| 403 | Valid key but access denied to this resource |
| 404 | Thread or message not found |
| 400 | Bad request — check your request body |

## Usage Patterns

**Start a conversation and capture the thread ID:**

```bash
RESPONSE=$(skills/zero-use/zero-curl -s -X POST https://api.vm0.ai/api/v1/chat-threads/messages \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Hello"}')

THREAD_ID=$(echo $RESPONSE | jq -r '.threadId')
```

**Poll for new messages using sinceId:**

```bash
skills/zero-use/zero-curl -s "https://api.vm0.ai/api/v1/chat-threads/<thread-id>/messages?sinceId=<last-id>" | jq '.messages'
```
