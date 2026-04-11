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

Subtitles are configured as a movie-level element in the `elements` array (not per-scene). JSON2Video auto-transcribes audio when `language` is set.

### Auto-Generated (Whisper)

```json
{
  "elements": [
    {
      "type": "subtitles",
      "language": "en",
      "settings": {
        "font-family": "Montserrat",
        "font-size": 24,
        "style": "classic",
        "position": "bottom-center",
        "line-color": "#FFFFFF",
        "outline-color": "#000000",
        "outline-width": 2
      }
    }
  ]
}
```

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

## Export Destinations

### Webhook Callback

```json
{
  "exports": [{
    "destination_type": "webhook",
    "webhook_url": "https://your-server.com/video-complete"
  }]
}
```

### FTP Upload

```json
{
  "exports": [{
    "destination_type": "ftp",
    "ftp_host": "ftp.example.com",
    "ftp_username": "user",
    "ftp_password": "pass",
    "ftp_path": "/videos/"
  }]
}
```

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
