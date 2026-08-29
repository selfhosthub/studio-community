# Trello Provider

Automate project management workflows, create cards from external events, and sync tasks across systems with Trello boards.

## Authentication

Trello uses **API Key + Token** authentication. You'll generate both from your Trello account.

### Setup Overview

1. Get your API Key from Trello
2. Generate an API Token
3. Add credentials in Studio
4. Start automating Trello workflows

### Step 1: Get API Key

1. Go to [Trello Power-Ups Admin](https://trello.com/power-ups/admin)
2. Click **New** to create a new Power-Up (or use existing)
3. Give it a name (e.g., "Studio Integration")
4. Your **API Key** is displayed on the Power-Up page
5. Copy and save it

**Alternative direct link:**
- Visit: [https://trello.com/app-key](https://trello.com/app-key)
- Your API Key is shown at the top

### Step 2: Generate Token

1. On the same Power-Up page, find the **Token** section
2. Click the link that says "you can manually generate a Token"
3. Or visit: `https://trello.com/1/authorize?expiration=never&name=Studio&scope=read,write&response_type=token&key=YOUR_API_KEY`
   - Replace `YOUR_API_KEY` with your actual API key
4. Click **Allow** to authorize
5. Copy the **Token** that appears
6. Save it securely

### Step 3: Find Board and List IDs

**Board ID** from URL:
```
https://trello.com/b/BOARD_ID/board-name
```
The alphanumeric string after `/b/` is your Board ID.

**List ID** (use Get Lists service):
1. Use **Get Lists** service with your board ID
2. Find the list you want by name
3. Copy its `id` field

### Step 4: Add Credentials in Studio

1. Go to **Providers > Trello > Credentials**
2. Click **Add Credential**
3. Enter your **API Key**
4. Enter your **API Token**
5. Test the connection

## Available Services

### Board Management

| Service | Description |
|---------|-------------|
| **Get Board** | Retrieve board info, lists, cards, and members |
| **Get Lists** | Get all lists on a board to find list IDs |

### Card Management

| Service | Description |
|---------|-------------|
| **List Cards** | Get all cards from a board with filters |
| **Get Card** | Retrieve a single card's details |
| **Create Card** | Create new cards in lists |
| **Update Card** | Update card properties, move between lists, archive |

## Common Use Cases

### 1. Form Submission → Trello Card

Create cards from form submissions:

**Workflow:**
1. **Trigger**: Typeform/Google Forms webhook
2. **Parse** form fields
3. **Create Card** in Trello with form data
4. **Send Message** to Slack with card link

**Example:**
```json
{
  "idList": "list123",
  "name": "New Lead: John Doe",
  "desc": "Email: john@example.com\nCompany: Acme Corp\nInterest: Enterprise Plan",
  "due": "2024-01-31T23:59:59Z",
  "idLabels": "label_lead"
}
```

### 2. Email → Support Card

Create support cards from emails:

**Workflow:**
1. **Trigger**: Gmail new email (support inbox)
2. **Get Email** content
3. **Create Card** in Support board
4. **Update** email with card link

### 3. Daily Task Summary

Get daily summary of completed tasks:

**Workflow:**
1. **Trigger**: Scheduled daily
2. **List Cards** with filter "open"
3. **Filter** cards due today
4. **Send Message** to Telegram with task list

### 4. Card Status → Database Sync

Sync Trello cards to database:

**Workflow:**
1. **Trigger**: Scheduled every hour
2. **List Cards** from board
3. **For Each** card, check if exists in Airtable
4. **Create/Update** record in Airtable

### 5. Move Card on Event

Automate card movement:

**Workflow:**
1. **Trigger**: Stripe payment succeeded
2. **List Cards** to find customer card
3. **Update Card** to move to "Paid" list
4. Set `dueComplete: true`

### 6. Archive Completed Cards

Clean up old completed cards:

**Workflow:**
1. **Trigger**: Scheduled weekly
2. **List Cards** with `filter: "open"`
3. **Filter** cards with `dueComplete: true` older than 30 days
4. **Update Card** with `closed: true` for each

## Service Details

### Get Board

Retrieve board information:

```json
{
  "board_id": "abc123",
  "lists": "all",
  "cards": "open",
  "members": "all"
}
```

**Returns:**
- Board name, description, URL
- Lists (if requested)
- Cards (if requested)
- Members (if requested)

### Get Lists

Get all lists on a board:

```json
{
  "board_id": "abc123",
  "filter": "open"
}
```

**Returns array of lists:**
```json
[
  {
    "id": "list123",
    "name": "To Do",
    "closed": false,
    "pos": 1
  },
  {
    "id": "list456",
    "name": "In Progress",
    "closed": false,
    "pos": 2
  },
  {
    "id": "list789",
    "name": "Done",
    "closed": false,
    "pos": 3
  }
]
```

### List Cards

Get cards from a board:

```json
{
  "board_id": "abc123",
  "filter": "open",
  "fields": "name,desc,due,labels,idList"
}
```

**Filter options:**
- `"open"` - Only active cards
- `"closed"` - Only archived cards
- `"all"` - All cards
- `"visible"` - All visible cards

**Include additional data:**
```json
{
  "board_id": "abc123",
  "members": true,
  "attachments": true,
  "checklists": "all"
}
```

### Get Card

Retrieve single card:

```json
{
  "card_id": "card123",
  "members": true,
  "attachments": true,
  "checklists": "all"
}
```

**Returns:**
- Card name, description
- Due date and completion status
- Labels
- Assigned members
- Attachments
- Checklists with items

### Create Card

Create a new card:

```json
{
  "idList": "list123",
  "name": "New Task",
  "desc": "This is the card description.\n\n**Supports Markdown!**",
  "pos": "top",
  "due": "2024-01-31T23:59:59Z",
  "idMembers": "member1,member2",
  "idLabels": "label1,label2"
}
```

**Position options:**
- `"top"` - Add to top of list
- `"bottom"` - Add to bottom of list

**Markdown support in description:**
- **Bold**: `**text**`
- *Italic*: `*text*`
- Links: `[text](url)`
- Lists: `- item`
- Code: `` `code` ``

### Update Card

Update card properties:

**Move card to different list:**
```json
{
  "card_id": "card123",
  "idList": "list456"
}
```

**Mark as complete:**
```json
{
  "card_id": "card123",
  "dueComplete": true
}
```

**Archive card:**
```json
{
  "card_id": "card123",
  "closed": true
}
```

**Update multiple fields:**
```json
{
  "card_id": "card123",
  "name": "Updated Task Name",
  "desc": "New description",
  "due": "2024-02-15T23:59:59Z",
  "dueComplete": false,
  "idMembers": "member3,member4",
  "idLabels": "label3"
}
```

**Note:** Setting `idMembers` or `idLabels` **replaces** existing values (doesn't append).

## Finding IDs

### Board ID

From URL:
```
https://trello.com/b/aBcD1234/my-board
                      ^^^^^^^^
                      Board ID
```

Or add `.json` to any board URL and look for `"id"` field.

### List ID

Use **Get Lists** service:
```json
{
  "board_id": "aBcD1234"
}
```

Find list by name in response and copy its `id`.

### Card ID

From card URL:
```
https://trello.com/c/xyz789/card-name
                      ^^^^^^
                      Card ID (short)
```

Or use **List Cards** to find card by name.

### Member ID

From board member list or add `.json` to board URL and look for `idMembers` array.

### Label ID

From board labels or add `.json` to board URL and look for `labels` array.

## Labels

Trello supports colored labels:

| Color | Common Use |
|-------|-----------|
| Green | Complete, Approved |
| Yellow | In Progress, Waiting |
| Orange | Important, Priority |
| Red | Urgent, Blocked |
| Purple | Feature Request |
| Blue | Information, Note |
| Sky | Bug, Issue |
| Lime | Enhancement |
| Pink | Question |
| Black | Archived, Deprecated |

Get label IDs from board JSON or create labels in Trello UI first.

## Rate Limits

Trello API rate limits:
- **300 requests per 10 seconds** per token
- **100 requests per 10 seconds** per token per board

**Best practices:**
- Batch operations when possible
- Add delays for bulk updates
- Cache board/list structure (rarely changes)

## Webhooks

Trello supports webhooks for real-time updates:

**Create webhook** (requires separate endpoint):
```
POST https://api.trello.com/1/webhooks
```

**Webhook events:**
- Card created
- Card updated
- Card moved
- Card deleted
- Comment added

**Note:** Webhook creation not included in basic services. Use direct API calls or Trello Power-Up for webhook setup.

## Error Handling

### Common Errors

**"invalid id"**
- **Cause**: Invalid board, list, or card ID
- **Solution**: Verify ID format, use Get Lists/List Cards to find valid IDs

**"invalid token"**
- **Cause**: Token expired or revoked
- **Solution**: Generate new token from Trello

**"unauthorized permission requested"**
- **Cause**: Token doesn't have required scope
- **Solution**: Generate token with read+write scope

**"requested resource not found"**
- **Cause**: Card/board doesn't exist or you don't have access
- **Solution**: Check permissions and verify ID

## Best Practices

1. **Cache List IDs**: Store list IDs instead of querying repeatedly
2. **Use Short Links**: Trello card URLs can be shortened (keep 6-character ID)
3. **Markdown in Descriptions**: Use Markdown for formatted card descriptions
4. **Batch Updates**: Group multiple card updates together
5. **Handle Archived**: Check `closed` field to exclude archived cards
6. **Position Carefully**: Use `"top"` for urgent tasks, `"bottom"` for backlog
7. **Member/Label IDs**: Get these from board data, not hardcoded

## Security

1. **API Credentials:**
   - Never expose API key or token publicly
   - Regenerate token if compromised
   - Use read-only token if write access not needed

2. **Board Privacy:**
   - Tokens have same permissions as user
   - Only share boards with trusted members
   - Be careful with public boards

3. **Data Privacy:**
   - Don't store sensitive data in card descriptions
   - Use private boards for confidential information

## Troubleshooting

### Can't find board/list

**Solution:**
- Verify you're a member of the board
- Check board isn't archived
- Use board shortLink (from URL) instead of full ID

### Cards not updating

**Solution:**
- Verify card ID is correct
- Check if card is archived (`closed: true`)
- Ensure token has write permission

### Rate limit errors

**Solution:**
- Add delays between requests (300ms recommended)
- Use pagination for large boards
- Cache static data (board structure)

## Trello vs Other Tools

| Feature | Trello | Notion | Airtable |
|---------|--------|--------|----------|
| **Best for** | Kanban boards | Wikis + databases | Structured data |
| **Card structure** | Simple | Flexible pages | Rich fields |
| **Automation** | Power-Ups | Limited | Strong |
| **API** | Mature | Growing | Excellent |

## Additional Resources

- [Trello API Documentation](https://developer.atlassian.com/cloud/trello/rest/)
- [API Reference](https://developer.atlassian.com/cloud/trello/rest/api-group-cards/)
- [Authentication Guide](https://developer.atlassian.com/cloud/trello/guides/rest-api/authorization/)
- [Power-Ups Admin](https://trello.com/power-ups/admin)
- [Get API Key](https://trello.com/app-key)

## Terms

Your use of Trello is governed by Trello's own terms, not by Studio's: [https://www.atlassian.com/legal](https://www.atlassian.com/legal). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
