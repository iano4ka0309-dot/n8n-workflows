# Task Manager Agent (n8n)

A Telegram bot where an AI agent manages a to-do list for a confectionery. The agent understands plain language ("add: order boxes", "ordered the boxes, mark it done") and calls three sub-workflows as tools.

## Workflows

| File | Role |
|---|---|
| `task-manager-agent.json` | Telegram Trigger → AI Agent (with memory and three tools) → reply in Telegram |
| `task-manager-list-tasks.json` | Tool: returns all unfinished tasks with their `id` |
| `task-manager-add-task.json` | Tool: inserts a new task (`task_description`, `is_finished = false`) |
| `task-manager-complete-task.json` | Tool: marks the task with the given `id` as finished |

All tools work with one n8n Data Table named `tasks`.

## Data Table `tasks`

| Column | Type |
|---|---|
| `id` | created automatically by n8n |
| `task_description` | string |
| `is_finished` | boolean |

## How the agent works

- Users name a task in words, not by `id`. To complete a task, the agent first calls `list-tasks`, finds the task that matches by meaning, and then calls `complete-task` with that `id`. The system prompt forbids inventing ids.
- If several tasks match, the agent lists them and asks which one. If none match, it says so and does not call `complete-task`.
- Requests not related to tasks get a polite refusal.

## Tech stack

n8n, Telegram Bot API, OpenRouter (LLM), n8n Data Tables, sub-workflows used as agent tools.

## Known limitations

- The Simple Memory node has no explicit session key. Set the key to the Telegram `chat.id` so that different users do not share one conversation history.
- Any Telegram user who finds the bot can change the list. Add a check of the allowed `chat.id` after the trigger.

## Run your own copy

1. Create the Data Table `tasks` with the columns above.
2. Import the three tool workflows, select your Data Table in each of them, and **publish** them.
3. Import `task-manager-agent.json`. The three tool workflow IDs are `REDACTED` in the export, so open each "Call ..." node and select the matching workflow.
4. Create credentials for Telegram and OpenRouter, publish the agent, and message your bot.

## Security note

Credential IDs, instance ID, webhook IDs, workflow IDs and the table ID are replaced with placeholders.
