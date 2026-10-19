# Cake Order Automation (n8n)

Turns incoming customer emails about cakes into a ready commercial proposal. The workflow reads the email, extracts the order with an AI agent, fills in a Google Docs template, and replies to the customer with a link.

## How it works

```
Gmail Trigger (every minute, query: subject:торт)
   → Edit Fields (combine subject + body, keep sender)
   → AI Agent (is_order_request, cake_count)  ┐
   → Merge (original fields + agent output)   ┘
   → If (is_order_request = true)
   → Code in Python (cake_count, total_price, client_email)
   → Google Drive: copy proposal template
   → Google Docs: replace {{total_price}} and {{cake_count}}
   → Gmail: reply with a link to the new document
```

- **AI Agent** uses a Structured Output Parser with the schema `{ "is_order_request": true, "cake_count": 2 }`. If the email is not an order, `cake_count` must be 0.
- **Merge (combine all)** is needed because the AI Agent returns only its own `output` object, so the original email fields would be lost.
- **Code in Python** computes `total_price = cake_count * 1000`.
- **Google Docs** replaces the placeholders in the copied template. Values are converted with `.toString()` because the Docs API expects strings.

## Tech stack

n8n, Gmail, OpenRouter (LLM), Google Drive, Google Docs, Python Code node.

## Known limitations

This is a learning project, and the following points are known:

- The If node runs before the normalization step and reads `output.is_order_request` directly. A cleaner order is AI Agent → Code → If, with the If checking `is_order_request` at the top level.
- The trigger reacts to a matching email from **any** sender. For production, add a sender allowlist or a rate limit.
- The agent input is the raw email text. A stricter system prompt (treat the email as data) and a maximum for `cake_count` would make it safer.
- The price is a fixed constant (1000 per cake), not read from a price list.
- The copied document is not shared automatically. Add a Drive "Share" step if the customer needs access.
- A small free model is used for extraction, which is fine for a demo but not for reliable production use.

## Run your own copy

1. Import `workflows/cake-order-automation.json`.
2. Create credentials: Gmail OAuth2, Google Drive OAuth2, Google Docs OAuth2, OpenRouter.
3. Create a Google Docs proposal template containing `{{total_price}}` and `{{cake_count}}`, and select it in the "Copy file" node (the template link is `REDACTED` in the export).
4. Send a test email with "торт" in the subject, and check the Executions tab.

## Security note

Credential IDs, instance ID, webhook ID and the template link are replaced with placeholders. All test data is fictional.
