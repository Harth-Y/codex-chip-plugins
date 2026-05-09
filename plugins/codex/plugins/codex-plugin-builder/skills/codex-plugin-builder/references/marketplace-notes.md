# Marketplace Notes

## Import UI Values

Use these values for a public GitHub marketplace repository:

```text
Source: https://github.com/<owner>/<repo>.git
Git reference: main
Sparse path: leave blank
```

`owner/repo` may work in some clients, but the full HTTPS Git URL is clearer
and matches the UI prompt style.

## Root Manifest Requirement

Codex marketplace import expects a supported manifest at the marketplace root:

```text
.agents/plugins/marketplace.json
```

In observed VS Code Codex behavior, entering a sparse path such as
`plugins/codex` filtered the checkout but did not make that subdirectory the
marketplace root. The import failed with:

```text
marketplace root does not contain a supported manifest
```

The compatibility fix is to add a root manifest and leave sparse path blank.
If keeping a nested marketplace layout too, maintain both manifests:

```text
.agents/plugins/marketplace.json
plugins/codex/.agents/plugins/marketplace.json
```

The root manifest must use paths relative to the Git repository root, for
example:

```json
"path": "./plugins/codex/plugins/om6626"
```

The nested manifest must use paths relative to `plugins/codex`, for example:

```json
"path": "./plugins/om6626"
```

## Validation Commands

Check raw GitHub availability:

```powershell
$u = "https://raw.githubusercontent.com/<owner>/<repo>/main/.agents/plugins/marketplace.json"
(Invoke-WebRequest -UseBasicParsing $u).StatusCode
```

Check Git ref:

```powershell
git ls-remote https://github.com/<owner>/<repo>.git refs/heads/main
```

Check sparse checkout behavior when needed:

```powershell
$tmp = Join-Path $env:TEMP ("marketplace-test-" + [guid]::NewGuid())
git clone --depth 1 --filter=blob:none --sparse https://github.com/<owner>/<repo>.git $tmp
Push-Location $tmp
git sparse-checkout set plugins/codex
Get-ChildItem -Recurse -Force plugins/codex
Pop-Location
Remove-Item -Recurse -Force $tmp
```

## VS Code Codex Logs

When the UI only says "add plugin marketplace failed", inspect logs:

```powershell
Get-ChildItem -Recurse -Force "$env:APPDATA\Code\logs" -Filter Codex.log |
  Sort-Object LastWriteTime -Descending |
  Select-Object -First 10
```

Search for:

```text
marketplace/add
marketplace root
supported manifest
plugin.json
marketplace.json
```

Temporary staging clones may be under:

```text
C:\Users\<user>\.codex\.tmp\marketplaces\.staging\
```

Inspect those directories to verify what Codex actually cloned.

## Updating Skills After Installation

If a project is mid-task and a reusable skill improvement is discovered:

1. Ask Codex in that project to summarize the general improvement.
2. Separate `SKILL.md` workflow changes from `references/` knowledge changes.
3. Avoid project-specific paths, temporary code, or private assumptions.
4. Update the marketplace repository source skill.
5. Push the marketplace update.
6. Refresh or reinstall the plugin at a task boundary, preferably in a new Codex session.

Deleting and reinstalling a plugin in the middle of a long conversation may not
change the already-loaded skill context for that conversation.
