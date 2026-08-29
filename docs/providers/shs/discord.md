# Discord Provider

Send messages to Discord servers using webhooks.

## How It Works

The Discord provider has one service, **Send Webhook Message**. It posts to the single
channel a webhook URL belongs to, with optional embeds for formatted content and an
override of the webhook's username and avatar. No Discord bot is required.

Studio does not receive from Discord, and it cannot edit or delete a message it has
sent, so a Discord message cannot start a workflow.

## Setup

Discord webhooks carry their own authentication in the URL, so there is no credential
to configure. The webhook URL is a parameter on the step.

1. Open Discord and go to the channel you want to post to
2. Click the gear icon (Edit Channel) > **Integrations**
3. Click **Webhooks** > **New Webhook**
4. Name the webhook and optionally set an avatar
5. Click **Copy Webhook URL**
6. In Studio, add a **Send Webhook Message** step and paste the URL into **Webhook URL**

The URL grants posting rights to that one channel. Treat it as a secret: anyone holding
it can post. Delete the webhook in Discord to revoke it.

## Available Services

### Send Webhook Message

Post a message via webhook (no bot required).

| Parameter | Description |
|-----------|-------------|
| **webhook_url** | The webhook URL copied from Discord |
| **content** | Message text (up to 2000 characters) |
| **username** | Override the webhook's default username |
| **avatar_url** | Override the webhook's default avatar |
| **embeds** | Rich embed objects for formatted content |

---

## Example Workflows

### System Alert Notifications

**Use case:** Post alerts when something needs attention

```
Trigger: Error detected
  ↓
Step 1: Discord Send Webhook
  - Content: "Alert: Server CPU at 95%"
  - Username: "System Monitor"
```

### Social Media Cross-Post

**Use case:** Share content to Discord when posted elsewhere

```
Trigger: New blog post
  ↓
Step 1: Discord Send Webhook
  - Content: "New post: {{ post.title }}\n{{ post.url }}"
```

### Daily Standup Reminder

**Use case:** Remind team of daily standup

```
Trigger: Schedule (daily at 9:55am)
  ↓
Step 1: Discord Send Webhook
  - Content: "@here Standup in 5 minutes!"
```

---

## Embed Examples

### Simple Embed

```json
{
  "embeds": [
    {
      "title": "New Order",
      "description": "Order #1234 received",
      "color": 5814783,
      "fields": [
        { "name": "Customer", "value": "John Doe", "inline": true },
        { "name": "Total", "value": "$99.99", "inline": true }
      ]
    }
  ]
}
```

### Rich Embed with Image

```json
{
  "embeds": [
    {
      "title": "{{ product.name }}",
      "description": "{{ product.description }}",
      "url": "{{ product.url }}",
      "color": 3447003,
      "thumbnail": {
        "url": "{{ product.image_url }}"
      },
      "author": {
        "name": "Product Alert",
        "icon_url": "https://example.com/icon.png"
      },
      "footer": {
        "text": "Posted via Studio"
      },
      "timestamp": "{{ now() }}"
    }
  ]
}
```

### Status Embed with Colors

```json
{
  "embeds": [
    {
      "title": "Build Status",
      "description": "{{ build.status }}",
      "color": "{{ build.success ? 5763719 : 15548997 }}"
    }
  ]
}
```

**Common Colors (decimal):**
- Green (success): `5763719`
- Red (error): `15548997`
- Blue (info): `3447003`
- Yellow (warning): `16776960`
- Purple: `10181046`

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Unknown Webhook" | Webhook was deleted - create a new one |
| "Invalid Webhook Token" | URL is malformed - copy fresh from Discord |
| "Request entity too large" | Content + embeds exceed 6000 characters |
| Rate limited | Max 30 messages/minute per webhook |

## Tips

1. **Rate Limits** - Discord allows 30 messages per minute per webhook
2. **Embed Limits** - Max 10 embeds per message, 6000 total characters
3. **Mentions** - Use `<@USER_ID>` to mention users, `@here` or `@everyone` for groups
4. **Markdown** - Discord supports markdown: `**bold**`, `*italic*`, `` `code` ``
5. **Multiple Webhooks** - Create different webhooks for different purposes

## Terms

Your use of Discord is governed by Discord's own terms, not by Studio's: [https://discord.com/terms](https://discord.com/terms). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
