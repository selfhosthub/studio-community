# Perplexity Provider

Connect to Perplexity AI for search-augmented chat completions with real-time web citations.

## Authentication

Perplexity uses **API Key** authentication (Bearer token).

### Setup

1. Go to [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api)
2. Generate an API key (starts with `pplx-`)
3. In Studio, go to **Providers > Perplexity > Credentials**
4. Click **Add Credential** and paste your API key

## Available Services

### Chat Completion (Sonar)

Search-augmented chat using Perplexity's Sonar models. Every response includes real-time web citations.

| Parameter | Description |
|-----------|-------------|
| **model** | Sonar model to use |
| **messages** | Conversation messages (system/user/assistant) |
| **temperature** | Randomness (0-2, default 0.2) |
| **max_tokens** | Maximum tokens in response |

**Search Parameters:**
- `search_recency_filter` - Limit web sources by time: `hour`, `day`, `week`, `month`
- `return_related_questions` - Include suggested follow-up questions
- `search_domain_filter` - Restrict search to specific domains (e.g., `wikipedia.org`). Prefix with `-` to exclude

**Advanced Parameters:**
- `top_p` - Nucleus sampling (0-1, default 0.9)
- `frequency_penalty` - Reduce token repetition (default 1)
- `presence_penalty` - Penalize tokens already in text (-2 to 2)

---

## Available Models

| Model | Description | Best For |
|-------|-------------|----------|
| **sonar** | Fast, lightweight search | Quick facts, current events, simple lookups |
| **sonar-pro** | Advanced search, more sources | Complex queries needing citations and follow-ups |
| **sonar-reasoning-pro** | Reasoning + live web search | Multi-step analysis with web grounding |
| **sonar-deep-research** | Exhaustive research reports | Comprehensive reports, async, longer runtime |

**Model selection tips:**
- Use `sonar` for fast answers where speed matters more than depth
- Use `sonar-pro` for research tasks that need comprehensive source coverage
- Use `sonar-reasoning-pro` when the query requires analysis and live web data in one step
- Use `sonar-deep-research` for exhaustive reports (may need higher timeouts)

---

## Response Format

Perplexity responses follow the OpenAI chat completion format with an added `citations` array:

```json
{
  "id": "chatcmpl-...",
  "model": "sonar",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The answer with inline citations [1][2]..."
      },
      "finish_reason": "stop"
    }
  ],
  "citations": [
    "https://example.com/source-1",
    "https://example.com/source-2"
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 150,
    "total_tokens": 175
  }
}
```

The `citations` array contains URLs referenced by bracketed numbers in the response text (e.g., `[1]` maps to `citations[0]`).

---

## Workflow Examples

### AI Research Assistant

**Use case:** Answer questions with cited web sources

```
Trigger: Manual
  ↓
Step 1: Perplexity Chat (Sonar Pro)
  - Model: sonar-pro
  - Messages: [system: "Research assistant", user: "{{ question }}"]
  - Search Recency: week
  ↓
Step 2: Format results with citations
```

### Competitive Intelligence

**Use case:** Research a company with real-time data

```
Trigger: Manual
  ↓
Step 1: Perplexity Chat (Sonar Pro) - Company overview
Step 2: Perplexity Chat (Sonar Pro) - Competitor analysis
Step 3: Perplexity Chat (Sonar) - Market trends
  ↓ (parallel research, fan-in)
Step 4: Claude - Synthesize into intelligence brief
```

### Fact-Checking Pipeline

**Use case:** Verify claims with web sources

```
Trigger: Manual
  ↓
Step 1: Perplexity Chat (Sonar Pro)
  - System: "Verify this claim. Cite sources for and against."
  - Search Recency: month
  ↓
Step 2: Claude - Summarize verdict with confidence score
```

---

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Invalid API key" | Check your key starts with `pplx-` and is complete |
| "Rate limited" | Reduce request frequency or check your Perplexity plan limits |
| Empty citations | Not all queries produce citations - try more specific questions |
| Stale results | Use `search_recency_filter` to limit to recent sources |

## Usage Tips

1. **System Prompts**: Use system messages to set the research context and output format
2. **Temperature**: Keep low (0.2) for factual research, higher for creative exploration
3. **Recency Filter**: Use `day` or `hour` for time-sensitive topics like news or stock data
4. **Domain Filter**: Restrict to trusted domains for sensitive research (e.g., `.gov`, `.edu`)
5. **Model Choice**: Start with `sonar` for cost efficiency; upgrade to `sonar-pro` for depth
6. **Citations**: Always surface the `citations` array to users - it's Perplexity's key differentiator

---

## Terms

Your use of Perplexity is governed by Perplexity's own terms, not by Studio's: [https://www.perplexity.ai/hub/legal/terms-of-service](https://www.perplexity.ai/hub/legal/terms-of-service). Costs, rate limits, content-ownership rules, and acceptable-use policies are set by the provider. You are responsible for complying with them and for any charges you incur. See LEGAL.md in the Studio repository.
