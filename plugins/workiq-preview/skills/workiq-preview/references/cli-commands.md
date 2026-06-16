# WorkIQ CLI commands (out-of-band of the MCP server)

Some WorkIQ operations are **not exposed as MCP tools** and must be run as shell commands — most commonly authentication management, scope consent, config inspection, and version checks. Run these in a terminal (not via the MCP server) when you need them.

MCP tool calls use the hosted WorkIQ prod endpoint from `.mcp.json` (`https://workiq.svc.cloud.microsoft/mcp`). The CLI is only for out-of-band setup and diagnostics.

The hosted endpoint requires a user token. The MCP host should attach that token automatically; use the CLI only when the user needs to sign in, refresh stale credentials, grant additional Graph scopes, or inspect local WorkIQ configuration. Never put tokens in prompts, `.mcp.json`, or tool arguments.

## Invocation

**When a CLI command is needed, invoke via `npx -y @microsoft/workiq@latest <command>`.** This uses the published WorkIQ CLI and avoids mismatches with any globally-installed `workiq` on `PATH`. Auth and config changes may require restarting your MCP host before retrying the original request.

## Command reference

| Task | Command |
|------|---------|
| Check installed version | `npx -y @microsoft/workiq@latest version` |
| Sign in (default account) | `npx -y @microsoft/workiq@latest auth login` |
| Sign in with a specific account | `npx -y @microsoft/workiq@latest auth login --account user@contoso.com` |
| Sign out / clear cached tokens | `npx -y @microsoft/workiq@latest auth logout` |
| Grant additional Graph scopes | `npx -y @microsoft/workiq@latest auth consent --scopes Mail.Read Calendars.Read` |
| Show current config | `npx -y @microsoft/workiq@latest config show` |
| Set a config value | `npx -y @microsoft/workiq@latest config set <key>=<value>` |
| Remove a config key | `npx -y @microsoft/workiq@latest config unset <key>` |
| Reset config to defaults | `npx -y @microsoft/workiq@latest config reset` |
| Accept the EULA | `npx -y @microsoft/workiq@latest accept-eula` |
| Generate a debug share link | `npx -y @microsoft/workiq@latest debug <conversationId>` |

## When to use these

- A tool call returns **HTTP 403 Forbidden** → run `auth consent --scopes <scopes>` with the scopes needed for that Graph path.
- A tool call returns **not signed in**, **unauthorized**, or **stale token** → if no account is known, ask the user which Microsoft 365 account WorkIQ should use. Then run `auth login --account <user@contoso.com>` (or plain `auth login` if the user has no preference) and restart the MCP host / Copilot CLI session.
- Sign-in is broken or hangs on Windows → `config set disableBrokeredAuth=true` to force browser-based login.
- You suspect a **stale token** or wrong cached account → `auth logout` then `auth login` (optionally with `--account`).
- You want to verify the running binary version matches the plugin → `version`.

After any `auth` or `config` change, restart your Copilot CLI session so the MCP server picks up the new state.
