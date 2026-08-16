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

- ComfyUI installed and running (any machine the worker can reach over HTTP)
- The model files for the packages you install (table below)
- The custom nodes for the packages you install (table below)
- A GPU: 8 GB VRAM covers the GGUF pipeline; Klein and bf16 Schnell want more.
  Apple Silicon runs ComfyUI natively (MPS); the GGUF pipeline is the low-memory pick.
- A Studio ComfyUI worker connected to your instance (quickstart below)

## Worker Quickstart

The worker is a small dispatcher: it claims jobs from your Studio instance,
sends the workflow graph to ComfyUI, and uploads the results. ComfyUI is a
separate process that runs wherever your GPU is. The worker finds it via
`SHS_COMFYUI_URL`.

Three layouts, all supported:

1. Worker and ComfyUI on the same machine: `SHS_COMFYUI_URL=http://127.0.0.1:8188`
2. Worker in Docker, ComfyUI on the host: `SHS_COMFYUI_URL=http://host.docker.internal:8188`
3. Worker anywhere, ComfyUI on your GPU box: `SHS_COMFYUI_URL=http://<gpu-host>:8188`

On macOS, run ComfyUI natively (Docker cannot use the Apple GPU) and prefer the
native worker.

Every worker needs four values. `studio-console worker-kit` prints them filled
in for your instance; SHS_PUBLIC_BASE_URL and SHS_WORKER_SHARED_SECRET come
from your Studio workspace `.env`.

### Native (pip)

```
uv tool install "studio-workers[comfyui]"     # Python 3.12

SHS_API_BASE_URL=http://127.0.0.1:8000 \
SHS_PUBLIC_BASE_URL=<your public API URL> \
SHS_WORKER_SHARED_SECRET=<from workspace .env> \
SHS_WORKSPACE_ROOT=~/studio-worker-scratch \
SHS_COMFYUI_URL=http://127.0.0.1:8188 \
studio-workers run --type comfyui-image
```

`studio-workers doctor` checks the host first if anything misbehaves.

### Docker

```
docker run -d --name studio-worker-comfyui \
  -e SHS_API_BASE_URL=http://<api-host>:80 \
  -e SHS_PUBLIC_BASE_URL=<your public API URL> \
  -e SHS_WORKER_SHARED_SECRET=<from workspace .env> \
  -e SHS_WORKSPACE_ROOT=/workspace \
  -e SHS_COMFYUI_URL=http://host.docker.internal:8188 \
  ghcr.io/selfhosthub/studio-worker-comfyui:<studio version>
```

When the worker registers, it syncs the installed ComfyUI packages from your
instance automatically and picks up catalog changes on its own. Watch its log
for `Synced comfyui package` and, on a job, `Using catalog package`.

## Model Files

Download only the files for the packages you install. Find your package here,
then get those files from the table below.

### Per-package file sets

| Package | Files |
|---------|-------|
| Text to Image (Flux.1 Schnell GGUF) | flux1-schnell-Q4_K_S.gguf, t5xxl_fp8_e4m3fn.safetensors, clip_l.safetensors, ae.safetensors |
| Text to Image (Flux.1 Schnell), FP8 | flux1-schnell-fp8-e4m3fn.safetensors, t5xxl_fp8_e4m3fn.safetensors, clip_l.safetensors, ae.safetensors |
| Text to Image (Flux.1 Schnell), bf16 | flux1-schnell.safetensors, t5xxl_fp16.safetensors, clip_l.safetensors, ae.safetensors |
| Text to Image (Flux 2 Klein), 4B | flux-2-klein-4b.safetensors, qwen_3_4b.safetensors, flux2-vae.safetensors |
| Text to Image (Flux 2 Klein), 9B | flux-2-klein-9b-fp8.safetensors, qwen_3_8b_fp8mixed.safetensors, flux2-vae.safetensors |
| Image Edit (Flux 2 Klein) | flux-2-klein-4b.safetensors, qwen_3_4b.safetensors, flux2-vae.safetensors |

Every package manifest lists the same set under `required_models`.

### All model files

