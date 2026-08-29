# Self-Host Studio Audio Provider

Generate speech audio locally using Chatterbox TTS with voice cloning support.

## How It Works

The SHS Audio provider runs entirely on your infrastructure using Chatterbox TTS as the backend. It provides:

- **Text to Speech** - Convert text to natural-sounding audio
- **Voice Cloning** - Clone any voice from a ~10 second reference clip
- **Turbo Mode** - Faster generation with the 350M parameter model
- **Paralinguistic Tags** - Add laughter, sighs, and other expressions
- **No API Costs** - Run unlimited generations on your hardware

## Authentication

**No credentials required** - This is an internal provider that runs on your Studio instance.

### Requirements

Your Studio instance needs:
- Chatterbox TTS model downloaded
- Studio audio worker running
- GPU recommended for faster generation (CPU works but is slower)

## Available Services

### Text to Speech

Generate speech audio from text using the full Chatterbox model.

| Parameter | Description |
|-----------|-------------|
| **text** | The text to convert to speech |
| **audio_prompt_path** | Reference audio clip for voice cloning (~10s, optional) |
| **exaggeration** | Expressiveness: 0 = neutral, 1 = highly expressive (default 0.5) |
| **cfg_weight** | Classifier-free guidance weight (default 0.5) |

### Text to Speech (Turbo)

Faster TTS with the 350M parameter turbo model. Supports paralinguistic tags for expressive speech.

| Parameter | Description |
|-----------|-------------|
| **text** | Text to convert (supports paralinguistic tags) |
| **audio_prompt_path** | Reference audio for voice cloning (optional) |
| **exaggeration** | Expressiveness level (default 0.5) |
| **cfg_weight** | Guidance weight (default 0.5) |

**Paralinguistic Tags (Turbo only):**
- `[laugh]` - Laughter
- `[chuckle]` - Light chuckle
- `[cough]` - Coughing
- `[sigh]` - Sighing
- `[gasp]` - Gasping
- `[groan]` - Groaning
- `[yawn]` - Yawning

**Example:** `"Well [laugh] that's certainly one way to do it [sigh]"`

---

## Example Workflows

### Simple Text to Speech

**Use case:** Convert text content to audio

```
Trigger: Manual
  |
Step 1: Text to Speech
  - Text: "Welcome to our product demo."
  - Exaggeration: 0.5
```

### AI-Generated Narration

**Use case:** Generate script with AI, then narrate

```
Trigger: Manual (topic input)
  |
Step 1: Claude/OpenAI
  - Generate narration script from topic
  |
Step 2: Text to Speech
  - Text: {{ step1.content }}
```

### Voice Cloned Narration

**Use case:** Generate audio in a specific voice

```
Trigger: Manual
  |
Step 1: Text to Speech
  - Text: "Your custom narration here"
  - Audio Prompt Path: /path/to/reference-voice.wav
  - Exaggeration: 0.3
```

---

## Multi-Step Workflows

> **Plus catalog:** Plus workflows and video walkthroughs come with a [SelfHost Innovators membership](https://www.skool.com/selfhostinnovators).

### Multi-Segment Audio Pipeline

Generate long-form audio by splitting text into segments:

1. **Split Text** - Break script into paragraphs
2. **Generate Audio** - TTS for each segment
3. **Concatenate** - Merge audio files
4. **Export** - Save final audio file

### Podcast Episode Generator

Create full podcast episodes from text:

1. **Generate Script** - AI writes conversational script
2. **Host Voice** - TTS with host voice clone
3. **Guest Voice** - TTS with different voice
4. **Mix Audio** - Combine with intro/outro music
5. **Export** - Final podcast episode

---

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Worker not available" | Ensure the audio worker is running |
| "Model not found" | Download Chatterbox TTS model files |
| "Out of memory" | Use Turbo model or reduce text length |
| "Reference audio too short" | Voice cloning needs ~10 seconds of clear speech |

## Tips

1. **Voice Cloning** - Use a clean, noise-free 10-second clip of the target voice
2. **Exaggeration** - Start at 0.5 and adjust; too high can distort output
3. **Turbo vs Standard** - Turbo is ~2x faster but slightly lower quality
4. **Paralinguistic Tags** - Only work with the Turbo model
5. **Long Text** - Split text into paragraph-sized chunks for best quality
