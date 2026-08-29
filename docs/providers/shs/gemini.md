# Gemini Provider

Connect to Google's Gemini AI models for chat, multimodal understanding, and function calling.

## Authentication

Gemini uses **API Key** authentication. Get your API key from Google AI Studio.

### Setup

1. Go to [aistudio.google.com](https://aistudio.google.com/)
2. Click **Get API Key**
3. Create a new API key or use an existing one
4. In Studio, go to **Providers > Gemini > Credentials**
5. Click **Add Credential** and paste your API key

## Available Services

### Generate Content

| Parameter | Description |
|-----------|-------------|
| **model** | Gemini model to use (3.5 Flash, 3.1 Pro, 3 Flash, 3.1 Flash-Lite) |
| **systemInstruction** | System-level instructions for the model |
| **contents** | Conversation history with role and parts |
| **generationConfig** | Temperature, maxOutputTokens, topP, topK |

**Content Format:**
```json
{
  "role": "user",
  "parts": [
    { "text": "What is the capital of France?" }
  ]
}
```

### Multimodal (Vision)

Analyze images by sending them as base64 data or Cloud Storage URIs.

**Image Input Format:**
```json
{
  "role": "user",
  "parts": [
    {
      "inlineData": {
        "mimeType": "image/jpeg",
        "data": "BASE64_ENCODED_IMAGE"
      }
    },
    { "text": "What is in this image?" }
  ]
}
```

**Supported formats:** JPEG, PNG, GIF, WebP, PDF

### Embeddings

Generate vector embeddings for semantic search and similarity.

| Parameter | Description |
|-----------|-------------|
| **model** | gemini-embedding-2 (recommended) or text-embedding-004 |
| **content** | Text to embed |
| **taskType** | RETRIEVAL_QUERY, RETRIEVAL_DOCUMENT, SEMANTIC_SIMILARITY, etc. |
| **outputDimensionality** | Optional reduced dimensions (1-768) |

## Available Models

| Model | Description | Best For |
|-------|-------------|----------|
| **gemini-3.5-flash** | Recommended, fast & efficient | General tasks, good cost/performance |
| **gemini-3.1-pro** | Most capable | Complex reasoning, long context |
| **gemini-3-flash** | Balanced | Everyday generation |
| **gemini-3.1-flash-lite** | Cheapest | High-volume, simple tasks |

## Function Calling

Gemini supports function calling through the tools parameter.

**Function Declaration:**
```json
{
  "functionDeclarations": [
    {
      "name": "get_weather",
      "description": "Get current weather for a location",
      "parameters": {
        "type": "object",
        "required": ["location"],
        "properties": {
          "location": {
            "type": "string",
            "description": "City name"
          }
        }
      }
    }
  ]
}
```

**Tool Config Modes:**
- `AUTO` - Model decides when to call functions (default)
- `ANY` - Model must call one of the functions
- `NONE` - Disable function calling

## Response Format

Gemini responses include:
- `candidates` - Array of response candidates
- `candidates[].content.parts` - Response content (text or functionCall)
- `candidates[].finishReason` - Why generation stopped
- `usageMetadata` - Token counts

**Example Response:**
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          { "text": "The capital of France is Paris." }
        ],
        "role": "model"
      },
      "finishReason": "STOP"
    }
  ],
  "usageMetadata": {
    "promptTokenCount": 10,
    "candidatesTokenCount": 8,
    "totalTokenCount": 18
  }
}
```

## Safety Settings

Configure content safety filters per category:

| Category | Description |
|----------|-------------|
| HARM_CATEGORY_HARASSMENT | Harassment content |
| HARM_CATEGORY_HATE_SPEECH | Hate speech |
| HARM_CATEGORY_SEXUALLY_EXPLICIT | Sexual content |
| HARM_CATEGORY_DANGEROUS_CONTENT | Dangerous content |

**Threshold Options:**
- `BLOCK_NONE` - No blocking
- `BLOCK_ONLY_HIGH` - Block high probability
- `BLOCK_MEDIUM_AND_ABOVE` - Block medium+ (default)
- `BLOCK_LOW_AND_ABOVE` - Block all flagged

## Troubleshooting

| Error | Solution |
|-------|----------|
| "API key not valid" | Check your API key is correct and active |
| "Model not found" | Use exact model IDs from the enum list |
| "Content blocked" | Adjust safety settings or modify content |
| "Quota exceeded" | Check your API quota at Google AI Studio |

## Key Differences from OpenAI/Claude

1. **Message format**: Uses `contents` with `parts` array (not `messages` with `content` string)
2. **Roles**: Uses `user` and `model` (not `user` and `assistant`)
3. **System prompt**: Uses `systemInstruction` object (not a message role)
4. **API key**: Passed as query parameter (not header)
5. **Model in URL**: Model specified in endpoint path, not body

---

## Terms

Your use of Google is governed by Google's own terms, not by Studio's: [https://ai.google.dev/gemini-api/terms](https://ai.google.dev/gemini-api/terms). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
