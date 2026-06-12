# Updating Plugins from the AI Catalog

This guide shows how to install the `ai-catalog` CLI tool and use it to update all platform-specific plugin manifests from the canonical [`ai-catalog.json`](./ai-catalog.json).

## Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download) or later

## 1. Install the AI Catalog CLI

```powershell
dotnet tool install --global SpecWorks.AiCatalog.Cli
```

To update an existing installation:

```powershell
dotnet tool update --global SpecWorks.AiCatalog.Cli
```


## 3. Export platform-specific plugin manifests

The `export` command regenerates all platform-specific marketplace files from the AI Catalog. This updates manifests for GitHub Copilot CLI, OpenAI Codex CLI, and Claude Code.

Export all platforms at once:

```powershell
ai-catalog export ai-catalog.json -o . --source-dir .
```


This regenerates the following manifests from `ai-catalog.json`:

| Platform           | Generated files                                                                 |
| ------------------ | ------------------------------------------------------------------------------- |
| GitHub Copilot CLI | `.github/plugin/marketplace.json`, `plugins/*/.github/plugin/marketplace.json`  |
| OpenAI Codex CLI   | `plugins/*/.codex-plugin/plugin.json`                                           |
| Claude Code        | `.claude-plugin/marketplace.json`                                               |

## 4. Typical workflow

When you add or modify skills, MCP servers, or plugin metadata:

1. **Edit `ai-catalog.json`** — Update entries, versions, descriptions, or add new artifacts.

2. **Re-export all manifests:**
   ```powershell
   ai-catalog export ai-catalog.json -o . --source-dir .
   ```

3. **Review the changes:**
   ```powershell
   git diff
   ```

4. **Reinstall plugins** in your Copilot CLI session to pick up changes:
   ```powershell
   copilot plugin uninstall workiq; copilot plugin install ./plugins/workiq
   copilot plugin uninstall workiq-preview; copilot plugin install ./plugins/workiq-preview
   copilot plugin uninstall microsoft-365-agents-toolkit; copilot plugin install ./plugins/microsoft-365-agents-toolkit
   copilot plugin uninstall workiq-productivity; copilot plugin install ./plugins/workiq-productivity
   ```

5. **Commit and push:**
   ```powershell
   git add -A
   git commit -m "chore: update plugin manifests from ai-catalog.json"
   git push
   ```

## Reference

- [AI Catalog specification](https://agent-card.github.io/ai-card/)
- [AI Catalog CLI documentation](https://spec-works.github.io/ai-catalog/)
- [AI Catalog GitHub repo](https://github.com/Agent-Card/ai-catalog)
