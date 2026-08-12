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
studio-community/
├── schemas/                          # JSON schemas for validation
├── comfyui/                          # ComfyUI workflow packages
├── prompts/                          # AI agent prompt packages
├── providers/                        # Provider packages
├── workflows/                        # Workflow packages
└── docs/                             # User-facing documentation
    ├── admin.md
    ├── super-admin.md
    ├── user.md
    ├── providers/                    # Per-provider documentation
    └── workflows/                    # Per-workflow documentation
```

### Components
```
studio-community/
├── comfyui/                          # ComfyUI workflow packages
├── prompts/                          # AI agent prompt packages
├── providers/                        # Provider packages
├── workflows/                        # Workflow packages
```

## Content Types

| Type | Catalog | Content directory | Description |
|------|---------|-------------------|-------------|
| ComfyUI | `comfyui-catalog.json` | `comfyui/` | ComfyUI API-format workflows |
| Docs | `docs-catalog.json` | `docs/` | In-app user, provider, and workflow documentation |
| Prompts | `prompts-catalog.json` | `prompts/` | Reusable prompt templates |
| Providers | `providers-catalog.json` | `providers/` | Installable service bundles |
| Workflows | `workflows-catalog.json` | `workflows/` | Standalone automation recipes |

## Tiers

- **Community** - default tier. Public download, no authentication required.
- **Plus** - Download requires an entitlement token.

## License

Content in this repository is governed by the **Studio Marketplace Package License** ([LICENSE](LICENSE)). It is not open source, and it is not a free public asset dump.

You may install and use packages on Studio deployments you operate, and modify them for your own use. You may not redistribute, resell, sublicense, or transfer the packages themselves. Sharing a workflow that *references* a package is fine; sharing the package is not.

The Studio platform itself is governed by the separate [Studio Use License](https://github.com/selfhosthub/studio/blob/main/LICENSE).
