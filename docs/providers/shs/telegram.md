# Telegram Provider

Send messages, photos, documents, and locations via Telegram bot API for notifications and automation.

## Authentication

Telegram uses **Bot Token** authentication. You'll need to create a bot with @BotFather to get your token.

### Setup Overview

1. Create a Telegram bot with @BotFather
2. Get your bot token
3. Add bot to your channel/group (optional)
4. Add credential in Studio with your bot token

### Step 1: Create Telegram Bot

1. Open Telegram and search for [@BotFather](https://t.me/botfather)
2. Send `/newbot` command
3. Follow the prompts:
   - **Bot name**: Your bot's display name (e.g., "Studio Notifications")
   - **Username**: Bot username, must end in "bot" (e.g., "studio_notif_bot")
4. @BotFather will send you a **bot token** (looks like `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)
5. **Save this token** - you'll need it for Studio

### Step 2: Get Chat ID

To send messages, you need the Chat ID of the recipient:

**For personal chat:**
1. Send a message to your bot
2. Visit: `https://api.telegram.org/bot<YourBotToken>/getUpdates`
3. Look for `"chat":{"id":123456789}` in the response
4. That number is your Chat ID

**For channels:**
1. Add your bot as an administrator to the channel
2. Post a message in the channel
3. Visit: `https://api.telegram.org/bot<YourBotToken>/getUpdates`
4. Look for `"chat":{"id":-1001234567890}` (negative number for channels)
5. Or use the channel username: `@yourchannel`

**For groups:**
1. Add your bot to the group
2. Send a message in the group
3. Visit the getUpdates URL above
4. Look for the chat ID (negative number for groups)

### Step 3: Add Credential in Studio

1. Go to **Providers > Telegram > Credentials**
2. Click **Add Credential**
3. Paste your **Bot Token** from @BotFather
4. Test the connection

## Available Services

### Messaging

| Service | Description |
|---------|-------------|
| **Send Message** | Send text messages with Markdown/HTML formatting |
| **Send Photo** | Send images with captions |
| **Send Document** | Send files (PDF, ZIP, etc.) |
| **Send Location** | Send geographic coordinates |

### Updates

| Service | Description |
|---------|-------------|
| **Get Updates** | Poll for incoming messages and events (alternative to webhooks) |

## Common Use Cases

### 1. Workflow Notifications

Send alerts when workflows complete, fail, or require approval:

**Workflow:**
1. **Trigger**: Workflow completion event
2. **Send Message** to Telegram
3. Include workflow name, status, and timestamp

**Example Message:**
```
✅ Workflow Complete

Name: Customer Onboarding
Status: Success
Duration: 2.5 minutes
Time: 2024-01-15 14:32 UTC
```

### 2. Error Alerts

Get notified when something goes wrong:

**Workflow:**
1. **Trigger**: Workflow failure event
2. **Send Message** with error details
3. Tag responsible team members

**Example:**
```
🚨 Workflow Error

Workflow: Payment Processing
Error: API timeout after 30s
Step: charge_customer
Retry: Automatic in 5 minutes
```

### 3. Form Submissions → Telegram

Forward form submissions to your team:

**Workflow:**
1. **Trigger**: Typeform/Google Forms submission
2. **Send Message** with submission details
3. **Send Photo** if form includes image upload

### 4. Customer Support Bot

Send automated responses:

**Workflow:**
1. **Get Updates** to poll for new messages
2. **Parse** message content
3. **Send Message** with automated response or ticket number

### 5. Daily Reports

Schedule daily summaries:

**Workflow:**
1. **Trigger**: Scheduled daily at 9 AM
2. **Query Database**: Get stats from Airtable/Notion
3. **Send Message** with formatted report

**Example:**
```
📊 Daily Report - Jan 15

New Customers: 42
Revenue: $3,245
Support Tickets: 18 (3 open)
Server Uptime: 99.97%
```

### 6. Location Tracking

Send location updates:

**Workflow:**
1. **Trigger**: GPS coordinate update
2. **Send Location** to Telegram
3. Add live tracking for moving assets

## Service Details

### Send Message

Send formatted text messages:

```json
{
  "chat_id": "123456789",
  "text": "Hello from Studio! 👋",
  "parse_mode": "Markdown"
}
```

**Formatting Options:**

**Markdown:**
```
*bold text*
_italic text_
[link text](http://example.com)
`inline code`
```

**HTML:**
```html
<b>bold</b>
<i>italic</i>
<a href="http://example.com">link</a>
<code>code</code>
```

**Advanced Options:**
- `disable_web_page_preview`: Disable link previews
- `disable_notification`: Silent message (no sound)
- `protect_content`: Prevent forwarding/saving
- `reply_to_message_id`: Reply to specific message
- `reply_markup`: Add buttons/keyboard

### Send Photo

Send images with captions:

```json
{
  "chat_id": "123456789",
  "photo": "https://example.com/image.jpg",
  "caption": "Check out this screenshot!",
  "parse_mode": "Markdown"
}
```

**Photo Sources:**
- **HTTP URL**: Telegram downloads from URL
- **File ID**: Reuse photo already on Telegram servers

**Limitations:**
- Max photo size: 10 MB
- Caption max length: 1024 characters

### Send Document

Send files:

```json
{
  "chat_id": "123456789",
  "document": "https://example.com/report.pdf",
  "caption": "Monthly sales report"
}
```

**Supported Formats:**
- PDF, ZIP, CSV, TXT
- Excel (.xlsx), Word (.docx)
- Any file type (max 50 MB for bots)

### Send Location

Send geographic coordinates:

```json
{
  "chat_id": "123456789",
  "latitude": 37.7749,
  "longitude": -122.4194
}
```

**Live Location:**
Set `live_period` (60-86400 seconds) for real-time tracking:
```json
{
  "chat_id": "123456789",
  "latitude": 37.7749,
  "longitude": -122.4194,
  "live_period": 3600,
  "heading": 45
}
```

### Get Updates

Poll for incoming messages:

```json
{
  "offset": 0,
  "limit": 100,
  "timeout": 30,
  "allowed_updates": ["message", "callback_query"]
}
```

**How it works:**
1. Call `getUpdates` with optional timeout for long polling
2. Process received updates
3. Call again with `offset` = highest `update_id + 1`

**Update Types:**
- `message`: New messages
- `edited_message`: Edited messages
- `callback_query`: Inline button clicks
- `channel_post`: New channel posts

## Reply Markup (Buttons)

Add interactive buttons to messages:

### Inline Keyboard (URL buttons)

```json
{
  "reply_markup": {
    "inline_keyboard": [
      [
        {
          "text": "View Workflow",
          "url": "https://studio.example.com/workflows/123"
        },
        {
          "text": "Approve",
          "callback_data": "approve_123"
        }
      ]
    ]
  }
}
```

### Custom Keyboard

```json
{
  "reply_markup": {
    "keyboard": [
      ["📊 Stats", "📝 Report"],
      ["⚙️ Settings"]
    ],
    "resize_keyboard": true,
    "one_time_keyboard": true
  }
}
```

## Finding Chat IDs

### Method 1: getUpdates API

1. Send a message to your bot or post in your channel
2. Visit: `https://api.telegram.org/bot<TOKEN>/getUpdates`
3. Find the chat ID in the response

### Method 2: Studio Workflow

Create a workflow with **Get Updates** service:
1. Add **Get Updates** step
2. Run workflow
3. Check results for chat IDs

### Chat ID Format

| Type | ID Format | Example |
|------|-----------|---------|
| Personal | Positive integer | `123456789` |
| Channel | Negative with -100 prefix | `-1001234567890` |
| Channel (username) | @username | `@mychannel` |
| Group | Negative integer | `-987654321` |

## Rate Limits

Telegram Bot API limits:
- **20 messages per second** to the same chat
- **30 messages per second** total (across all chats)
- **Bots in groups**: 1 message per second per group

**Best practices:**
- Add delays between bulk messages
- Use message batching for multiple recipients
- Implement retry logic with exponential backoff

## Error Handling

### Common Errors

**"Chat not found"**
- **Cause**: Invalid chat ID or bot blocked by user
- **Solution**: Verify chat ID, ensure user hasn't blocked bot

**"Forbidden: bot was blocked by the user"**
- **Cause**: User blocked your bot
- **Solution**: Remove user from notification list

**"Bad Request: message is too long"**
- **Cause**: Message exceeds 4096 characters
- **Solution**: Split into multiple messages

**"Bad Request: wrong file identifier/HTTP URL specified"**
- **Cause**: Invalid photo URL or file_id
- **Solution**: Verify URL is accessible, use HTTPS

## Best Practices

1. **Store Chat IDs**: Save chat IDs when users first interact with your bot
2. **Use Markdown/HTML**: Format messages for better readability
3. **Disable Notifications**: Use `disable_notification: true` for non-urgent updates
4. **Handle Errors Gracefully**: Check for blocked users, invalid IDs
5. **Use Buttons**: Add inline keyboards for actionable notifications
6. **Test in Private Chat**: Test workflows in personal chat before broadcasting to channels
7. **Respect Rate Limits**: Add delays for bulk messages

## Security

1. **Protect Your Token**: Never expose bot token in public repositories
2. **Use HTTPS**: Always use HTTPS URLs for webhooks and media
3. **Validate Input**: Sanitize user input before sending to Telegram
4. **Protect Content**: Use `protect_content: true` for sensitive information
5. **Bot Permissions**: Only grant necessary admin rights in channels/groups

## Troubleshooting

### Bot not receiving messages in group

**Solution:**
1. Disable Privacy Mode in @BotFather:
   - Send `/setprivacy` to @BotFather
   - Select your bot
   - Choose **Disable**
2. Or mention the bot in messages: `@yourbot hello`

### Messages not formatting correctly

**Solution:**
- Check `parse_mode` is set correctly ("Markdown", "MarkdownV2", or "HTML")
- Escape special characters in MarkdownV2
- Use HTML for complex formatting

### getUpdates returns empty array

**Solution:**
- Send a test message to your bot first
- Check if webhook is enabled (conflicts with polling)
- Disable webhook: `https://api.telegram.org/bot<TOKEN>/deleteWebhook`

## Additional Resources

- [Telegram Bot API Documentation](https://core.telegram.org/bots/api)
- [Bot API Reference](https://core.telegram.org/bots/api#available-methods)
- [Formatting Options](https://core.telegram.org/bots/api#formatting-options)
- [@BotFather Commands](https://core.telegram.org/bots#6-botfather)
- [Telegram API Limits](https://core.telegram.org/bots/faq#my-bot-is-hitting-limits-how-do-i-avoid-this)

## Terms

Your use of Telegram is governed by Telegram's own terms, not by Studio's: [https://telegram.org/tos](https://telegram.org/tos). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
