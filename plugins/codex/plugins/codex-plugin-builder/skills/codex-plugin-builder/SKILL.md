---
name: codex-plugin-builder
description: Create, update, validate, publish, and troubleshoot Codex skills, Codex plugins, and GitHub-hosted plugin marketplaces. Use when Codex needs to turn a skill folder into a plugin, add plugins to marketplace.json, publish a marketplace repository, debug "add plugin marketplace failed" errors, explain sparse path versus root marketplace behavior, or document reusable skill update workflows.
---

# Codex Plugin Builder

## Purpose

Build Codex skills and plugins as reusable marketplace artifacts, not as one-off
project notes. Preserve the distinction between:

- `skill`: reusable instructions and references for Codex.
- `plugin`: package that can expose skills, apps, MCP config, assets, or hooks.
- `marketplace`: catalog that tells Codex where plugins live and how they can be installed.

## Default Workflow

1. Inspect the current repository and identify the intended marketplace root.
2. Use `skill-creator` when creating or revising a skill.
3. Use `plugin-creator` when creating plugin scaffolding or marketplace entries.
4. Keep source material separate from published plugin content unless the user asks to publish it.
5. Validate JSON and skill metadata before committing.
6. Push to GitHub only after confirming the staged scope excludes local-only source material.
7. Tell the user the exact values to enter in Codex's "Add plugin marketplace" UI.

## Recommended Repository Layout

Prefer a root marketplace manifest for maximum Codex compatibility:

```text
.agents/
  plugins/
    marketplace.json
plugins/
  codex/
    plugins/
      plugin-name/
        .codex-plugin/
          plugin.json
        skills/
          skill-name/
            SKILL.md
            references/
```

The root `.agents/plugins/marketplace.json` should point to plugin paths relative
to the Git repository root, for example:

```json
{
  "name": "my-marketplace",
  "interface": {
    "displayName": "My Marketplace"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "source": {
        "source": "local",
        "path": "./plugins/codex/plugins/plugin-name"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Coding"
    }
  ]
}
```

If the user explicitly needs a sparse marketplace subdirectory, it is acceptable
to also keep `plugins/codex/.agents/plugins/marketplace.json`, but do not rely
on sparse path support unless it has been verified in that Codex client.

## Plugin Manifest Rules

For each plugin, keep:

```text
plugins/codex/plugins/<plugin-name>/.codex-plugin/plugin.json
```

Use these practical defaults:

- `name`: exactly matches the plugin directory name.
- `version`: semantic version such as `0.1.0`.
- `description`: concise plugin purpose.
- `repository`: GitHub repository URL.
- `skills`: `./skills/` when the plugin exposes skills.
- `interface.displayName`: human-readable plugin name.
- `interface.category`: `Coding` for developer workflow plugins.

Do not leave scaffold `[TODO: ...]` placeholders in published manifests.
Remove optional `hooks`, `mcpServers`, `apps`, icon, logo, or screenshot fields
when those files do not exist.

## Skill Content Rules

Use the skill as reusable operating knowledge:

- Keep `SKILL.md` under about 500 lines when possible.
- Put detailed notes, examples, and troubleshooting in `references/`.
- Avoid project-specific paths, temporary filenames, private business logic, or one-off debugging history.
- Make frontmatter `description` explicit about when the skill should trigger.
- Use imperative instructions and concrete validation commands.

When a user discovers a reusable improvement while working in another project,
ask them for a distilled update:

```text
Summarize what should be added to the skill. Separate SKILL.md workflow rules
from references knowledge. Do not include project-specific paths or temporary
implementation details.
```

## Validation

Validate marketplace and plugin JSON:

```powershell
Get-Content -Raw .agents/plugins/marketplace.json | ConvertFrom-Json
Get-Content -Raw plugins/codex/plugins/<plugin-name>/.codex-plugin/plugin.json | ConvertFrom-Json
```

Validate a skill folder with `skill-creator`:

```powershell
python C:/Users/61447/.codex/skills/.system/skill-creator/scripts/quick_validate.py plugins/codex/plugins/<plugin-name>/skills/<skill-name>
```

Confirm required files are not ignored:

```powershell
git check-ignore -v .agents/plugins/marketplace.json plugins/codex/plugins/<plugin-name>/.codex-plugin/plugin.json plugins/codex/plugins/<plugin-name>/skills/<skill-name>/SKILL.md
```

## Publishing

Before commit:

- Run `git status -sb`.
- Stage explicit published paths.
- Exclude local-only source folders such as `source-skills/` unless the user explicitly wants them published.

Typical publish commands:

```powershell
git add .agents plugins README.md .gitignore
git commit -m "Add <plugin-name> Codex plugin"
git push
```

## Loading In Codex

For GitHub-hosted marketplaces, prefer:

```text
Source: https://github.com/<owner>/<repo>.git
Git reference: main
Sparse path: leave blank
```

Read `references/marketplace-notes.md` before debugging marketplace import
failures or deciding whether to use a sparse path.
