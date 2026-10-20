# n8n Workflows

A collection of automation projects built with n8n: Telegram bots, Gmail, Google Sheets and Docs, AI agents, and sub-workflows used as agent tools.

## Projects

| Project | Description | Stack |
|---|---|---|
| [call-analysis](call-analysis/) | Call transcription, sentiment classification and a chat agent for statistics | n8n, AssemblyAI, LLM, Data Tables, Google Sheets |
| [bot-image-generator](bot-image-generator/) | Telegram bot that generates Instagram and Facebook images from a text command | n8n, Telegram, OpenAI |
| [cake-order-automation](cake-order-automation/) | Emails about cakes → AI extracts the order → Google Docs proposal → reply with a link | n8n, Gmail, LLM, Google Drive, Google Docs |
| [task-manager](task-manager/) | Telegram AI agent that manages a to-do list through three sub-workflow tools | n8n, Telegram, LLM, Data Tables |
| [translator-bot](translator-bot/) | Telegram translator that asks clarifying questions, with a separate error workflow | n8n, Telegram, LLM |
| [lead-form-handler](lead-form-handler/) | Wix landing form → Google Sheets row + Telegram notification for every new lead | n8n, Wix, Webhook, Google Sheets, Telegram |

Each folder contains a README, the exported workflows in `workflows/`, and screenshots where available.

## How to import a workflow

1. In n8n, open Workflows → Import from file.
2. Select a JSON file from a project's `workflows/` folder.
3. Create your own credentials and replace placeholders (`REDACTED`, `YOUR_CHAT_ID`).

## About the exports

All exports are sanitized: credential IDs, instance and webhook IDs, chat IDs, and links to private documents are replaced with placeholders. Test data is fictional.
