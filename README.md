# Studio Community

Public content catalog and community resources for Self-Host Studio.

## What This Repo Contains

- **Catalogs** — single source of truth for all released content (community + plus)
- **Community content** — provider packages, workflows, blueprints, prompts, ComfyUI workflows
- **Schemas** — JSON schemas for validating catalog and content files
- **Docs** — user-facing documentation served in-app

## File Structure

```
studio-community/
├── schemas/                          # JSON schemas for validation
├── blueprints-catalog.json           # Blueprints catalog
├── comfyui-catalog.json              # ComfyUI Workflows catalog
├── docs-catalog.json                 # Documentation catalog
├── prompts-catalog.json              # AI Agent prompts catalog
├── providers-catalog.json            # Providers and Service definition catalog
├── workflows-catalog.json            # Workflows catalog
├── blueprints/                       # Blueprint packages
├── comfyui/                          # ComfyUI workflow packages
├── prompts/                          # AI Agent prompt packages
├── providers/                        # Provider packages
├── workflows/                        # Workflow packages
└── docs/                             # User facing documentation
    ├── admin.md
    ├── super-admin.md
    ├── user.md
    └── providers/                    # Per-provider documentation
```

## Content Types

| Type | Catalog | Content directory | Description |
|------|---------|-------------------|-------------|
| Blueprints | `blueprints-catalog.json` | `blueprints/` | Workflow templates |
| ComfyUI | `comfyui-catalog.json` | `comfyui/` | ComfyUI API-format workflows |
| Docs | `docs-catalog.json` | `docs/` | In-app user and provider documentation |
| Prompts | `prompts-catalog.json` | `prompts/` | Reusable prompt templates |
| Providers | `providers-catalog.json` | `providers/` | Installable service bundles |
| Workflows | `workflows-catalog.json` | `workflows/` | Standalone automation recipes |

## Tiers

Catalogs list **all** released content across both tiers:

- **Community** — content files live in this repo. Catalog entries have a `download_url` pointing here.
- **Plus** — content files live in the member [studio-plus](https://github.com/selfhosthub/studio-plus) repo. Requires entitlement token to download.

## Access

- **Community content**: public, no authentication
- **Plus content**: listed in catalogs (visible to all), download requires entitlement token
