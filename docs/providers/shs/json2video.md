# JSON2Video Provider

Create videos programmatically using JSON definitions.

## How It Works

JSON2Video is a powerful API for programmatic video generation. Define your video structure in JSON and the service renders it in the cloud:

- **Scenes & Elements** - Videos are built from scenes containing elements (video, image, audio, text)
- **Text-to-Speech** - Multiple TTS providers including Azure and ElevenLabs
- **AI Image Generation** - Generate images within your video using AI
- **Subtitles** - Auto-generate or manual captions
- **Transitions** - Scene transitions and visual effects

Videos are rendered asynchronously - you submit a job and get notified when complete.

## Authentication

JSON2Video uses **API Key** authentication.

### Setup

1. Go to [json2video.com/dashboard](https://json2video.com/dashboard)
2. Find your API key in account settings
3. In Studio, go to **Providers > JSON2Video > Credentials**
4. Click **Add Credential** and paste your API key

## Available Services

### Generate Video

Submit a video render job.

| Parameter | Description |
|-----------|-------------|
| **scenes** | Array of scene objects |
| **width** | Output width (default 1920) |
| **height** | Output height (default 1080) |
| **quality** | low, medium, high, very_high |
| **draft** | Quick preview mode (no credits) |
| **variables** | Global variables for templates |
| **exports** | Delivery destinations (webhook, FTP, email) |

**Scene Structure:**
```json
{
  "duration": 5,
  "background-color": "#000000",
  "elements": [
    { "type": "video", "src": "https://..." },
    { "type": "text", "text": "Hello World" }
  ]
}
```

**Mapping files from a previous step:** map an element's `src` to a previous
step's file output directly (e.g. an image step's `downloaded_files`). You do
not pick a URL sub-field - Studio fills in the fetch URL for each file
automatically. When you map a **list** of files, all of them are sent together
and combined into one video (the step does not run once per file); use the
element's `count` to control how many are placed per scene.

### Get Video

Check job status and get download URL.

| Parameter | Description |
|-----------|-------------|
| **project** | Project ID from Generate Video response |

**Status Values:**
- `pending` - Queued for processing
- `running` - Currently rendering
- `done` - Complete, URL available
- `error` - Failed, check message

---

## Element Types

### Video

```json
{
  "type": "video",
  "src": "https://example.com/video.mp4",
  "duration": 10,
  "volume": 0.5,
  "position": "center-center"
}
```

### Image

```json
{
  "type": "image",
  "src": "https://example.com/image.jpg",
  "duration": 5,
  "pan": "right",
  "zoom": 1.2
}
```

**AI-Generated Image:**
```json
{
  "type": "image",
  "prompt": "Beautiful sunset over mountains",
  "ai_model": "flux-pro",
  "duration": 5
}
```

### Text

```json
{
  "type": "text",
  "text": "Hello World",
  "style": "001",
  "position": "center-center",
  "settings": {
    "font-family": "Montserrat",
    "font-size": "48px",
    "color": "#FFFFFF"
  }
}
```

### Voice (TTS)

```json
{
  "type": "voice",
  "text": "Welcome to our video",
  "voice": "en-US-AriaNeural",
  "model": "azure"
}
```

### Audio

```json
{
  "type": "audio",
  "src": "https://example.com/music.mp3",
  "volume": 0.3,
  "loop": -1
}
```

---

## Basic Workflows

### Social Media Video

**Use case:** Generate videos for social posts

```
Trigger: New post data
  ↓
Step 1: JSON2Video Generate
  - Scenes:
    - Image with title overlay
    - Content slides
    - Call to action
  - Width: 1080
  - Height: 1920 (vertical)
  ↓
Step 2: Wait for completion (poll or webhook)
  ↓
Step 3: Upload to social platform
```

### Product Demo Video

**Use case:** Create product showcases

