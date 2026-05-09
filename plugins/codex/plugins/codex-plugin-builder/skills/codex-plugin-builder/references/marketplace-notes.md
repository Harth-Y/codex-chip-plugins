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

## Marketplace Cache And CLI Recovery

If GitHub contains a new plugin but the Codex UI still shows the old list, check
the local marketplace cache before changing repository files. A valid cache can
exist while the UI still needs a reload.

Check configured marketplaces:

```powershell
$lines = Get-Content "$env:USERPROFILE\.codex\config.toml"
$lines | Select-String -Pattern "\[marketplaces\.|last_updated|last_revision|source|ref"
```

Important fields:

```text
[marketplaces.<marketplace-name>]
last_updated = ...
last_revision = ...
source = ...
ref = ...
```

Check the installed marketplace root:

```powershell
$root = "$env:USERPROFILE\.codex\.tmp\marketplaces\<marketplace-name>"
Get-Content -Raw "$root\.agents\plugins\marketplace.json"
Get-ChildItem -Name "$root\plugins\codex\plugins"
```

If the cache has the expected plugin but the UI does not, reload VS Code first:

```text
Developer: Reload Window
```

If reload is not enough, restart VS Code.

Use Codex CLI when the UI has no remove or refresh control:

```powershell
codex plugin marketplace --help
codex plugin marketplace remove <marketplace-name>
codex plugin marketplace add https://github.com/<owner>/<repo>.git --ref main
```

For this style of public GitHub marketplace, re-add without `--sparse` when the
root `.agents/plugins/marketplace.json` exists.

`upgrade` may be available:

```powershell
codex plugin marketplace upgrade <marketplace-name>
```

If `upgrade` hangs, stop the orphaned `codex.exe` process that was launched for
the upgrade, then use `remove` plus `add`. Do not terminate unrelated active
Codex sessions unless the user explicitly approves it.

After re-adding, verify:

```powershell
Get-Content -Raw "$env:USERPROFILE\.codex\.tmp\marketplaces\<marketplace-name>\.agents\plugins\marketplace.json"
Get-ChildItem -Name "$env:USERPROFILE\.codex\.tmp\marketplaces\<marketplace-name>\plugins\codex\plugins"
```

If a marketplace appears twice in the dropdown with the same display name, it
may be one GitHub marketplace plus one local workspace marketplace. A workspace
containing `.agents/plugins/marketplace.json` can be shown as a local marketplace
using the folder name as its id. Treat the GitHub id as the published source and
the folder id as the local development copy.

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
