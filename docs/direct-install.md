# Direct MCP installation

[Home](../README.md) · [Configuration](configuration.md) · [Operations](operations.md)

Use a direct registration when you want to choose an npm package version,
per-server environment settings, or a team `.mcp.json`. A direct server is managed with
`claude mcp`; a marketplace plugin is managed with `claude plugin`.

## Choose one route for each connection

| Route | Configuration owner | Version choice |
| --- | --- | --- |
| Marketplace plugin | The upstream plugin manifest | Catalog pins the manifest; BookStack also pins its npm runtime to 5.7.1. |
| Direct MCP registration | You, through `claude mcp add` or `.mcp.json` | You choose the package version in the command. |

Before switching an existing connection, [disable its plugin](operations.md#disable-or-remove-a-plugin)
or remove its direct registration at the scope you used. This avoids two servers
exposing the same service under different names.

## Register an exact package version

Export credentials using the [configuration guide](configuration.md). Then run
the appropriate command. Each is one line and works in Bash or PowerShell:

```text
claude mcp add --scope user --transport stdio gitlab -- npx -y @ttpears/gitlab-mcp-server@3.0.0
claude mcp add --scope user --transport stdio bookstack -- npx -y bookstack-mcp@5.7.1
claude mcp add --scope user --transport stdio mediawiki -- npx -y mediawiki-mcp@2.5.4
```

These registrations inherit credentials each time Claude Code starts. Export them
again for later sessions or use personal settings. The versions match this
catalog's documented snapshot; change an exact package version deliberately when
upgrading.

All Claude options belong before `--`. Everything after it is the server command
and its arguments. The direct command invokes `npx` itself; it does not use the
GitLab plugin's separate `sh` wrapper.

## Store credentials on one registration

For example, BookStack in Bash. The server name goes first here because `--env`
accepts a list of values and can otherwise consume a following bare name:

```bash
claude mcp add bookstack --scope user --transport stdio \
  --env BOOKSTACK_BASE_URL=https://bookstack.example.com \
  --env BOOKSTACK_TOKEN_ID=replace-with-token-id \
  --env BOOKSTACK_TOKEN_SECRET=replace-with-token-secret \
  --env BOOKSTACK_ENABLE_WRITE=false \
  -- npx -y bookstack-mcp@5.7.1
```

`--env` writes the supplied values to the registration, and literal credentials
may also remain in your shell history. For a shared project, use variable
references as shown below instead of stored credentials.

## Share a project configuration

[mcp-project.example.json](../examples/mcp-project.example.json) contains one
BookStack registration with references to environment variables. Copy it to the
root of your own project as `.mcp.json`, merging with existing servers if needed:

```json
{
  "mcpServers": {
    "bookstack": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "bookstack-mcp@5.7.1"],
      "env": {
        "BOOKSTACK_BASE_URL": "${BOOKSTACK_BASE_URL}",
        "BOOKSTACK_TOKEN_ID": "${BOOKSTACK_TOKEN_ID}",
        "BOOKSTACK_TOKEN_SECRET": "${BOOKSTACK_TOKEN_SECRET}",
        "BOOKSTACK_ENABLE_WRITE": "${BOOKSTACK_ENABLE_WRITE:-false}"
      }
    }
  }
}
```

Each collaborator exports their own values before launching Claude Code from the
project. Claude Code expands these references; they are not actual credentials
inside the committed file. Missing required variables still need to be supplied.
Project MCP servers may require approval before they connect. See the
[official MCP scope and configuration guide](https://code.claude.com/docs/en/mcp).

## Verify or remove the direct server

```text
claude mcp get bookstack
claude mcp remove bookstack --scope user
```

Use the scope you registered: `local` is the direct-command default, `user`
applies across your projects, and `project` uses `.mcp.json`. Inspect `/mcp` in a
fresh session after changing a registration. Removing a direct registration does
not uninstall a separately installed marketplace plugin.
