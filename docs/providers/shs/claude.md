# Claude Provider

Connect to Anthropic's Claude AI models for chat, analysis, tool use, and agentic coding tasks.

## Authentication

Claude uses **API Key** authentication. Get your API key from the Anthropic Console.

### Setup

1. Go to [console.anthropic.com](https://console.anthropic.com/)
2. Navigate to **API Keys**
3. Create a new API key
4. In Studio, go to **Providers > Claude > Credentials**
5. Click **Add Credential** and paste your API key

## Available Services

### Chat (Messages API)

| Parameter | Description |
|-----------|-------------|
| **model** | Claude model to use (Opus 4.8, Sonnet 4.6, Haiku 4.5, Opus 4.6) |
| **system** | System prompt to set context and behavior |
| **messages** | Conversation history (user/assistant turns) |
| **max_tokens** | Maximum tokens in response (required) |
| **temperature** | Randomness (0-1, default 1) |

**Advanced Parameters:**
- `top_p` - Nucleus sampling threshold
- `top_k` - Top-K sampling
- `stop_sequences` - Custom stop strings
- `tools` - Function/tool definitions
- `tool_choice` - How model chooses tools (auto/any/specific)

### Vision Analysis

Analyze images by sending them as base64 or URLs. All Claude 3+ models support vision.

**Content Block Format:**
```json
{
  "role": "user",
  "content": [
    {
      "type": "image",
      "source": {
        "type": "url",
        "url": "https://example.com/image.jpg"
      }
    },
    {
      "type": "text",
      "text": "What is in this image?"
    }
  ]
}
```

---

## Available Models

| Model | Description | Best For |
|-------|-------------|----------|
| **claude-opus-4-8** | Most capable, best reasoning | Complex analysis, coding, research |
| **claude-sonnet-4-6** | Balanced performance (default) | General tasks, good cost/performance |
| **claude-haiku-4-5-20251001** | Fast and efficient | High-volume, simple tasks |
| **claude-opus-4-6** | Previous generation flagship | Stable, well-tested |

---

## Example Workflows

### Inbound Support Triage

**Use case:** Classify a support request and draft a reply before a human sees it

```
Trigger: Incoming Webhook (helpdesk or form)
  |
Step 1: Chat (Messages API)
  - System: "Classify this ticket as billing, bug, or how-to.
    Return JSON with category, urgency and a two-sentence summary."
  - Messages: {{ trigger.body }}
  |
Step 2: Chat (Messages API)
  - System: "Draft a reply in the voice of our support team."
  - Messages: {{ step1.result }}
  |
Step 3: Wait for Approval
  - An agent approves or rejects the draft
  |
Step 4: Send Notification
  - Body: {{ step2.result }}
```

### Screenshot to Bug Report

**Use case:** Turn a screenshot a user attached into a structured report

```
Trigger: Incoming Webhook
  |
Step 1: Vision Analysis
  - Messages: the screenshot, with "Describe what is on screen,
    quote any visible error text, and name the screen."
  |
Step 2: Chat (Messages API)
  - System: "Write a bug report: title, steps to reproduce,
    expected, actual. Use only what the description states."
  - Messages: {{ step1.result }}
  |
Step 3: Send Notification
```

### Weekly Content Digest

**Use case:** Summarize the week and send it out on a schedule

```
Trigger: Schedule, weekly
  |
Step 1: HTTP Request
  - Pull the week's items from your own endpoint
  |
Step 2: Run Once Per Item
  - Chat (Messages API) summarizes each item in two sentences
  |
Step 3: Chat (Messages API)
  - System: "Assemble these summaries into one digest, ranked
    by importance."
  |
Step 4: Send Notification
```

---

## Multi-Step Workflows

> **Plus catalog:** Plus workflows and video walkthroughs come with a [SelfHost Innovators membership](https://www.skool.com/selfhostinnovators).

### Image Library Captioning

Caption a batch of images and keep the results:

1. **List Images** - Read the batch from your workspace
2. **Run Once Per Item** - Iterate the images
3. **Vision Analysis** - Describe each image and pull out any visible text
4. **Chat (Messages API)** - Rewrite each description as alt text under 125 characters
5. **Send Notification** - Report the batch when it finishes

### Document Question Answering

Answer a question against a document set:

1. **Set Fields** - Collect the question and the document text
2. **Run Once Per Item** - Iterate the documents
3. **Chat (Messages API)** - Answer from that document only, or say it is not covered
4. **Chat (Messages API)** - Merge the per-document answers into one, citing which document each claim came from
5. **Wait for Approval** - A human signs off before the answer is sent

---

## Tool Use (Function Calling)

Claude can use tools you define. Useful for structured output or calling external functions.

**Tool Definition:**
```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "input_schema": {
    "type": "object",
    "required": ["location"],
    "properties": {
      "location": {
        "type": "string",
        "description": "City and state, e.g. San Francisco, CA"
      }
    }
  }
}
```

**Tool Choice Options:**
- `auto` - Model decides when to use tools (default)
- `any` - Model must use one of the provided tools
- `tool` - Model must use a specific named tool

## Response Format

Claude responses include:
- `content` - Array of content blocks (text or tool_use)
- `stop_reason` - Why generation stopped (end_turn, max_tokens, tool_use)
- `usage` - Token counts (input_tokens, output_tokens)

**Example Response:**
```json
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "The capital of France is Paris."
    }
  ],
  "model": "claude-sonnet-4-6",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 25,
    "output_tokens": 12
  }
}
```

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Invalid API key" | Check your API key starts with `sk-ant-` and is complete |
| "max_tokens is required" | Claude API requires max_tokens - set a value (e.g., 4096) |
| "Overloaded" | API is at capacity - retry with exponential backoff |
| "Rate limited" | Reduce request frequency or upgrade your API plan |

## Usage Tips

1. **System Prompts**: Use system prompts for consistent behavior across turns
2. **Temperature**: Use 0 for deterministic outputs, 0.7-1 for creative tasks
3. **Max Tokens**: Set based on expected response length to control costs
4. **Tool Use**: Define clear descriptions for better tool selection
5. **Vision**: Send an image URL rather than base64 when the file already has a signed URL

---

## Terms

Your use of Anthropic is governed by Anthropic's own terms, not by Studio's: [https://www.anthropic.com/legal/consumer-terms](https://www.anthropic.com/legal/consumer-terms). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
