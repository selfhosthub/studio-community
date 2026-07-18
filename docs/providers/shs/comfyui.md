# Self-Host Studio ComfyUI Provider

Generate AI images locally using ComfyUI with Flux and Stable Diffusion models.

## How It Works

The SHS ComfyUI provider runs entirely on your infrastructure using ComfyUI as the backend. It provides:

- **Flux Models** - Fast, high-quality image generation
  - Flux.1 Schnell - 4-step generation, extremely fast
  - Flux.2 Klein - Image editing and transformation
- **Style Presets** - 100+ built-in style presets
- **No API Costs** - Run unlimited generations on your hardware
- **Full Control** - Your images, your data, your infrastructure

## Authentication

**No credentials required** - This is an internal provider that runs on your Studio instance.

### Requirements

Your Studio instance needs:
- ComfyUI installed and running
- Flux models downloaded
- GPU with sufficient VRAM (8GB+ recommended)
- Studio worker configured for ComfyUI queue

## Available Services

### Text to Image

Generate images from text prompts using Flux.1 Schnell.

| Parameter | Description |
|-----------|-------------|
| **prompt** | Text description of the image |
| **negative_prompt** | What to avoid (optional) |
| **styles** | Style presets to apply |
| **width** | Image width (512-1536) |
| **height** | Image height (512-1536) |
| **seed** | -1 for random, or specific seed |
| **steps** | Diffusion steps (4 optimal for Schnell) |
| **batch_size** | Generate 1-4 images |

### Image Edit

Transform existing images using Flux.2 Klein.

| Parameter | Description |
|-----------|-------------|
| **image** | Source image URL or path |
| **prompt** | How to transform the image |
| **negative_prompt** | What to avoid (optional) |
| **width** | Output width |
| **height** | Output height |
| **seed** | -1 for random, or specific seed |
| **steps** | Diffusion steps (8 optimal for Klein) |

---

## Style Presets

Apply artistic styles to your generations. Multiple styles can be combined.

### Categories

**Advertising:**
- ads-advertising, ads-automotive, ads-corporate
- ads-fashion editorial, ads-food photography
- ads-luxury, ads-real estate, ads-retail

**Art Styles:**
- artstyle-abstract, artstyle-art deco, artstyle-art nouveau
- artstyle-cubist, artstyle-expressionist, artstyle-impressionist
- artstyle-pop art, artstyle-renaissance, artstyle-surrealist
- artstyle-watercolor, artstyle-hyperrealism

**Cinematic:**
- cinematic-default, cinematic-diva
- sai-cinematic, sai-analog film

**Futuristic:**
- futuristic-cyberpunk cityscape, futuristic-sci-fi
- futuristic-vaporwave, futuristic-retro futurism

**Game Styles:**
- game-cyberpunk game, game-pokemon, game-minecraft
- game-retro arcade, game-rpg fantasy game

**Photography:**
- photo-film noir, photo-hdr, photo-neon noir
- photo-long exposure, photo-tilt-shift
- sai-photographic

**Digital Art:**
- sai-digital art, sai-3d-model, sai-anime
- sai-comic book, sai-pixel art, sai-isometric

### Style Merge Modes

When using multiple styles:

| Mode | Behavior |
|------|----------|
| **nested** | Last style dominant |
| **stacked** | First style dominant |
| **blended** | All styles equal weight |
| **minimal** | Styles without negatives |

---

## Example Workflows

### Product Image Generation

**Use case:** Generate product visuals

```
Trigger: New product added
  ↓
Step 1: SHS ComfyUI Text to Image
  - Prompt: "Professional product photo of {{ product.name }},
    white background, studio lighting, commercial photography"
  - Styles: ["sai-photographic", "ads-advertising"]
  - Width: 1024
  - Height: 1024
  ↓
Step 2: Save to product gallery
```

### Social Media Graphics

**Use case:** Create eye-catching visuals

