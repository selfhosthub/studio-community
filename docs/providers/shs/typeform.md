# Typeform Provider

Automate form submission workflows, sync responses to databases, and trigger actions based on form completions.

## Authentication

Typeform uses **Personal Access Token** authentication. You'll create a token in your Typeform account settings.

### Setup Overview

1. Log into your Typeform account
2. Generate a Personal Access Token
3. Add credential in Studio with your token
4. Start automating form workflows

### Step 1: Create Personal Access Token

1. Go to [Typeform](https://www.typeform.com/) and log in
2. Click your profile icon in the top right
3. Select **Settings** from the dropdown
4. In the left sidebar, click **Personal tokens**
5. Click **Generate a new token**
6. Give it a name (e.g., "Studio Integration")
7. Select the scopes you need:
   - **forms:read** - Read form definitions
   - **responses:read** - Read form submissions
   - **webhooks:write** - Create/manage webhooks
8. Click **Generate token**
9. **Copy and save** the token immediately (you won't see it again)

### Step 2: Find Your Form ID

1. Go to your Typeform dashboard
2. Open the form you want to automate
3. Look at the URL: `https://admin.typeform.com/form/FORM_ID/create`
4. The `FORM_ID` is the alphanumeric string in the URL
5. Or click **Share** and look at the form URL: `https://yourname.typeform.com/to/FORM_ID`

### Step 3: Add Credential in Studio

1. Go to **Providers > Typeform > Credentials**
2. Click **Add Credential**
3. Paste your **Personal Access Token**
4. Test the connection

## Available Services

### Form Management

| Service | Description |
|---------|-------------|
| **Get Form** | Retrieve form structure, fields, and logic |

### Response Management

| Service | Description |
|---------|-------------|
| **List Responses** | Get form submissions with filtering and pagination |
| **Get Response** | Retrieve a single response by token |

### Webhooks

| Service | Description |
|---------|-------------|
| **Create Webhook** | Set up real-time notifications for new submissions |
| **Delete Webhook** | Remove a webhook |

## Common Use Cases

### 1. Form Submission → Database

Automatically save form responses to Airtable or Notion:

**Workflow:**
1. **Trigger**: Typeform webhook (new submission)
2. **Get Response** to get full details
3. **Create Record** in Airtable/Notion
4. **Send Email** confirmation

### 2. Lead Capture → CRM

Send new leads to your CRM:

**Workflow:**
1. **Trigger**: Typeform webhook
2. **Parse** form fields (name, email, company)
3. **Create Customer** in Stripe or HubSpot
4. **Send Message** to Slack with lead details

### 3. Survey Results → Report

Daily summary of survey responses:

**Workflow:**
1. **Trigger**: Scheduled daily
2. **List Responses** from last 24 hours
3. **Calculate** statistics/aggregations
4. **Send Message** to Telegram with summary

### 4. Contact Form → Support Ticket

Create support tickets from contact forms:

**Workflow:**
1. **Trigger**: Typeform webhook
2. **Get Response** details
3. **Create Task** in project management tool
4. **Send Email** to support team

### 5. Registration → Calendar Event

Schedule appointments from booking forms:

**Workflow:**
1. **Trigger**: Typeform webhook
2. **Parse** date/time from submission
3. **Create Event** in Google Calendar
4. **Send Email** with calendar invite

### 6. Poll Responses for Processing

Periodically check for new submissions:

**Workflow:**
1. **Trigger**: Scheduled every 15 minutes
2. **List Responses** since last check
3. **For Each** response, process data
4. **Update** tracking database

## Service Details

### Get Form

Retrieve form structure:

```json
{
  "form_id": "aBcD1234"
}
```

**Returns:**
- Form title and settings
- Field definitions with types
- Logic jumps and calculations
- Theme and design settings

**Use to:**
- Understand form structure before processing responses
- Dynamically map form fields to database columns
- Validate incoming webhook data

### List Responses

Get form submissions:

```json
{
  "form_id": "aBcD1234",
  "page_size": 100,
  "completed": true,
  "sort": "submitted_at,desc"
}
```

**Filter by date range:**
```json
{
  "form_id": "aBcD1234",
  "since": "2024-01-01T00:00:00Z",
  "until": "2024-01-31T23:59:59Z"
}
```

**Pagination:**
```json
{
  "form_id": "aBcD1234",
  "page_size": 25,
  "after": "pagination_token_from_previous_response"
}
```

**Returns:**
- `total_items`: Total number of responses
- `page_count`: Number of pages
- `items[]`: Array of responses with:
  - `landing_id`: Unique response ID
  - `token`: Response token
  - `submitted_at`: Submission timestamp
  - `answers[]`: Array of field answers

### Get Response

Retrieve single response:

```json
{
  "form_id": "aBcD1234",
  "response_token": "abc123def456"
}
```

**Returns:**
Complete response object with all answers and calculated fields.

### Create Webhook

Set up real-time notifications:

```json
{
  "form_id": "aBcD1234",
  "tag": "studio_production",
  "url": "https://studio.example.com/webhooks/typeform",
  "enabled": true
}
```

**Webhook payload** (what Typeform sends):
```json
{
  "event_id": "event_123",
  "event_type": "form_response",
  "form_response": {
    "form_id": "aBcD1234",
    "token": "response_token",
    "landed_at": "2024-01-15T10:30:00Z",
    "submitted_at": "2024-01-15T10:32:15Z",
    "definition": {
      "id": "aBcD1234",
      "title": "Contact Form",
      "fields": [...]
    },
    "answers": [
      {
        "type": "text",
        "text": "John Doe",
        "field": {
          "id": "field_1",
          "type": "short_text",
          "ref": "name"
        }
      },
      {
        "type": "email",
        "email": "john@example.com",
        "field": {
          "id": "field_2",
          "type": "email",
          "ref": "email"
        }
      }
    ]
  }
}
```

**Tag usage:**
- Use meaningful tags for organization (e.g., "production", "staging")
- One webhook per tag per form
- Use tag to delete/update webhooks later

### Delete Webhook

Remove a webhook:

```json
{
  "form_id": "aBcD1234",
  "tag": "studio_production"
}
```

## Response Field Types

Typeform supports various field types. Here's how answers are structured:

### Short Text / Long Text
```json
{
  "type": "text",
  "text": "User's text answer"
}
```

### Email
```json
{
  "type": "email",
  "email": "user@example.com"
}
```

### Multiple Choice
```json
{
  "type": "choice",
  "choice": {
    "label": "Option A"
  }
}
```

### Multiple Choice (Multiple Select)
```json
{
  "type": "choices",
  "choices": {
    "labels": ["Option A", "Option B"]
  }
}
```

### Number
```json
{
  "type": "number",
  "number": 42
}
```

### Date
```json
{
  "type": "date",
  "date": "2024-01-15"
}
```

### Rating / Opinion Scale
```json
{
  "type": "number",
  "number": 5
}
```

### File Upload
```json
{
  "type": "file_url",
  "file_url": "https://typeform.com/files/..."
}
```

## Webhook Setup Best Practices

1. **Test First**: Use a webhook testing tool (e.g., webhook.site) before production
2. **HTTPS Only**: Typeform requires HTTPS URLs for webhooks
3. **Verify SSL**: Keep `verify_ssl: true` for security
4. **Use Tags**: Organize webhooks with descriptive tags
5. **Handle Retries**: Typeform retries failed webhook deliveries
6. **Respond Quickly**: Return 200 OK within 10 seconds
7. **Process Async**: Queue webhook payloads for background processing

## Rate Limits

Typeform API rate limits:
- **1 request per second** per token (average)
- **Burst**: Up to 100 requests in quick succession

**Best practices:**
- Add delays between bulk operations
- Use pagination for large result sets
- Cache form definitions (they don't change often)

## Webhook Verification

Typeform includes a signature in webhook headers for verification:

**Header:** `Typeform-Signature`

Verify signature using HMAC-SHA256 with your webhook secret (optional feature in Typeform settings).

## Error Handling

### Common Errors

**"Invalid form id"**
- **Cause**: Form doesn't exist or wrong ID
- **Solution**: Verify form ID from Typeform admin URL

**"Forbidden"**
- **Cause**: Token doesn't have required scope
- **Solution**: Regenerate token with correct scopes

**"Response not found"**
- **Cause**: Invalid response token
- **Solution**: Verify response token from webhook or list responses

**"Webhook already exists"**
- **Cause**: Tag already used for this form
- **Solution**: Use different tag or delete existing webhook first

## Best Practices

1. **Use Webhooks**: For real-time processing, webhooks are more efficient than polling
2. **Store Response Tokens**: Save tokens for future lookups
3. **Handle Partial Submissions**: Check `completed` status in responses
4. **Map by Field Ref**: Use field `ref` (reference) instead of `id` for stability
5. **Test Webhooks**: Always test webhook URLs before going live
6. **Graceful Degradation**: Have fallback polling if webhook fails
7. **Deduplicate**: Use `landing_id` or `token` to avoid processing duplicates

## Security

1. **Token Security:**
   - Never expose token in public repos
   - Regenerate if compromised
   - Use minimum required scopes

2. **Webhook Security:**
   - Use HTTPS only
   - Verify signatures (if enabled)
   - Validate payload structure
   - Rate limit webhook endpoint

3. **Data Privacy:**
   - Handle personal data according to GDPR/privacy laws
   - Delete responses when no longer needed
   - Secure webhook endpoints

## Troubleshooting

### Webhook not receiving data

**Check:**
1. Webhook URL is HTTPS and accessible publicly
2. Webhook is enabled (`enabled: true`)
3. Form has been submitted (webhooks fire on new submissions only)
4. Check Typeform webhook logs in admin panel
5. Verify SSL certificate is valid

### Pagination not working

**Solution:**
- Use `after` token from response for next page
- Don't mix `after`/`before` with `since`/`until`
- Check `page_count` to know if more pages exist

### Missing answers in response

**Solution:**
- Check if response is completed (`completed: true`)
- Some fields may be optional and skipped by users
- Hidden fields won't appear in answers array

## Pricing

Typeform pricing affects API access:
- **Basic**: Limited API access
- **Plus**: Full API access
- **Business**: Full API + webhooks
- **Enterprise**: Higher rate limits

Check [Typeform Pricing](https://www.typeform.com/pricing/) for your plan's API features.

## Additional Resources

- [Typeform API Documentation](https://developer.typeform.com/)
- [Webhooks Guide](https://developer.typeform.com/webhooks/)
- [Responses API Reference](https://developer.typeform.com/responses/)
- [Authentication Guide](https://developer.typeform.com/get-started/)
- [Admin Panel](https://admin.typeform.com/)

## Terms

Your use of Typeform is governed by Typeform's own terms, not by Studio's: [https://www.typeform.com/legal/](https://www.typeform.com/legal/). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