```
Trigger: Product updated
  ↓
Step 1: JSON2Video Generate
  - Scene 1: Product image with zoom
  - Scene 2: Feature highlights (text overlays)
  - Scene 3: Pricing with CTA
  - Voice: AI narration of features
  ↓
Step 2: Save to product library
```

### News Recap Video

**Use case:** Automated news summaries

```
Trigger: Daily schedule
  ↓
Step 1: Fetch top stories
  ↓
Step 2: Generate voice scripts
  ↓
Step 3: JSON2Video Generate
  - For each story: image + voice + text
  - Auto-generated subtitles
  ↓
Step 4: Post to YouTube
```

---

## Advanced Workflows

> **Skool Members:** Access advanced video templates and production workflows at [skool.com/selfhost](https://skool.com/selfhost)

### Dynamic Video Template System

Personalized videos at scale:

1. **Load Template** - Base video structure from database
2. **Merge Data** - Replace variables with customer data
3. **Generate Batch** - Submit multiple personalized videos
4. **Track Progress** - Monitor all jobs
5. **Deliver** - Email or store completed videos

### Multi-Language Video Pipeline

Create localized content:

1. **Create Base Video** - Source language version
2. **Translate Text** - AI translation of scripts
3. **Generate Voices** - TTS in each language
4. **Render Versions** - Parallel video generation
5. **Organize** - Store by language/region

### Interactive Video Campaign

Branching video experiences:

1. **Generate Segments** - Create video modules
2. **Define Paths** - Map user choices to segments
3. **Build Player** - Interactive video player
4. **Track Engagement** - Log user paths
5. **Optimize** - A/B test different paths

---

## Subtitle Configuration

Subtitles are configured as a movie-level element in the `elements` array (not per-scene). JSON2Video auto-transcribes audio when `language` is set. All styling and behavior lives under the `settings` object (keys use hyphens, matching the JSON2Video API spec exactly).

### Auto-Generated (Whisper)

```json
{
  "elements": [
    {
      "type": "subtitles",
      "language": "en",
      "model": "whisper",
      "settings": {
        "style": "classic",
        "position": "bottom-center",
        "font-family": "Montserrat",
        "font-size": 24,
        "line-color": "#FFFFFF",
        "outline-color": "#000000",
        "outline-width": 2
      }
    }
  ]
}
```

### Full Settings Reference

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `style` | string | `classic` | `classic`, `classic-progressive`, `classic-one-word`, `boxed-line`, `boxed-word` |
| `position` | string | `bottom-center` | 9-grid (`top-left` … `bottom-right`), `mid-top-center`, `mid-bottom-center`, or `custom` |
| `x`, `y` | integer | 0 | Pixel coordinates (only when `position` is `custom`) |
| `font-family` | string | `Arial` | Any Google Font or a font supplied via `font-url` |
| `font-size` | int/string | 5% of width | Typical range 90-150 |
| `font-weight` | string | - | Numeric string `100`-`900` (Google Fonts) |
| `font-url` | uri | - | Custom TTF font URL (font-family must match) |
| `all-caps` | boolean | false | Render subtitles in uppercase |
| `line-color` | color | `#FFFFFF` | Non-active word color |
| `word-color` | color | `#FFFF00` | Currently-spoken word color |
| `box-color` | color | `#000000` | Box behind subtitles (depends on style) |
| `outline-color` | color | `#000000` | Outline stroke color |
| `outline-width` | integer | 0 | Outline thickness (0+) |
| `shadow-color` | color | `#000000` | Drop-shadow color |
| `shadow-offset` | integer | 0 | Shadow offset in pixels |
| `max-words-per-line` | integer | 4 | Set to 1 for one-word-at-a-time |
| `keywords` | string[] | - | Brand names / technical terms for better transcription (default model only). Array of strings. |
| `replace` | object | - | Post-transcription word substitutions as `{mis-transcribed: correct, ...}`. |

### Manual Captions (ASS format)

```json
{
  "elements": [
    {
      "type": "subtitles",
      "language": "auto",
      "captions": "[Script Info]\nScriptType: v4.00+\n..."
    }
  ]
}
```

## Completion Mode (Studio)

The Generate Video step supports two completion modes (set per step in the flow editor):

- **Poll for result** (`get`) — Studio polls the render status until it's `done`, then returns the video. Synchronous; holds a worker for the render duration.
- **Wait for provider callback** (`webhook`) — Studio mints a **unique callback URL per render**, injects it into the request automatically, and releases the worker. json2video posts the finished video back to that URL and Studio downloads it and continues.

Webhook mode needs no configuration on json2video and no callback key: the callback URL is generated per render (it carries an unguessable token that is the only auth, since json2video sends no signature), so concurrent runs and fan-out iterations all self-route. The deployment must have `API_BASE_URL` set to an absolute, publicly reachable URL — json2video calls it directly, so a relative URL won't work (Studio fails the step at enqueue if it's unset).

