# Studio Workflows

The Studio workflow catalog. Some entries are installable today; the rest are in development and listed here as the roadmap. Each entry says which.

**Last Updated:** 2026-07-14

---

## What Are Workflows?

A workflow is a sequence of connected steps that automate a task end to end. Each step calls a provider service - an AI model, a SaaS API, or a local tool - and passes its output to the next step. You configure a workflow once, run it whenever you need it, and pay nothing beyond the cost of your own API keys and compute.

**Available to install today: 3.** One Community workflow (Faceless Video Pipeline) and two Plus workflows (Faceless Video API, Faceless Video API Webhook). The other 21 entries below are in development and cannot be installed yet.

Plus workflows require an entitlement token, which comes with a [SelfHost Innovators membership](https://www.skool.com/selfhostinnovators).

### Why Self-Hosted?

- **Your data stays on your servers.** Nothing leaves your infrastructure unless you tell it to.
- **No per-run platform fees.** Local models (ComfyUI, Chatterbox TTS) run on your GPU at the cost of electricity.
- **Run any model.** OpenAI, Claude, Gemini, and open-source models side by side - swap providers without rewriting workflows.
- **No vendor lock-in.** Export, modify, and extend any workflow on your own deployment. The workflow configurations you create are yours; Studio and the marketplace packages are licensed, not sold.

---

## Community Workflows

Included free with every Studio installation. Each one solves a specific, common task in 1-4 steps.

### AI Blog Post Generator

Generate a structured blog post from a topic, starting with an outline and then writing the full post.

| | |
|---|---|
| **Providers** | OpenAI |
| **Steps** | 2 |
| **Category** | AI / Writing |

---

### AI Image Generator

Generate high-quality AI images from text prompts using ComfyUI with customizable styles and batch sizes. Runs entirely on your local GPU - no API costs per image.

| | |
|---|---|
| **Providers** | ComfyUI |
| **Steps** | 1 |
| **Category** | AI / Image |

---

### Summarize & Send Email Report

Fetch data from Google Sheets, summarize it into an executive report using AI, and send via Gmail. Great for automated weekly reports, status updates, or digest emails.

| | |
|---|---|
| **Providers** | OpenAI, Google |
| **Steps** | 3 |
| **Category** | Productivity |

---

### Social Post Writer

Generate platform-specific social media posts for LinkedIn, Twitter/X, and Instagram from a single topic. One input, three outputs, each optimized for the platform.

| | |
|---|---|
| **Providers** | OpenAI |
| **Steps** | 3 |
| **Category** | Marketing |

---

### Notion Page Creator

Generate structured content with AI and create a new page in a Notion database. Meeting notes, project briefs, research summaries - pick a type and go.

| | |
|---|---|
| **Providers** | OpenAI, Notion |
| **Steps** | 3 |
| **Category** | Productivity |

---

### Telegram Notification Bot

Format raw data into a polished notification message with AI and send it to a Telegram chat. Supports text, photos, and document attachments.

| | |
|---|---|
| **Providers** | OpenAI, Telegram |
| **Steps** | 2 |
| **Category** | Communication |

---

### Airtable Record Creator

Extract structured data from unstructured text using AI and create records in Airtable. Paste an email, note, or description - AI pulls out the key fields and writes them to your base.

| | |
|---|---|
| **Providers** | OpenAI, Airtable |
| **Steps** | 2 |
| **Category** | Productivity |

---

### Text to Speech

Convert text to natural-sounding speech audio using the SHS Audio TTS service. Multiple voices, adjustable speed, powered by Chatterbox running locally.

| | |
|---|---|
| **Providers** | SHS Audio |
| **Steps** | 1 |
| **Category** | AI / Audio |

---

### Customer Support Reply Drafter

Analyze customer messages for sentiment and issues, draft professional replies, and queue for human approval. Claude identifies the core issue and writes a response that matches your brand tone.

| | |
|---|---|
| **Providers** | Claude, Core |
| **Steps** | 2 |
| **Category** | AI / Support |

---

### Google Drive Organizer

List files from a Google Drive folder and use AI to categorize them into suggested folder structures. Bring order to a messy Drive folder in one run.

| | |
|---|---|
| **Providers** | OpenAI, Google |
| **Steps** | 4 |
| **Category** | Productivity |

---

### Faceless Video Pipeline

Turn a news article into a narrated video with AI-generated images, TTS audio, per-image zoom effects, and subtitles. The flagship workflow that demonstrates the full power of local AI infrastructure.

| | |
|---|---|
| **Providers** | OpenAI, ComfyUI, SHS Audio, SHS Video, Core |
| **Steps** | 7+ |
| **Category** | Video |

---

## Plus Workflows

Require an entitlement token, which comes with a [SelfHost Innovators membership](https://www.skool.com/selfhostinnovators). Multi-step automations aimed at the kind of work agencies bill for.

### AI Email Triage Agent

24/7 AI email agent that reads every incoming message, classifies it by type and urgency, drafts a reply for anything actionable, logs to your CRM, and pings your team on Telegram with a summary.

| | |
|---|---|
| **Replaces** | Commercial email-triage tooling |
| **Providers** | Claude, Google, Airtable, Notion, Telegram |
| **Steps** | 6 |
| **Trigger** | Schedule (every 15 min) |
| **Category** | AI / Productivity |

---

### Lead Capture & Outreach Pipeline

The full AI SDR loop. A lead fills out your form, AI instantly researches their company, scores the lead against your ICP, writes a personalized first-touch email, logs to your CRM, waits for approval, and sends.

| | |
|---|---|
| **Replaces** | Commercial AI SDR tooling |
| **Providers** | Claude, Perplexity, Airtable |
| **Steps** | 8 |
| **Trigger** | Webhook |
| **Category** | Sales |

---

### Client Onboarding Automation

One form submission, complete client setup in under 60 seconds. AI generates a personalized welcome message and project brief, creates the client's Notion workspace, logs to your CRM, sends a warm welcome email, and pings your team.

| | |
|---|---|
| **Replaces** | Manual client onboarding (30-60 min per client) |
| **Providers** | OpenAI, Notion, Airtable, Google, Telegram |
| **Steps** | 6 |
| **Trigger** | Webhook |
| **Category** | Operations |

---

### AI Document Intelligence

Drop any document into a watched Drive folder - invoice, contract, resume, report - and get structured data extracted automatically. AI reads it with vision, pulls every field, flags anomalies, and writes clean records to your database.

| | |
|---|---|
| **Replaces** | Manual data entry and commercial OCR tooling |
| **Providers** | Claude, Airtable, Google, Telegram |
| **Steps** | 6 |
| **Trigger** | Webhook |
| **Category** | AI / Data |

---

### Multi-LLM Content Evaluator

Studio's killer differentiator. Send any brief simultaneously to OpenAI, Claude, and Gemini, collect all three outputs, then run a synthesis pass that picks the strongest elements from each. Three models, one run, one winner.

| | |
|---|---|
| **Replaces** | Running the same prompt across several models by hand |
| **Providers** | OpenAI, Claude, Gemini, Notion |
| **Steps** | 6 (3-way parallel fanout) |
| **Trigger** | Manual |
| **Category** | AI |

---

### Social Listening & Content Pipeline

Every week Studio pulls top-performing content in your niche from YouTube, AI analyzes what formats and hooks are winning, generates a matching image with ComfyUI, writes platform-specific captions, and queues everything for review.

| | |
|---|---|
| **Replaces** | Content agency retainers for research, creation, and scheduling |
| **Providers** | Claude, OpenAI, YouTube, ComfyUI |
| **Steps** | 8 |
| **Trigger** | Schedule (weekly) |
| **Category** | Marketing |

---

### AI Reputation Manager

Your brand's comment section, always monitored. Every 6 hours Studio pulls new comments from YouTube, AI classifies each by sentiment and topic, drafts professional responses to negatives, logs everything, and sends a digest.

| | |
|---|---|
| **Replaces** | Commercial reputation-monitoring tooling |
| **Providers** | Gemini, Claude, YouTube, Airtable, Telegram |
| **Steps** | 7 |
| **Trigger** | Schedule (every 6 hours) |
| **Category** | Marketing |

---

### AI SMS Lead Qualifier

Every inbound SMS to your Twilio number gets an instant AI response. Claude reads the message, determines intent, pulls context from Airtable if the number is known, crafts a natural reply, and sends it - all in under 5 seconds.

| | |
|---|---|
| **Replaces** | First-touch lead qualification by SMS tools or live agents |
| **Providers** | Claude, Twilio, Airtable, Telegram |
| **Steps** | 6 |
| **Trigger** | Webhook |
| **Category** | Sales |

---

### Competitive Intelligence Report

Deep research on any company, competitor, or market - fully automated. Perplexity runs real-time web research, Claude synthesizes findings into a structured intelligence brief, and the final report lands in Notion and your inbox.

| | |
|---|---|
| **Replaces** | Analyst time and commercial competitive-intelligence tooling |
| **Providers** | Claude, OpenAI, Perplexity, Notion, Google |
| **Steps** | 6 (3-way parallel research) |
| **Trigger** | Schedule (weekly) or Manual |
| **Category** | Research |

---

### AI Video Ad Generator

Brief in, three ad variants out. AI writes three distinct script angles, ComfyUI generates matching visuals, Chatterbox narrates them, and the video worker assembles three complete ad videos in one run. Review all three, publish the winner.

| | |
|---|---|
| **Replaces** | Video ad production and editing time |
| **Providers** | OpenAI, ComfyUI, SHS Audio |
| **Steps** | 13 (6-way parallel fanout) |
| **Trigger** | Manual |
| **Category** | Video |

---

## Guides

| Guide | Description |
|-------|-------------|
| [Scene Templates](shs/scene-templates.md) | How scene templates drive video composition |
| [Run Once Per Item](shs/item-groups.md) | How list inputs are paired and iterated across step runs - covers single vs. list inputs, zip behavior, chop vs. loop |

---

## Provider Coverage

Studio workflows use a range of providers. Per-provider documentation lives in `docs/providers/`.

| Provider | Workflows | Type |
|----------|-----------|------|
| OpenAI | 10 | AI (Cloud) |
| Claude | 8 | AI (Cloud) |
| ComfyUI | 4 | Image Generation (Local) |
| Google | 5 | SaaS |
| Airtable | 5 | SaaS |
| Telegram | 5 | Communication |
| Notion | 5 | SaaS |
| Gemini | 2 | AI (Cloud) |
| Perplexity | 3 | AI (Cloud) |
| YouTube | 3 | Social |
| SHS Audio | 3 | TTS (Local) |
| SHS Video | 1 | Video (Local) |
| Core | 3 | Workflow Control |
| Twilio | 1 | Communication |
| Leonardo AI | 0 | AI (Cloud) |
| Stripe | 0 | Payments |
