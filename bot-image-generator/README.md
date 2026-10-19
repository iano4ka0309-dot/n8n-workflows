# Image Generator Telegram Bot (n8n)

A Telegram bot that generates images for social networks from a short text command.

## Usage

| Message | Result |
|---|---|
| `insta <description>` | Vertical image, 1024x1536 (Instagram) |
| `fb <description>` | Horizontal banner, 1536x1024 (Facebook) |
| anything else, or a command without a description | Help message with examples |

Command words also work in Ukrainian and Russian spellings (`інста`, `инста`, `фб`).

## How it works

```
Telegram Trigger → Code (parse message) → Switch (insta / fb / help)
   insta, fb → OpenAI "Generate an image" → Telegram "Send a photo"
   help      → Telegram "Send a text message"
```

- The **Code node** extracts `route`, `size`, `prompt` and `chatId`. It matches the command as a whole word (Unicode-aware regex, so Cyrillic works) and sends empty prompts to the help branch.
- The **Switch node** routes by `route`.
- The **OpenAI node** takes its prompt and size from the Code node via expressions, so each branch produces the right orientation.
- The generated image is returned as binary data and sent back with the Telegram photo node.

## Tech stack

n8n, Telegram Bot API, OpenAI image generation.

## Run your own copy

1. Import `workflows/bot-image-generator.json`.
2. Create credentials: Telegram (token from @BotFather) and OpenAI. Make sure the OpenAI account has available balance.
3. Replace the `REDACTED` placeholders if any remain, publish the workflow, and send a message to your bot.

## Security note

Credential IDs, instance ID and webhook IDs are replaced with placeholders. Never commit a real bot token or API key.
