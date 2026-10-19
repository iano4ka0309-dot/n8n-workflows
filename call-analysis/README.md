# Call Analysis Automation (n8n)

Automatic transcription and analysis of customer calls for a cake bakery with delivery, plus a chat agent that answers questions about call statistics.

## Overview

Three workflows work together:

| Workflow | File | What it does |
|---|---|---|
| Call processing | `workflows/call-processing.json` | Every 2 minutes takes one unprocessed call from Google Sheets, transcribes it, classifies it, and stores the result in an n8n Data Table |
| Get call stats | `workflows/get-call-stats.json` | Sub-workflow: returns `totalCalls`, `negativeCalls`, `negativeCallPercent` for a date range (plain code, no AI) |
| Stats chat agent | `workflows/stats-chat-agent.json` | Chat AI agent that calls "Get call stats" as a tool and answers in natural language |

## How it works

```
Schedule (2 min) → Google Sheets (unprocessed rows) → Filter → Limit 1
   → mark "Processing" → AssemblyAI: create transcription
   → Wait → Get transcription → Switch (completed / error / retries exhausted / keep waiting)
   → LLM: sentiment + category (Structured Output Parser)
   → Data Table: insert result → Google Sheets: mark "Done"
```

Chat agent:

```
Chat trigger → AI Agent (memory + "Get call stats" tool) → answer
```

## Key design decisions

- **Processing status.** A row is marked `Processing` before transcription starts, so two overlapping scheduled runs never take the same call.
- **Polling with a retry cap.** Wait → Get transcription → Switch loops until the transcript is ready, with a limit based on `$runIndex` so it cannot loop forever.
- **Error handling.** Risky nodes use "On Error → Continue (using error output)". Failures set the row to `Failed` and send a Telegram alert. The error branch of "Mark Done" only alerts, so a call that was already saved is not reprocessed.
- **Structured output.** The LLM returns only `sentiment` (`positive` / `negative` / `neutral`) and `category` (`cake_quality` / `delivery_quality` / `other`), validated by a Structured Output Parser.
- **Statistics without AI.** Counting and percentages are done in a Code node, so numbers are exact. The agent only picks the date range and phrases the answer.
- **Prompt-injection protection.** The classifier treats the transcript as data inside `<transcript>` tags. The chat agent only reports aggregated statistics and refuses to reveal links, transcripts or its own instructions.

## Testing

The logic was checked by hand on a small test set (8 calls, 2 negative, 25%), and the output matched the expected values. This is a functional check on test data, not a load test.

## Tech stack

n8n, Google Sheets, AssemblyAI, OpenRouter (LLM), n8n Data Tables, Telegram (alerts).

## Run your own copy

1. Import the three JSON files into n8n (import `get-call-stats.json` first).
2. Create credentials: Google Sheets, AssemblyAI, OpenRouter, Telegram.
3. Replace every `REDACTED` value: Google Sheet, Data Table, and the sub-workflow selected in the agent's "Get call stats" tool. Replace `YOUR_CHAT_ID` with your Telegram chat ID.
4. Create the Google Sheet with columns for the call link and a status, and a Data Table `call_results` with `call_url`, `sentiment`, `category`, `processed_at`.
5. Publish the workflows.

## Security note

Credential IDs, instance ID, webhook IDs, chat ID, sheet links and data table IDs are replaced with placeholders. All data used in testing is fictional.