Directory is where ComfyUI Manager's Model Manager saves the file, so installing
through Manager puts it in the right place. Manager does not carry the FP8
Schnell build or any Flux 2 Klein file, so those go in the plain type folder
shown. The file name links to its Hugging Face repository. Klein diffusion
weights are gated: downloading requires an account and accepting the license
terms in the browser first. Everything else is an open download.

| File | Directory | Size | Access | Used by |
|------|-----------|------|--------|---------|
| [clip_l.safetensors](https://huggingface.co/comfyanonymous/flux_text_encoders) | text_encoders | 0.2 GB | open | all Schnell |
| [ae.safetensors](https://huggingface.co/black-forest-labs/FLUX.1-schnell) | vae/FLUX1 | 0.3 GB | open | all Schnell |
| [flux1-schnell-Q4_K_S.gguf](https://huggingface.co/city96/FLUX.1-schnell-gguf) | diffusion_models/FLUX1 | 6.8 GB | open | Schnell GGUF |
| [t5xxl_fp8_e4m3fn.safetensors](https://huggingface.co/comfyanonymous/flux_text_encoders) | text_encoders/t5 | 4.9 GB | open | Schnell GGUF, FP8 |
| [flux1-schnell-fp8-e4m3fn.safetensors](https://huggingface.co/Kijai/flux-fp8) | diffusion_models | 11.9 GB | open | Schnell FP8 |
| [flux1-schnell.safetensors](https://huggingface.co/black-forest-labs/FLUX.1-schnell) | diffusion_models/FLUX1 | 23.8 GB | open | Schnell bf16 |
| [t5xxl_fp16.safetensors](https://huggingface.co/comfyanonymous/flux_text_encoders) | text_encoders/t5 | 9.8 GB | open | Schnell bf16 |
| [flux2-vae.safetensors](https://huggingface.co/Comfy-Org/vae-text-encorder-for-flux-klein-4b) | vae | 0.3 GB | open | all Klein |
| [flux-2-klein-4b.safetensors](https://huggingface.co/black-forest-labs/FLUX.2-klein-4B) | diffusion_models | 7.8 GB | gated | Klein 4B, Image Edit |
| [qwen_3_4b.safetensors](https://huggingface.co/Comfy-Org/vae-text-encorder-for-flux-klein-4b) | text_encoders | 8.0 GB | open | Klein 4B, Image Edit |
| [flux-2-klein-9b-fp8.safetensors](https://huggingface.co/black-forest-labs/FLUX.2-klein-9B) | diffusion_models | 9.4 GB | gated | Klein 9B |
| [qwen_3_8b_fp8mixed.safetensors](https://huggingface.co/Comfy-Org/vae-text-encorder-for-flux-klein-9b) | text_encoders | 8.7 GB | open | Klein 9B |

## Custom Nodes

Install into ComfyUI's `custom_nodes` directory (ComfyUI Manager or `git clone`),
then restart ComfyUI.

| Node pack | Required by | Repository |
|-----------|-------------|------------|
| ComfyUI-GGUF | Text to Image (Flux.1 Schnell GGUF) | https://github.com/city96/ComfyUI-GGUF |

The other packages run on ComfyUI's built-in nodes.

## Available Services

### Text to Image

| Parameter | Description |
|-----------|-------------|
| **prompt** | Text description of the image |
| **model** | Flux.1 Schnell (bf16/FP8/Q4) or Flux 2 Klein (4B/9B) |
| **dimensions** | Output size preset (1:1, 16:9, 9:16, 3:2, 2:3) |
| **styles** | Style presets to apply (see below) |
| **seed** | -1 for random, or a specific seed |
| **steps** | Diffusion steps (1 for Schnell, 8 for Klein defaults) |
| **batch_size** | Generate 1-4 images |
| **fast mode** | Generate small, upscale to target (on by default) |

### Image Edit

| Parameter | Description |
|-----------|-------------|
| **image** | Source image (from a previous step or URL) |
| **prompt** | How to transform the image |
| **negative_prompt** | What to avoid (optional) |
| **dimensions** | Output size preset |
| **seed** | -1 for random, or a specific seed |
| **steps** | Diffusion steps (Klein edit default 1, up to 20) |

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