The inbound callback body is **flat** (not nested under `movie` like the GET status response): success carries `$.url` (the MP4); a failed render fires with the url empty.

In webhook mode the step also exposes a **Check Status** button (manual poll fallback via the GET status endpoint) in case a callback is delayed or lost. In poll mode no button is needed — Studio polls automatically.

## Export Destinations

Export destinations deliver the finished render to an endpoint of your choice (webhook, FTP, email). These are **separate from the Completion Mode above**: the Completion Mode is how Studio itself detects the render finished (and is auto-managed in webhook mode); an export `webhook` destination is an *additional* delivery to your own system.

Destinations are nested under `exports[].destinations[]` and use json2video's native field names (`type`, `endpoint`, `host`, etc.). Studio's form collects them flat (one row per destination) and the request transform wraps them into this shape automatically.

How outputs interact with completion mode:

- **Webhook URL left blank** (webhook output row) — Studio auto-generates the endpoint. In **webhook completion mode** this is the same callback Studio already injects, so an empty webhook row is redundant; you don't need to add one.
- **Webhook URL filled in** — json2video also POSTs to your endpoint. In **webhook completion mode** the render then has *two* webhook destinations (yours + Studio's auto callback) and **both fire** — the intended "notify a second listener" (e.g. monitoring) case, not a duplicate bug.
- **Poll completion mode + your own webhook output** — Studio doesn't inject a callback (it polls for completion), so json2video delivers only to your endpoint. Output and completion are independent; both work.

### Webhook Callback

```json
{
  "exports": [{
    "destinations": [{
      "type": "webhook",
      "endpoint": "https://your-server.com/video-complete",
      "content-type": "json"
    }]
  }]
}
```

### FTP Upload

```json
{
  "exports": [{
    "destinations": [{
      "type": "ftp",
      "host": "ftp.example.com",
      "port": 21,
      "username": "user",
      "password": "pass",
      "remote-path": "./videos/",
      "file": "__random__.mp4"
    }]
  }]
}
```

### Email Delivery

```json
{
  "exports": [{
    "destinations": [{
      "type": "email",
      "to": "recipient@example.com",
      "subject": "Your video is ready",
      "message": "Download your video: __filename__"
    }]
  }]
}
```

json2video's email destination has no `from` field — the sender is fixed by your json2video account.

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Invalid scene structure" | Check JSON syntax and required fields |
| "Media not accessible" | Verify URLs are publicly accessible |
| "Render timeout" | Reduce video complexity or duration |
| "Insufficient credits" | Purchase more render time |

## Tips

1. **Draft Mode** - Use `draft: true` for quick previews without using credits
2. **Caching** - Enable `cache: true` for faster re-renders
3. **Variables** - Use `{{ var.name }}` syntax for dynamic content
4. **Webhooks** - Set up webhook exports to avoid polling
5. **Quality** - Use "medium" for drafts, "high" for production
