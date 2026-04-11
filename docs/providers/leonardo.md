# Leonardo AI Provider

Generate high-quality AI images using Leonardo.ai's models.

## How It Works

Leonardo.ai is a powerful AI image generation platform offering:

- **Multiple Models** - Leonardo, Stable Diffusion XL, anime, realistic styles
- **Advanced Features** - Prompt Magic, Alchemy for enhanced quality
- **Image-to-Image** - Transform existing images with AI
- **Upscaling** - Increase resolution of generated images

Leonardo uses a credit system where different features and models consume varying amounts of credits.

## Authentication

Leonardo uses **API Key** authentication.

### Setup

1. Go to [app.leonardo.ai](https://app.leonardo.ai/)
2. Sign up or log in
3. Go to your **Profile** > **API Access**
4. Copy your API key
5. In Studio, go to **Providers > Leonardo AI > Credentials**
6. Click **Add Credential** and paste your API key

## Available Services

### Generate Image

Create images from text prompts.

| Parameter | Description |
|-----------|-------------|
| **prompt** | Text description of the image |
| **negative_prompt** | What to avoid in the image |
| **model** | Leonardo model to use |
| **width** | Image width (512-1024) |
| **height** | Image height (512-1024) |
| **num_images** | Number of images to generate (1-8) |
| **guidance_scale** | Prompt adherence (1-20, higher = more literal) |
| **seed** | Reproducibility seed |

### Upscale Image

Increase resolution of generated images.

| Parameter | Description |
|-----------|-------------|
| **image_id** | ID of image to upscale |
| **scale** | Upscale factor (2x, 4x) |

### List Models

Get available generation models and their capabilities.

---

## Basic Workflows

### Product Image Generation

**Use case:** Generate product mockups

```
Trigger: New product added
  ↓
Step 1: Leonardo Generate Image
  - Prompt: "Professional product photography of {{ product.name }},
    white background, studio lighting, 4K, detailed"
  - Model: Leonardo Diffusion XL
  - Width: 1024
  - Height: 1024
  ↓
Step 2: Save to storage
```

### Social Media Content

**Use case:** Generate visuals for posts

```
Trigger: Content calendar event
  ↓
Step 1: Leonardo Generate Image
  - Prompt: "{{ post.visual_prompt }}, vibrant colors,
    social media style, eye-catching"
  - Num Images: 4
  ↓
Step 2: Save options to review
```

### Avatar Generation

**Use case:** Create profile images

```
Trigger: User requests avatar
  ↓
Step 1: Leonardo Generate Image
  - Prompt: "Portrait of {{ user.description }},
    digital art style, detailed face"
  - Model: Anime Model
  - Width: 512
  - Height: 512
```

---

## Advanced Workflows

> **Skool Members:** Access advanced Leonardo workflows and model guides at [skool.com/selfhost](https://skool.com/selfhost)

### Brand Asset Pipeline

Generate consistent brand imagery:

1. **Load Brand Guidelines** - Colors, style preferences
2. **Generate Variations** - Multiple images with brand prompt
3. **Upscale Best** - Increase resolution on selected
4. **Store Assets** - Save to brand asset library
5. **Generate Metadata** - AI-generated alt text

### Image Transformation Workflow

Transform user photos:

1. **Receive Upload** - User submits photo
2. **Analyze Content** - Describe image elements
3. **Generate Transformation** - img2img with style prompt
4. **Compare & Select** - Generate multiple options
5. **Deliver Result** - Send transformed image

### Batch Content Generation

Generate images at scale:

1. **Load Prompts** - Query prompt list from database
2. **Parallel Generation** - Generate images in batches
3. **Quality Check** - Validate outputs
4. **Organize Results** - Save with metadata
5. **Report Status** - Log success/failure rates

---

## Model Selection Guide

| Model | Best For | Style |
|-------|----------|-------|
| **Leonardo Diffusion XL** | General purpose | Versatile |
| **Leonardo Creative** | Artistic images | Creative |
| **DreamShaper** | Fantasy, concept art | Stylized |
| **Absolute Reality** | Photorealistic | Realistic |
| **Anime Pastel Dream** | Anime/manga style | Anime |

## Prompt Engineering Tips

### Structure
```
[Subject] + [Style] + [Details] + [Quality Keywords]
```

### Example Prompts

**Product Photography:**
```
Professional product photo of a sleek smartphone,
white background, studio lighting, commercial photography,
8K, highly detailed, sharp focus
```

**Artistic Portrait:**
```
Portrait of a woman with flowing hair,
oil painting style, dramatic lighting,
rich colors, masterpiece, artstation quality
```

**Landscape:**
```
Majestic mountain landscape at sunset,
golden hour lighting, mist in valleys,
cinematic, wide angle, 4K wallpaper quality
```

## Troubleshooting

| Error | Solution |
|-------|----------|
| "Insufficient credits" | Purchase more credits at Leonardo.ai |
| "Invalid model" | Check model ID is correct |
| "Image generation failed" | Try different prompt or settings |
| "Rate limited" | Wait and retry, or reduce request frequency |

## Tips

1. **Negative Prompts** - Use to avoid common artifacts: "blurry, low quality, distorted, watermark"
2. **Guidance Scale** - 7-10 for balanced results, higher for strict prompt adherence
3. **Seeds** - Save seeds of good results for reproducibility
4. **Alchemy** - Enable for enhanced quality (uses more credits)
5. **Prompt Magic** - Automatically enhance prompts (uses more credits)
