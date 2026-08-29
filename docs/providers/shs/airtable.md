# Airtable Provider

Connect to Airtable to manage database records, query tables, and automate data workflows.

## How It Works

Airtable is a flexible database platform that combines spreadsheet simplicity with database power. Studio connects to Airtable via their REST API, enabling you to:

- **List Records** - Query and filter data from any table
- **Get Record** - Retrieve a single record by ID
- **Create Records** - Add new rows to tables
- **Update Records** - Modify existing data
- **Delete Records** - Remove records
- **Schema Discovery** - List bases and tables dynamically

Studio automatically discovers your Airtable schema, populating dropdowns with your actual bases, tables, views, and fields.

## Authentication

Airtable uses **Personal Access Token** (PAT) authentication.

### Setup

1. Go to [airtable.com/create/tokens](https://airtable.com/create/tokens)
2. Click **Create new token**
3. Name your token (e.g., "Studio Integration")
4. Add scopes:
   - `data.records:read` - Read records
   - `data.records:write` - Create/update/delete records
   - `schema.bases:read` - List bases and tables
5. Select which bases to grant access to (or all)
6. Click **Create token** and copy it
7. In Studio, go to **Providers > Airtable > Credentials**
8. Click **Add Credential** and paste your token

## Available Services

### List Records

Retrieve records from a table with filtering, sorting, and pagination.

| Parameter | Description |
|-----------|-------------|
| **base_id** | Select from your bases (auto-populated) |
| **table_id** | Select a table (auto-populated based on base) |
| **view** | Optional: use a specific view's filters |
| **fields** | Return only specific fields |
| **filterByFormula** | Airtable formula to filter records |
| **maxRecords** | Limit total records returned |
| **sort** | Sort by field(s) ascending/descending |

**Filter Formula Examples:**
```
{Status} = 'Active'
AND({Priority} = 'High', {Assigned} != '')
SEARCH('keyword', {Description})
{Created} >= '2024-01-01'
```

### Get Record

Retrieve a single record by its ID.

| Parameter | Description |
|-----------|-------------|
| **base_id** | Base containing the record |
| **table_id** | Table containing the record |
| **record_id** | The record ID (starts with `rec`) |

### Create Records

Add one or more new records (up to 10 per request).

| Parameter | Description |
|-----------|-------------|
| **base_id** | Target base |
| **table_id** | Target table |
| **records** | Array of records with field values |
| **typecast** | Auto-convert strings to proper types (select, date, etc.) |

### Update Records

Modify existing records (PATCH - only specified fields change).

| Parameter | Description |
|-----------|-------------|
| **base_id** | Base containing records |
| **table_id** | Table containing records |
| **records** | Array with record IDs and fields to update |
| **typecast** | Auto-convert strings to proper types |

### Delete Records

Remove records by their IDs (up to 10 per request).

| Parameter | Description |
|-----------|-------------|
| **base_id** | Base containing records |
| **table_id** | Table containing records |
| **records** | Array of record IDs to delete |

### List Bases

Discover all bases you have access to.

### Get Base Schema

Get table and field definitions for a base.

---

## Example Workflows

### Form Submission to Database

**Use case:** Store web form submissions in Airtable

```
Trigger: Webhook (form submission)
  ↓
Step 1: Airtable Create Records
  - Base: Lead Database
  - Table: Submissions
  - Records:
    - Name: {{ trigger.name }}
    - Email: {{ trigger.email }}
    - Message: {{ trigger.message }}
    - Status: "New"
```

### Daily Report Generation

**Use case:** Query data and send summary email

```
Trigger: Schedule (daily at 9am)
  ↓
Step 1: Airtable List Records
  - Table: Orders
  - Filter: {Created} >= TODAY() - 1
  ↓
Step 2: Send Email
  - Subject: "Daily Order Report"
  - Body: "{{ step1.records.length }} orders yesterday"
```

### Status Update Automation

**Use case:** Update record status based on conditions

```
Trigger: Incoming Webhook (posted by an Airtable automation)
  ↓
Step 1: Check conditions
  - If all subtasks complete
  ↓
Step 2: Airtable Update Records
  - Record: {{ trigger.record_id }}
  - Status: "Complete"
  - Completed Date: {{ now() }}
```

---

## Multi-Step Workflows

> **Plus catalog:** Plus workflows and video walkthroughs come with a [SelfHost Innovators membership](https://www.skool.com/selfhostinnovators).

### CRM Data Sync Pipeline

Sync customer data between Airtable and external systems:

1. **Query Changed Records** - Filter by last modified date
2. **Data Transformation** - Map fields between schemas
3. **Upsert to External CRM** - Create or update records
4. **Update Sync Status** - Mark records as synced
5. **Error Handling** - Log failures for review

### Inventory Management System

Automate stock tracking and reordering:

1. **Monitor Stock Levels** - Query products below threshold
2. **Generate Purchase Orders** - Create PO records
3. **Notify Suppliers** - Send automated emails
4. **Update Status** - Mark items as "On Order"
5. **Receipt Processing** - Update stock on delivery

### Multi-Table Relationship Automation

Maintain data consistency across linked tables:

1. **Detect Changes** - Monitor parent record updates
2. **Cascade Updates** - Apply changes to linked records
3. **Recalculate Rollups** - Update summary fields
4. **Audit Logging** - Track all changes

---

## Airtable Formula Reference

Common formulas for filtering:

| Use Case | Formula |
|----------|---------|
| Active records | `{Status} = 'Active'` |
| Today's records | `IS_SAME({Date}, TODAY(), 'day')` |
| This week | `{Date} >= DATEADD(TODAY(), -7, 'days')` |
| Not empty | `{Field} != ''` |
| Contains text | `SEARCH('keyword', {Field})` |
| Multiple conditions | `AND({A} = 1, {B} = 2)` |
| Either condition | `OR({A} = 1, {B} = 2)` |

## Troubleshooting

| Error | Solution |
|-------|----------|
| "INVALID_PERMISSIONS" | Check your token has the required scopes |
| "NOT_FOUND" | Verify base/table/record IDs are correct |
| "INVALID_FILTER_BY_FORMULA" | Check formula syntax at Airtable docs |
| "INVALID_REQUEST_UNKNOWN" | Verify field names match exactly (case-sensitive) |
| "Rate limited" | Airtable allows 5 requests/second per base |

## Tips

1. **Use Views** - Create Airtable views with pre-applied filters, then query the view
2. **Typecast** - Enable typecast when creating records with text values for select fields
3. **Batch Operations** - Create/update up to 10 records per request
4. **Field Names** - Field names are case-sensitive and must match exactly
5. **Pagination** - Use `offset` from response for large datasets

## Terms

Your use of Airtable is governed by Airtable's own terms, not by Studio's: [https://www.airtable.com/company/tos](https://www.airtable.com/company/tos). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
