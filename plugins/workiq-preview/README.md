# Work IQ Plugin

Full WorkIQ tool surface for GitHub Copilot CLI: agentic semantic queries via `ask` **plus** direct, structured reads and writes against Microsoft 365 — emails, meetings, calendar, documents, Teams messages, OneDrive/SharePoint files, and people.

## Installation

### Via GitHub Copilot CLI Plugin Marketplace

```bash
/plugin install workiq-preview@work-iq
```

### Via MCP Configuration

Add to your `.mcp.json` or IDE MCP settings:

```json
{
  "mcpServers": {
    "workiq-preview": {
      "url": "https://workiq.svc.cloud.microsoft/mcp"
    }
  }
}
```

The plugin connects to the hosted WorkIQ MCP prod endpoint. It does **not** launch a local `npx` MCP server for tool calls.

## Updating

The MCP tool surface is served by the hosted WorkIQ endpoint above, so updating a local npm package is not required for MCP tool calls.

Some setup and diagnostics operations are CLI-only (`auth login`/`logout`, `auth consent`, `config show`/`set`/`reset`, `version`). When you need those, run the published CLI via `npx`:

```bash
npx -y @microsoft/workiq@latest <command>
```

To verify the CLI version:

```bash
npx -y @microsoft/workiq@latest version
```

## Usage

The plugin exposes the WorkIQ MCP tool surface — read **and** write — from `https://workiq.svc.cloud.microsoft/mcp`.

### Semantic queries (`ask`)

```
"What did John say about the proposal?"
"Summarize emails from the leadership team this week"
"What's top of mind for Sarah?"
"Find the design doc for the authentication system"
"Who is working on Project Alpha?"
```

### Structured reads (`fetch`, `search_paths`, `get_schema`, `fetch_blob`)

```
"List my unread emails from Sarah this week"
"What meetings do I have Monday?"
"Show me the channels in the DevX team"
"List files in my OneDrive 'Specs' folder"
"Who are Rob's direct reports?"
```

### Writes (`create_entity`, `update_entity`, `delete_entity`, `do_action`, `upload_blob`)

> ⚠️ Writes execute immediately and are visible to other people or unrecoverable. The skill is instructed to confirm with you before sending mail, forwarding, accepting/declining meetings, or permanently deleting.

```
"Send the draft email to the engineering distribution list"
"Create a calendar event Friday at 3pm with the design team"
"Accept the 2pm meeting from Rob"
"Decline the Monday standup — I'll catch up on the recording"
"Mark Sarah's last three emails as read"
"Reply to the deadline thread with 'on track for Friday'"
"Move the design review thread to the Archive folder"
```

> ⚠️ `fetch_blob` and `upload_blob` are documented for future reference but are not released in the current WorkIQ MCP surface. For downloads, fetch metadata and return `webUrl`; for uploads, direct the user to OneDrive / SharePoint until raw byte support is released.

### CLI commands (out-of-band of the MCP server)

Some operations are not exposed as MCP tools and must be run as shell commands — `auth login`/`logout`, `auth consent` (granting additional Graph scopes), `config show`/`set`/`reset`, `version`. Invoke them via `npx -y @microsoft/workiq@latest <command>` when needed. See [`skills/workiq-preview/references/cli-commands.md`](./skills/workiq-preview/references/cli-commands.md) for the full reference.

## Skills

| Skill | Description |
|-------|-------------|
| [**workiq-preview**](./skills/workiq-preview/SKILL.md) | Guides usage of the full WorkIQ tool surface — `ask` for semantic questions plus entity tools for fast, structured M365 reads and writes |

## Platform Support

Supported on `win_x64`, `win_arm64`, `linux_x64`, `linux_arm64`, `osx_x64`, and `osx_arm64`.

## License

See the root [LICENSE](../../LICENSE) file.
