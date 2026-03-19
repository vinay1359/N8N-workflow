# Automatic Google Meet Link from Telegram

This workflow lets you schedule a Google Calendar event from Telegram and automatically returns a Google Meet link in the same chat.

It supports both:
- Text messages in Telegram
- Voice messages in Telegram (voice is transcribed first)

After parsing the request, it creates a calendar event, adds attendees, enables Google Meet conferencing, and sends back the meeting URL.

## File in this folder

- `automatic gmeet link.json` - n8n workflow export

## How the workflow works

1. `Telegram Trigger` listens for new Telegram messages.
2. `Switch` routes incoming messages:
   - Audio path: voice message -> download file -> transcribe using Gemini
   - Text path: read message text directly
3. Both paths normalize data into a `text` field.
4. `AI Agent` + `Structured Output Parser` extracts:
   - `start_time`
   - `end_time`
   - `candidates` (attendee emails)
5. `Create an event` creates a Google Calendar event with Google Meet conference data.
6. `Send a text message` replies in Telegram with the Meet link.

## Requirements

Before setup, keep these ready:
- Running n8n instance
- Telegram bot token (from BotFather)
- Google Calendar OAuth2 credentials connected in n8n
- Google Gemini API key connected in n8n
- Calendar account where events should be created

## Setup steps

1. Open n8n.
2. Import workflow from `automatic gmeet link.json`.
3. Open each credentialed node and map your own credentials:
   - Telegram Trigger / Telegram nodes -> Telegram credential
   - Google Gemini nodes -> Gemini API credential
   - Google Calendar node -> Google Calendar OAuth2 credential
4. In `Create an event`, select your target calendar.
5. In `AI Agent`, review the system prompt and keep date/time parsing rules as needed.
6. Save and activate the workflow.
7. Send a test message to your Telegram bot.

## Example messages to send

Text example:
- Schedule a meeting tomorrow 4 pm to 5 pm with abc@gmail.com and xyz@gmail.com

Voice example:
- Send the same request as a voice note

## Expected output

Telegram should reply with a message like:
- Your meeting is scheduled, here is the meet link: https://meet.google.com/...

## Notes

- If `start_time` or `end_time` is missing, the workflow currently falls back to now and now + 1 hour.
- Attendees are read from the parsed `candidates` array.
- Timezone behavior depends on your n8n server timezone, calendar settings, and what the AI extracts.

## Troubleshooting

If no reply appears in Telegram:
- Confirm workflow is active.
- Verify Telegram credentials and bot token.
- Check bot permissions and whether the bot received the message.

If event is created but no Meet link:
- Confirm Google Calendar account allows Meet conference creation.
- Recheck `conferenceDataUi -> conferenceSolution = hangoutsMeet` in `Create an event`.

If date/time is wrong:
- Ask users to provide clear date and time with timezone when possible.
- Tighten AI prompt instructions and parser schema if needed.

If voice messages fail:
- Confirm Telegram file download works.
- Confirm Gemini audio transcription node is correctly authenticated.

## Security reminder

This exported workflow contains credential references by name and id. Before sharing publicly:
- Remove or replace sensitive identifiers
- Reconnect credentials in the target n8n environment
