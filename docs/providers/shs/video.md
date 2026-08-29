# Self-Host Studio Video Provider

Create videos locally using FFmpeg with AI-powered subtitles via Whisper.

## How It Works

The SHS Video provider runs entirely on your infrastructure - no external API calls for video rendering. It combines:

- **FFmpeg** - Professional-grade video processing
- **Whisper STT** - OpenAI's speech-to-text for auto-subtitles
- **Ken Burns Effects** - Pan and zoom animations for images
- **Scene Composition** - Layer images, video, and audio

Videos are processed by the Studio worker on your server, with results stored in your configured storage.

## Authentication

**No credentials required** - This is an internal provider that runs on your Studio instance.

### Requirements

Your Studio worker needs:
- FFmpeg installed and in PATH
- Whisper model downloaded (for subtitle generation)
- Sufficient disk space for video processing
- GPU recommended for faster Whisper processing

## Available Services

### Create Video

Compose a video from scenes containing images, video clips, and audio.

| Parameter | Description |
|-----------|-------------|
| **scenes** | Array of scenes with elements |
| **width** | Output width (default 1920) |
| **height** | Output height (default 1080) |
| **framerate** | FPS: 24 (cinematic), 30, 60 |
| **quality** | low, medium, high |

**Subtitle Options:**

| Parameter | Description |
|-----------|-------------|
| **subtitles_source** | none, auto (Whisper), captions, src, text |
| **subtitles_style** | standard or karaoke (word highlighting) |
| **subtitles_font_family** | Font selection |
| **subtitles_position** | top, center, bottom |

**Default Settings:**

| Parameter | Description |
|-----------|-------------|
| **default_duration** | Element duration when not specified |
| **default_zoom_start/end** | Ken Burns zoom levels |
| **resize** | cover, contain, fill, natural |
| **padding_color** | Letterbox background color |

---

## Element Types

### Image

```json
{
  "type": "image",
  "src": "https://example.com/photo.jpg",
  "duration": 5,
  "zoom_start": 1.0,
  "zoom_end": 1.2,
  "pan": "right",
  "resize": "cover"
}
```

**Ken Burns Effect:**
- `zoom_start` / `zoom_end` - Animate from one zoom level to another
- `pan` - Camera movement direction (left, right, up, down, corners)

### Video Clip

```json
{
  "type": "video",
  "src": "https://example.com/clip.mp4",
  "duration": -1,
  "volume": 0.8,
  "muted": false
}
```

### Audio Track

```json
{
  "type": "audio",
  "src": "https://example.com/music.mp3",
  "volume": 0.3
}
```

---

## Example Workflows

### Photo Slideshow with Music

**Use case:** Create slideshow from photo album

```
Trigger: Photo album ready
  ↓
Step 1: SHS Video Create
  - Scenes: (one per photo)
    - type: image
    - src: {{ photo.url }}
    - duration: 5
    - zoom_start: 1.0
    - zoom_end: 1.1
    - pan: right
  - Background audio: music.mp3
  - Quality: high
  ↓
Step 2: Upload to storage
```

### Podcast Video with Waveform

**Use case:** Turn audio podcast into video

```
Trigger: New podcast episode
  ↓
Step 1: SHS Video Create
  - Scene:
    - Background image: podcast cover
    - Audio: {{ episode.audio_url }}
    - Subtitles: auto (Whisper)
  - Width: 1920
  - Height: 1080
  ↓
Step 2: Upload to YouTube
```

### Social Media Clip

**Use case:** Create vertical videos for social

```
Trigger: Content request
  ↓
Step 1: SHS Video Create
  - Width: 1080
  - Height: 1920
  - Scenes:
    - Hook image (3 sec)
    - Content images with text
    - CTA slide
  - Subtitles: karaoke style
  ↓
Step 2: Post to platform
```

---

## Multi-Step Workflows

