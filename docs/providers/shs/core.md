# Core Provider

Built-in workflow utilities for control flow, data transformation, HTTP requests, and human-in-the-loop steps.

## How It Works

The Core provider ships with every Studio instance. It provides foundational building blocks that don't require external API credentials:

- **Data Transformation** - Set fields, map values, restructure data
- **Control Flow** - Stop workflows, branch logic
- **HTTP** - Call webhooks, make HTTP requests, wait for callbacks
- **Human-in-the-Loop** - Approval gates, notifications, logging

## Authentication

**No credentials required** - Core is a built-in provider available to all workflows.

## Available Services

### Set Fields

Create or modify fields with static values or expressions.

| Parameter | Description |
|-----------|-------------|
| **mode** | `manual` (field list) or `json` (raw JSON) |
| **fields** | Array of field definitions: name, type, value, is_expression |

**Field Types:** `string`, `number`, `boolean`, `array`, `object`

Use expressions (`is_expression: true`) to reference previous step outputs: `{{ step1.result }}`

### Call Webhook

Send an HTTP request to a webhook URL for outbound integrations.

| Parameter | Description |
|-----------|-------------|
| **url** | Webhook URL to call |
| **method** | HTTP method (POST, GET, PUT, etc.) |
| **headers** | Custom headers (optional) |
| **body** | Request body (optional) |

### HTTP Request

Send an HTTP request to any URL with full control over method, headers, and body.

| Parameter | Description |
|-----------|-------------|
| **url** | Target URL |
| **method** | GET, POST, PUT, PATCH, DELETE |
| **headers** | Custom request headers |
| **body** | Request body |
| **timeout** | Request timeout in seconds |

### Wait for Webhook

Pause workflow execution until an external webhook callback is received.

| Parameter | Description |
|-----------|-------------|
| **path** | Webhook path to listen on |
| **timeout** | Max wait time in seconds |
| **expected_status** | HTTP status code to return |

### Wait for Approval

Pause workflow execution until a human approves or rejects.

| Parameter | Description |
|-----------|-------------|
| **message** | Message shown to the approver |
| **timeout** | Max wait time (optional) |

### Stop Workflow

Immediately stop workflow execution.

| Parameter | Description |
|-----------|-------------|
| **reason** | Why the workflow is being stopped |

### Send Notification

Send an in-app notification to users.

| Parameter | Description |
|-----------|-------------|
| **message** | Notification message |
| **level** | `info`, `warning`, `error` |

### Log Message

Log a message to workflow execution history for debugging and audit trails.

| Parameter | Description |
|-----------|-------------|
| **message** | Log message content |
| **level** | `info`, `warning`, `error` |

### Poll Service

Repeatedly call an API endpoint until it returns a success status and an optional condition is met.

| Parameter | Description |
|-----------|-------------|
| **url** | Endpoint to poll |
| **method** | HTTP method |
| **interval** | Seconds between polls |
| **max_attempts** | Maximum poll attempts |
| **success_condition** | Expression to evaluate response |
| **credential_id** | Credential for authenticated polling (optional) |

---

## Basic Workflows

### Data Pipeline

**Use case:** Transform data between steps

```
Trigger: Webhook
  |
Step 1: Set Fields
  - Extract and reshape incoming data
  |
Step 2: HTTP Request
  - Forward transformed data to external API
```

### Approval Gate

**Use case:** Require human sign-off before proceeding

```
Step 1: Generate report
  |
Step 2: Wait for Approval
  - Message: "Review the generated report before publishing"
  |
Step 3: Publish report
```

### Async Job Polling

**Use case:** Wait for an external job to complete

```
Step 1: Start external job (HTTP Request)
  |
Step 2: Poll Service
  - URL: {{ step1.status_url }}
  - Interval: 10s
  - Success Condition: status == "complete"
  |
Step 3: Process results
```

---

## Tips

1. **Set Fields** is the most versatile step - use it to restructure data between any two steps
2. **Expressions** reference previous outputs with `{{ step_name.field }}` syntax
3. **Poll Service** is ideal for async APIs that return a job ID and status URL
4. **Approval** steps pause indefinitely until a human acts - set timeouts for time-sensitive flows
5. **Log** steps are free and useful for debugging - add them liberally during development
