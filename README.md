# zero-use

Agent skills for the [Zero](https://vm0.ai) chat API.

## Install

```bash
npx skills add vm0-ai/zero-use
```

## Setup

Set your API key:

```bash
export ZERO_API_KEY=your_api_key_here
```

## Skills

| Skill | Description |
|-------|-------------|
| `zero-chat` | Send messages, manage threads, and list messages via the Zero API |

## Usage

Once installed, your agent can interact with the Zero API at `https://api.vm0.ai`. Ask it to:

- "Send a message to Zero: hello world"
- "List messages in thread `<uuid>`"
- "Continue this conversation in Zero thread `<uuid>`"
