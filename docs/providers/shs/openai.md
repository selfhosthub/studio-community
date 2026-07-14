# OpenAI Provider

Connect to OpenAI's GPT models for chat completion, embeddings, and DALL-E image generation.

## How It Works

OpenAI's API provides access to powerful language models (GPT-4, GPT-4o) and image generation (DALL-E 3). Studio integrates these capabilities into your workflows:

- **Chat Completion** sends messages to GPT models and receives intelligent responses
- **Embeddings** converts text into vector representations for semantic search
- **Image Generation** creates images from text descriptions using DALL-E

All requests are authenticated with your OpenAI API key and billed to your OpenAI account.

## Authentication

OpenAI uses **API Key** authentication with Bearer token.

### Setup

1. Go to [platform.openai.com](https://platform.openai.com/)
2. Navigate to **API Keys** in the sidebar
3. Click **Create new secret key**
4. Copy the key (it won't be shown again)
5. In Studio, go to **Providers > OpenAI > Credentials**
6. Click **Add Credential** and paste your API key

**Optional:** Add an Organization ID if you want billing attributed to a specific org.

## Available Services

### Chat Completion

Generate conversational responses using GPT models.

| Parameter | Description |
|-----------|-------------|
| **model** | GPT-4o (recommended), GPT-4o Mini, GPT-4 Turbo, GPT-4, GPT-3.5 Turbo |
| **messages** | Conversation history with roles (system, user, assistant) |
| **temperature** | Creativity (0 = focused, 2 = creative) |
| **max_tokens** | Maximum response length |

**Advanced Parameters:**
- `top_p` - Nucleus sampling
- `frequency_penalty` - Reduce repetition
- `presence_penalty` - Encourage new topics
- `response_format` - Force JSON output
- `seed` - Reproducible outputs

### Embeddings

Generate vector embeddings for semantic search and similarity.

| Parameter | Description |
|-----------|-------------|
| **model** | text-embedding-3-small (recommended), text-embedding-3-large, text-embedding-ada-002 |
| **input** | Text to convert to vectors |
| **dimensions** | Output vector size (smaller = faster, larger = more accurate) |

### Image Generation (DALL-E)

Create images from text descriptions.

| Parameter | Description |
|-----------|-------------|
| **prompt** | Description of the image to create |
| **model** | DALL-E 3 (recommended) or DALL-E 2 |
| **size** | 1024x1024, 1792x1024 (landscape), 1024x1792 (portrait) |
| **quality** | standard or hd (DALL-E 3 only) |
| **style** | vivid or natural (DALL-E 3 only) |

## Available Models

### Chat Models
| Model | Best For | Context |
|-------|----------|---------|
| **gpt-4o** | General tasks, fast and capable | 128K |
| **gpt-4o-mini** | Cost-effective, high volume | 128K |
| **gpt-4-turbo** | Complex reasoning | 128K |
| **gpt-4** | Highest quality | 8K |
| **gpt-3.5-turbo** | Simple tasks, lowest cost | 16K |

### Embedding Models
| Model | Dimensions | Use Case |
|-------|------------|----------|
| **text-embedding-3-small** | 1536 | General purpose, cost-effective |
| **text-embedding-3-large** | 3072 | Highest accuracy |
| **text-embedding-ada-002** | 1536 | Legacy compatibility |

---

## Basic Workflows

### Customer Support Auto-Response

**Use case:** Automatically respond to customer inquiries

```
Trigger: New support ticket
  ↓
Step 1: OpenAI Chat
  - System: "You are a helpful customer service agent for [Company]"
  - User: {{ ticket.message }}
  ↓
Step 2: Send Email
  - To: {{ ticket.email }}
  - Body: {{ step1.choices.0.message.content }}
```

### Content Summarization

**Use case:** Summarize long documents

```
Trigger: New document uploaded
  ↓
Step 1: OpenAI Chat
  - System: "Summarize the following document in 3 bullet points"
  - User: {{ document.content }}
  ↓
Step 2: Save to Airtable
  - Summary: {{ step1.choices.0.message.content }}
```

### Product Image Generation

**Use case:** Generate product mockups

```
Trigger: New product in database
  ↓
Step 1: OpenAI Images
  - Prompt: "Professional product photo of {{ product.name }}, white background, studio lighting"
  - Size: 1024x1024
  - Quality: hd
  ↓
Step 2: Upload to storage
```

---

## Advanced Workflows

> **Skool Members:** Access advanced workflow templates and video tutorials at [skool.com/selfhost](https://skool.com/selfhost)

### Multi-Step Content Pipeline

Generate blog posts with AI-created images:

1. **Outline Generation** - GPT creates article outline
2. **Section Expansion** - Each section expanded in parallel
3. **Image Generation** - DALL-E creates header and inline images
4. **SEO Optimization** - Final pass for keywords
5. **Publishing** - Auto-post to CMS

### Semantic Search System

Build a knowledge base with AI search:

1. **Document Ingestion** - Split documents into chunks
2. **Embedding Generation** - Convert chunks to vectors
3. **Vector Storage** - Store in database with metadata
4. **Query Processing** - Convert search query to embedding
5. **Similarity Search** - Find relevant chunks
6. **Answer Synthesis** - GPT generates answer from context

### Intelligent Data Extraction

Extract structured data from unstructured text:

1. **Input Normalization** - Clean raw text
2. **Entity Extraction** - GPT identifies entities with JSON mode
3. **Validation** - Verify extracted data
4. **Database Update** - Store structured records

---

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Invalid API key" | Check your API key at platform.openai.com |
| "Rate limit exceeded" | Reduce request frequency or upgrade plan |
| "Context length exceeded" | Reduce message history or use model with larger context |
| "Content policy violation" | Modify prompt to comply with usage policies |
| "Insufficient quota" | Add billing details at platform.openai.com |

## Tips for Better Results

1. **System prompts** - Always include a system message to set behavior
2. **Temperature** - Use 0 for factual tasks, 0.7-1 for creative tasks
3. **Token limits** - Set max_tokens to control costs and response length
4. **JSON mode** - Use `response_format: json_object` for structured output
5. **Embeddings** - Use the same model for queries and documents
