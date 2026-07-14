# Studio Catalog

Catalog of released content for Self-Host Studio.

## What This Repo Contains

- **Catalogs** - JSON indexes for every content type
- **Content** - provider packages, workflows, ComfyUI workflows, prompts
- **Schemas** - JSON schemas for validating catalog and content files
- **Docs** - user-facing documentation served in-app

## File Structure

### Catalogs and docs
```
studio-cat/
├── schemas/                          # JSON schemas for validation
├── comfyui/                          # ComfyUI workflow packages
├── prompts/                          # AI agent prompt packages
├── providers/                        # Provider packages
├── workflows/                        # Workflow packages
└── docs/                             # User-facing documentation
    ├── admin.md
    ├── super-admin.md
    ├── user.md
    └── providers/                    # Per-provider documentation
```

### Components
```
studio-cat/
├── comfyui/                          # ComfyUI workflow packages
├── prompts/                          # AI agent prompt packages
├── providers/                        # Provider packages
├── workflows/                        # Workflow packages
```

## Content Types

| Type | Catalog | Content directory | Description |
|------|---------|-------------------|-------------|
| ComfyUI | `comfyui-catalog.json` | `comfyui/` | ComfyUI API-format workflows |
| Docs | `docs-catalog.json` | `docs/` | In-app user and provider documentation |
| Prompts | `prompts-catalog.json` | `prompts/` | Reusable prompt templates |
| Providers | `providers-catalog.json` | `providers/` | Installable service bundles |
| Workflows | `workflows-catalog.json` | `workflows/` | Standalone automation recipes |

## Tiers

- **Community** - default tier. Public download, no authentication required.
- **Plus** - Download requires an entitlement token.
