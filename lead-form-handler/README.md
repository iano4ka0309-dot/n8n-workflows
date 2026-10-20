# Lead Form Handler (n8n)

Collects leads from a one-page Wix landing: every form submission is saved to Google Sheets and sent to Telegram, with no manual copying.

## How it works

```
Wix form submitted (Wix Automation → "Send HTTP request", POST)
   → n8n Webhook (POST)
   → Google Sheets: append row (date/time, name, phone, e-mail, comment)
   → Telegram: "New lead" message with the lead's data
```

- **Webhook** receives the Wix payload. The form fields arrive as an array `body.data.submissions` of `{ label, value }` objects.
- **Field lookup by label** (`submissions.find(s => s.label === 'Phone')?.value`) is used instead of array positions, because an empty optional field is missing from the array and would shift the positions.
- **`body.data.contact`** is Wix's own contact record, not what the visitor typed, so it is not used for the lead data.
- **Date/time** comes from `submissionTime` (UTC) and is converted to Kyiv time with Luxon.
- **Google Sheets** columns: `date/time`, `name`, `phone`, `e-mail`, `comment`.

## Tech stack

Wix (landing + Automations), n8n (Webhook, Google Sheets, Telegram).

## Known limitations

- The webhook has no authentication. Anyone who knows the URL can send fake leads. A secret header (Webhook → Header Auth, sent by the Wix HTTP request) would close this.
- Telegram `parse_mode` is set to HTML. Visitor text containing `<` or `&` can make Telegram reject the message. Plain text (no parse mode) is safer.
- No duplicate check: a repeated submission creates a second row.
- No error handling or retry if Google Sheets or Telegram is unavailable.

## Run your own copy

1. Import `workflows/lead-form-handler.json`.
2. Create credentials: Google Sheets OAuth2 and Telegram API. Replace `YOUR_CHAT_ID` with your chat ID.
3. Create a Google Sheet with the header row `date/time | name | phone | e-mail | comment` and select it in the "Append row in sheet" node (the document link is `REDACTED` in the export).
4. Set your own random path in the Webhook node, publish the workflow, and put its **Production URL** into the Wix automation (trigger "Form submitted", action "Send HTTP request", method POST).
5. Submit the form once and check the Executions tab, the sheet, and Telegram.

## Security note

Webhook path, webhook IDs, credential IDs, instance ID, chat ID and the sheet link are replaced with placeholders. All test data is fictional.