```
Trigger: Content calendar event
  ↓
Step 1: SHS ComfyUI Text to Image
  - Prompt: "{{ post.visual_concept }}"
  - Styles: ["sai-digital art"]
  - Width: 1024
  - Height: 1024
  - Batch Size: 4
  ↓
Step 2: Save options for review
```

### Avatar Generation

**Use case:** Create profile images

```
Trigger: User avatar request
  ↓
Step 1: SHS ComfyUI Text to Image
  - Prompt: "Portrait of {{ style_description }}, centered face,
    detailed, high quality"
  - Styles: ["sai-anime"] or ["sai-photographic"]
  - Width: 512
  - Height: 512
```

---

## Multi-Step Workflows

> **Plus catalog:** Plus workflows and video walkthroughs come with a [SelfHost Innovators membership](https://www.skool.com/selfhostinnovators).

### Image Transformation Pipeline

Transform user photos with AI:

1. **Receive Upload** - User submits photo
2. **Image Edit** - Flux.2 Klein transformation
3. **Generate Variations** - Multiple style options
4. **Quality Check** - Validate outputs
5. **Deliver Results** - Return to user

### Batch Asset Generation

Generate assets at scale:

1. **Load Prompts** - Query prompt list
2. **Parallel Generation** - Multiple images per prompt
3. **Style Variations** - Apply different presets
4. **Organize Output** - Store with metadata
5. **Generate Gallery** - Create preview page

### Consistent Character Generation

Generate consistent characters:

1. **Create Base** - Initial character generation
2. **Extract Seed** - Save successful seed
3. **Generate Poses** - Same seed, different prompts
4. **Maintain Style** - Consistent style presets
5. **Build Library** - Character asset collection

---

## Image Edit Examples

### Style Transfer

Transform a photo into artwork:

```json
{
  "image": "https://example.com/photo.jpg",
  "prompt": "oil painting, impressionist style, brushstrokes visible",
  "steps": 8
}
```

### Object Transformation

Change elements in an image:

```json
{
  "image": "https://example.com/person.jpg",
  "prompt": "wearing a superhero costume, cape flowing",
  "steps": 10
}
```

### Background Change

Replace image background:

```json
{
  "image": "https://example.com/product.jpg",
  "prompt": "on a beach at sunset, tropical setting",
  "steps": 8
}
```

## Prompt Engineering Tips

### Structure
```
[Subject] + [Style modifiers] + [Quality keywords]
```

### Quality Keywords
```
high quality, detailed, sharp focus, 8K, professional
```

### Avoiding Issues
**Negative prompt suggestions:**
```
blurry, low quality, distorted, watermark, text,
deformed, ugly, bad anatomy, bad proportions
```

### Example Prompts

**Portrait:**
```
Portrait of a young woman with flowing auburn hair,
soft natural lighting, shallow depth of field,
professional photography, detailed eyes, sharp focus
```

**Landscape:**
```
Majestic mountain range at golden hour,
dramatic clouds, mist in valleys, ultra wide angle,
National Geographic quality, 8K resolution
```

**Product:**
```
Sleek modern smartphone floating in space,
dramatic lighting, reflections, commercial photography,
product shot, white background, studio lighting
```

## Troubleshooting

| Error | Solution |
|-------|----------|
| "ComfyUI not responding" | Check ComfyUI is running |
| "Model not found" | Download required models |
| "Out of VRAM" | Reduce batch size or image dimensions |
| "Slow generation" | Ensure GPU is being used |

## Tips

1. **Flux.1 Schnell** - Optimal at 4 steps, more steps don't improve quality
2. **Flux.2 Klein** - Use 8+ steps for image editing
3. **Seeds** - Save seeds from good results for reproducibility
4. **Batch Size** - Generate multiple to find best result
5. **Style Stacking** - Combine 2-3 complementary styles
6. **Negative Prompts** - Essential for avoiding common artifacts