> **Plus catalog:** Plus workflows and video walkthroughs come with a [SelfHost Innovators membership](https://www.skool.com/selfhostinnovators).

### Automated Course Video Production

Create educational content:

1. **Parse Script** - Extract narration and visual cues
2. **Generate Audio** - TTS for narration
3. **Match Visuals** - Select/generate images per section
4. **Compose Video** - SHS Video with all elements
5. **Add Subtitles** - `subtitles_source: auto` burns captions into the render
6. **Deliver** - Upload to course platform

### Multi-Language Video Pipeline

Localize videos efficiently:

1. **Translate Script** - AI translation of the source script
2. **Generate Voices** - SHS Audio TTS per language
3. **Render Variants** - One video step per language, same scenes, swapped audio

### Batch Video Processing

Process video content at scale:

1. **Queue Jobs** - Load video specifications
2. **Parallel Processing** - Multiple workers
3. **Quality Check** - Validate outputs
4. **Organize** - Store with metadata
5. **Notify** - Alert on completion

---

## Subtitle Configuration

### Auto-Generated (Whisper)

```json
{
  "subtitles_source": "auto",
  "subtitles_language": "en",
  "subtitles_style": "karaoke",
  "subtitles_font_family": "Luckiest Guy",
  "subtitles_font_size": 48,
  "subtitles_all_caps": true,
  "subtitles_position": "bottom",
  "subtitles_highlight_color": "FFFF00"
}
```

### Manual Captions

```json
{
  "subtitles_source": "captions",
  "subtitles_captions": [
    { "start": 0, "end": 2.5, "text": "Welcome to our channel" },
    { "start": 2.5, "end": 5, "text": "Today we're covering..." }
  ]
}
```

### Available Fonts

- **Luckiest Guy** - Bold display (default)
- **Roboto** - Clean sans-serif
- **Comic Neue** - Casual friendly
- **Lobster** - Script style
- **Oswald** - Condensed headers
- **Pacifico** - Brush script
- **Permanent Marker** - Handwritten
- **Noto Sans** (KR, SC, TC) - Asian languages

## Visual Effects

### Ken Burns Pan & Zoom

```json
{
  "type": "image",
  "src": "photo.jpg",
  "duration": 8,
  "zoom_start": 1.2,
  "zoom_end": 1.0,
  "pan": "left"
}
```

**Pan Directions:** left, right, up, down, top-left, top-right, bottom-left, bottom-right

### Positioning

```json
{
  "horizontal_position": "center",
  "vertical_position": "middle",
  "resize": "contain"
}
```

**Resize Modes:**
- `cover` - Fill frame, crop excess
- `contain` - Fit in frame, letterbox
- `fill` - Stretch to fill
- `custom` - Use explicit width/height

## Scene Groups

Scene groups let you build multi-scene videos from arrays of content. Instead of manually creating individual scenes, you provide arrays of images and audio and the worker expands them into scenes automatically.

### Static Scene vs. Scene Group

```json
// Static scene - one set of elements, one clip
{
  "type": "scene",
  "elements": [
    { "type": "image", "src": "intro.jpg", "duration": 3 }
  ]
}

// Scene group - element templates expanded into N scenes
{
  "type": "item_group",
  "repeat": 5,
  "elements": [
    {
      "type": "image",
      "src": ["img1.jpg", "img2.jpg", ..., "img15.jpg"],
      "count": 3,
      "duration": -1
    },
    {
      "type": "audio",
      "src": ["narration1.mp3", ..., "narration5.mp3"],
      "count": 1
    }
  ]
}
```

Both types can be mixed in a single timeline (e.g., static intro → scene group → static outro).

### Circular Stacking

Each element's `src` array is a circular stack. The worker pulls `count` items per scene, advancing through the array and wrapping when exhausted:

```
src = [A, B, C, D, E, F, G, H, I, J, K, L, M, N, O]  (15 items)
count = 3, repeat = 5

Scene 0: [A, B, C]
Scene 1: [D, E, F]
Scene 2: [G, H, I]
Scene 3: [J, K, L]
Scene 4: [M, N, O]
```

If the array doesn't divide evenly, it wraps to the beginning.

### Auto-Duration from Audio

Set `duration: -1` on images with audio in the group. The worker calculates display time from the audio duration divided by the number of images.

### Output Modes

- **Single video** (default) - all scenes concatenate into one file
- **Per-group** (`output_per_group: true`) - each scene uploaded separately as a `downloaded_files` array

### Multi-Platform Pattern

Pair scene groups with multiple video steps for different formats. One step renders 1920x1080 (YouTube), another renders 1080x1920 (TikTok/Reels), both consuming the same scene group data.

For the full technical architecture (circular stack algorithm, processing pipeline, design decisions), see the video scene groups documentation in the main Studio repository's `docs/architecture/` directory.

---

## Troubleshooting

| Error | Solution |
|-------|----------|
| "FFmpeg not found" | Install FFmpeg on worker server |
| "Whisper model not found" | Download Whisper model |
| "Out of memory" | Process shorter segments or add RAM |
| "Slow processing" | Use GPU for Whisper, reduce quality |

## Tips

1. **GPU Acceleration** - Whisper runs much faster with CUDA
2. **Quality vs Speed** - Use "medium" for drafts, "high" for final
3. **Ken Burns** - Set zoom_start > 1 for pan headroom
4. **Audio Sync** - Match audio duration to scene duration
5. **Memory** - Process long videos in segments
