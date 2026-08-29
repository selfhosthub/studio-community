# Notion Provider

Connect to Notion workspaces for pages, databases, and knowledge management automation.

## Authentication

Notion uses an **Internal Integration Secret** you paste into Studio. Each organization creates its own Notion integration, giving you full control over which pages and databases Studio can access.

### Setup Overview

1. Create a Notion integration in your workspace
2. Share pages/databases with your integration
3. Add a credential in Studio and paste the Internal Integration Secret

### Step 1: Create Notion Integration

1. Go to [My Integrations](https://www.notion.so/my-integrations)
2. Click **+ New integration**
3. Fill in details:
   - **Name**: Your integration name (e.g., "Studio Automation")
   - **Associated workspace**: Select your workspace
   - **Type**: Internal integration
4. **Capabilities**: Select the permissions you need:
   - Read content
   - Update content
   - Insert content
5. Save the integration
6. Copy the **Internal Integration Secret** (starts with `ntn_`)

### Step 2: Share Content with Integration

**Important**: Notion integrations can only access pages/databases that are explicitly shared with them.

To share a page or database:
1. Open the page/database in Notion
2. Click **Share** in the top right
3. Click **Invite**
4. Search for your integration name
5. Click **Invite**

Repeat for any page or database you want Studio to access.

### Step 3: Add Credential in Studio

1. Go to **Providers > Notion > Credentials**
2. Click **Add Credential**
3. Paste the Internal Integration Secret into **Internal Integration Secret**
4. Save

## Available Services

### Database Operations

| Service | Description |
|---------|-------------|
| **Query Database** | Search and filter database items with complex conditions |
| **Get Database** | Retrieve database schema and property definitions |

### Page Operations

| Service | Description |
|---------|-------------|
| **Create Page** | Create new pages in Notion (in databases or as subpages) |
| **Update Page** | Modify page properties, archive status, icon, or cover |
| **Get Page** | Retrieve page details and properties |
| **Append Block Children** | Add content blocks (paragraphs, headings, lists) to pages |

### Search

| Service | Description |
|---------|-------------|
| **Search** | Search all accessible pages and databases by title |

## Common Use Cases

### 1. Form Submission → Notion Database

Automatically create database entries from form submissions:

**Workflow:**
1. **Trigger**: Form submission (Typeform, Google Forms)
2. **Create Page** in Notion database
3. **Notification**: Send Slack message with link

**Example Properties:**
```json
{
  "properties": {
    "Name": {
      "title": [{"text": {"content": "John Doe"}}]
    },
    "Email": {
      "email": "john@example.com"
    },
    "Status": {
      "select": {"name": "New"}
    },
    "Created": {
      "date": {"start": "2024-01-15"}
    }
  }
}
```

### 2. Task Management Automation

Update task status based on external events:

**Workflow:**
1. **Query Database**: Find tasks with status "In Progress"
2. **Check External API**: Get completion status
3. **Update Page**: Change status to "Complete"

### 3. Content Publishing

Publish Notion pages to other platforms:

**Workflow:**
1. **Query Database**: Find pages with "Ready to Publish" status
2. **Get Page**: Retrieve full content
3. **Transform**: Convert Notion blocks to HTML/Markdown
4. **Publish**: Send to blog, CMS, or social media

### 4. Meeting Notes → Calendar Events

Create calendar events from meeting notes:

**Workflow:**
1. **Query Database**: Find pages with tag "Meeting"
2. **Extract**: Parse date/time from properties
3. **Create Event**: Add to Google Calendar
4. **Update Page**: Mark as "Scheduled"

### 5. Knowledge Base Sync

Keep external systems in sync with Notion:

**Workflow:**
1. **Search**: Find pages modified in last 24 hours
2. **Get Page**: Retrieve updated content
3. **Transform**: Convert to target format
4. **Sync**: Update external knowledge base

## Finding IDs

### Database ID

From the database URL:
```
https://www.notion.so/[WORKSPACE]/[DATABASE_ID]?v=[VIEW_ID]
```

The `DATABASE_ID` is the 32-character string (with or without hyphens).

**Alternative**:
1. Click **Share** on the database
2. Click **Copy link**
3. The ID is in the URL

### Page ID

From the page URL:
```
https://www.notion.so/[PAGE_TITLE]-[PAGE_ID]
```

The `PAGE_ID` is the last 32 characters (with or without hyphens).

## Property Types

Notion supports many property types. Here's how to structure them:

### Title
```json
{
  "Name": {
    "title": [
      {"text": {"content": "Page Title"}}
    ]
  }
}
```

### Rich Text
```json
{
  "Description": {
    "rich_text": [
      {"text": {"content": "Some text"}}
    ]
  }
}
```

### Select
```json
{
  "Status": {
    "select": {"name": "In Progress"}
  }
}
```

### Multi-select
```json
{
  "Tags": {
    "multi_select": [
      {"name": "Important"},
      {"name": "Urgent"}
    ]
  }
}
```

### Date
```json
{
  "Due Date": {
    "date": {"start": "2024-01-15"}
  }
}
```

### Date Range
```json
{
  "Event": {
    "date": {
      "start": "2024-01-15",
      "end": "2024-01-16"
    }
  }
}
```

### Checkbox
```json
{
  "Completed": {
    "checkbox": true
  }
}
```

### Number
```json
{
  "Priority": {
    "number": 5
  }
}
```

### URL
```json
{
  "Link": {
    "url": "https://example.com"
  }
}
```

### Email
```json
{
  "Contact": {
    "email": "user@example.com"
  }
}
```

### Phone
```json
{
  "Phone": {
    "phone_number": "+1-555-0123"
  }
}
```

## Content Blocks

When creating or appending content, use block objects:

### Paragraph
```json
{
  "object": "block",
  "type": "paragraph",
  "paragraph": {
    "rich_text": [
      {"text": {"content": "This is a paragraph."}}
    ]
  }
}
```

### Heading 1
```json
{
  "object": "block",
  "type": "heading_1",
  "heading_1": {
    "rich_text": [
      {"text": {"content": "Main Heading"}}
    ]
  }
}
```

### Bulleted List
```json
{
  "object": "block",
  "type": "bulleted_list_item",
  "bulleted_list_item": {
    "rich_text": [
      {"text": {"content": "List item"}}
    ]
  }
}
```

### To-Do List
```json
{
  "object": "block",
  "type": "to_do",
  "to_do": {
    "rich_text": [
      {"text": {"content": "Task to complete"}}
    ],
    "checked": false
  }
}
```

## Filtering Databases

Use the `filter` parameter in **Query Database** to search with conditions:

### Simple Filter
```json
{
  "property": "Status",
  "select": {
    "equals": "In Progress"
  }
}
```

### Compound Filter (AND)
```json
{
  "and": [
    {
      "property": "Status",
      "select": {"equals": "Active"}
    },
    {
      "property": "Priority",
      "number": {"greater_than": 3}
    }
  ]
}
```

### Compound Filter (OR)
```json
{
  "or": [
    {
      "property": "Status",
      "select": {"equals": "Complete"}
    },
    {
      "property": "Archived",
      "checkbox": {"equals": true}
    }
  ]
}
```

## Sorting

Sort database results:

```json
{
  "sorts": [
    {
      "property": "Priority",
      "direction": "descending"
    },
    {
      "property": "Created",
      "direction": "ascending"
    }
  ]
}
```

## Rate Limits

Notion API has rate limits:
- **3 requests per second** per integration
- Use pagination for large result sets
- Implement exponential backoff for rate limit errors

## Best Practices

1. **Share Selectively**: Only share pages/databases the integration needs
2. **Use Database Templates**: Create database templates for consistent structure
3. **Property Names**: Use exact property names (case-sensitive)
4. **Error Handling**: Check for missing properties or permissions errors
5. **Pagination**: Always handle `has_more` and `next_cursor` for complete results
6. **Testing**: Test with a dedicated test workspace before production use

## Troubleshooting

### "Object not found" error

**Cause**: Page/database not shared with integration

**Solution**: Share the page/database with your integration

### "Validation failed" error

**Cause**: Property name or type mismatch

**Solution**: Check database schema with **Get Database** service to see exact property names and types

### "Rate limit exceeded"

**Cause**: Too many requests in short time

**Solution**: Add delays between requests or implement retry logic with exponential backoff

## Additional Resources

- [Notion API Documentation](https://developers.notion.com/)
- [Notion API Reference](https://developers.notion.com/reference)
- [Block Types Reference](https://developers.notion.com/reference/block)
- [Property Value Reference](https://developers.notion.com/reference/property-value-object)

## Terms

Your use of Notion is governed by Notion's own terms, not by Studio's: [https://notion.notion.site/Terms-and-Privacy-28ffdd083dc3473e9c2da6ec011b58ac](https://notion.notion.site/Terms-and-Privacy-28ffdd083dc3473e9c2da6ec011b58ac). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
