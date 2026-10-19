# Translator Bot (n8n)

A Telegram bot that translates text into the language the user asks for. If the language, the text, or the intent is unclear, the bot asks a clarifying question instead of guessing.

## Workflows

| File | Role |
|---|---|
| `translator-bot-main.json` | Main bot: Telegram Trigger → AI Agent (with memory) → Code → Telegram reply |
| `translator-error-handler.json` | Error workflow with an Error Trigger node, connected to the main workflow in its settings |

## How it works

```
Telegram Trigger → AI Agent (system prompt + memory) → Code (parse JSON) → Send a text message
```

- The agent must answer with strict JSON: `{ "type": ..., "message": ... }`, where `type` is one of `translation`, `clarification_language`, `clarification_text`, `clarification_intent`.
- Four scenarios are covered: text and language present (translate), language missing (ask for it), text missing (ask for it), unclear intent (ask "should this be translated?").
- Simple Memory (last 6 messages) is keyed by the Telegram `chat.id`, so each chat has its own context. The prompt tells the agent to use history only to complete a translation that was explicitly started.
- The Code node parses the agent's JSON and passes only `message` to Telegram.

## Tech stack

n8n, Telegram Bot API, OpenRouter (LLM), Error Trigger.

## Known limitations

- The Code node uses `JSON.parse` without a fallback. If the model returns invalid JSON, the run fails and the error workflow is triggered.
- The error workflow contains only the Error Trigger node. Add a notification (for example a Telegram message) to make it useful.
- A free model is used, so quality and speed can vary.

## Run your own copy

1. Import both workflows. Import the error handler first.
2. In the main workflow settings, select the imported error handler as the Error Workflow (its ID is `REDACTED` in the export).
3. Create credentials for Telegram and OpenRouter, publish the main workflow, and message your bot.

## Security note

Credential IDs, instance ID, webhook IDs and the error workflow ID are replaced with placeholders.
