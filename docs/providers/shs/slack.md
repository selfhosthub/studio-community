# Slack Provider

Send messages to Slack channels and users from your workflows.

## How It Works

The Slack provider has one service, **Send Message**. It posts to any channel the bot
has been invited to, optionally as a reply in a thread, and the message body can carry
Block Kit blocks for rich formatting.

Studio does not receive from Slack. There is no Events API listener, slash command
handler or interactivity endpoint, so a Slack message or button press cannot start a
workflow.

## Setup

Slack uses **OAuth2**. You create a Slack app in your own workspace, give Studio its
client ID and secret, and authorize once. Studio holds the resulting token.

### Step 1: Create the Slack app

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **Create New App** > **From scratch**
3. Name the app and pick your workspace

### Step 2: Add the redirect URL

1. Open **OAuth & Permissions**
2. Under **Redirect URLs**, click **Add New Redirect URL** and enter:
   ```
   https://your-studio-domain.com/api/v1/oauth/slack/callback
   ```
   Replace `your-studio-domain.com` with your Studio API domain.
3. Click **Add**, then **Save URLs**

### Step 3: Add the bot scopes

On the same page, under **Bot Token Scopes**, add:

- `chat:write` - send messages
- `channels:read` - list channels
- `files:write` - attach files

### Step 4: Copy the app credentials

Open **Basic Information** > **App Credentials** and copy the **Client ID** and
**Client Secret**.

### Step 5: Connect in Studio

1. Go to **Providers > Slack > Credentials** and add a credential
2. Paste the Client ID and Client Secret, then save
3. Click **Authorize**, pick your workspace, and approve

Slack redirects back and stores the token. The credential is ready.

### Step 6: Invite the bot

In Slack, run `/invite @YourAppName` in each channel the workflow posts to.

A Slack bot token does not expire and Slack issues no refresh token for it unless
you enable token rotation, so the credential keeps working without renewal.

## Available Services

### Send Message

Post a message to a channel or user.

| Parameter | Description |
|-----------|-------------|
| **channel** | Channel name (#general) or ID (C1234567890) |
| **text** | Message content (supports markdown) |
| **blocks** | Optional: Block Kit blocks for rich formatting |
| **thread_ts** | Optional: Reply to this message's timestamp |

**Slack Markdown:**
- `*bold*` for bold
- `_italic_` for italic
- `` `code` `` for inline code
- `> quote` for block quotes
- `<https://url|Link text>` for links
- `<@U1234567890>` to mention users

---

## Example Workflows

### Error Alert Notifications

**Use case:** Alert your team when workflows fail

```
Trigger: Workflow error
  ↓
Step 1: Slack Send Message
  - Channel: #alerts
  - Text: "Workflow failed: {{ error.message }}"
```

### Daily Summary Reports

**Use case:** Post daily metrics to a channel

```
Trigger: Schedule (daily at 9am)
  ↓
Step 1: Query database for metrics
  ↓
Step 2: Slack Send Message
  - Channel: #metrics
  - Text: "Daily Report: {{ step1.total }} orders, ${{ step1.revenue }} revenue"
```

### Form Submission Notification

**Use case:** Notify team of new submissions

```
Trigger: Webhook (form)
  ↓
Step 1: Slack Send Message
  - Channel: #leads
  - Text: "New lead from {{ trigger.name }} ({{ trigger.email }})"
```

---

## Block Kit Examples

### Simple Message with Button

```json
{
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "New request from *{{ user.name }}*"
      }
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": { "type": "plain_text", "text": "Approve" },
          "style": "primary",
          "action_id": "approve"
        },
        {
          "type": "button",
          "text": { "type": "plain_text", "text": "Reject" },
          "style": "danger",
          "action_id": "reject"
        }
      ]
    }
  ]
}
```

### Status Update Card

```json
{
  "blocks": [
    {
      "type": "header",
      "text": { "type": "plain_text", "text": "Order Update" }
    },
    {
      "type": "section",
      "fields": [
        { "type": "mrkdwn", "text": "*Order:*\n#{{ order.id }}" },
        { "type": "mrkdwn", "text": "*Status:*\n{{ order.status }}" }
      ]
    }
  ]
}
```

## Troubleshooting

| Error | Solution |
|-------|----------|
| "channel_not_found" | Bot must be invited to private channels |
| "not_in_channel" | Add bot to channel with `/invite @botname` |
| "invalid_auth" | Re-authorize the credential in Studio |
| "missing_scope" | Add the scope in **Bot Token Scopes**, reinstall the app, then re-authorize |

## Tips

1. **Invite Bot** - Bot must be added to private channels before posting
2. **Channel IDs** - Use channel IDs for reliability (names can change)
3. **Rate Limits** - Slack allows ~1 message/second per channel
4. **Formatting** - Use Block Kit for rich, interactive messages
5. **Threads** - Use thread_ts to keep conversations organized

## Terms

Your use of Slack is governed by Slack's own terms, not by Studio's: [https://slack.com/terms-of-service/api](https://slack.com/terms-of-service/api). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
