# GitLab MCP

[Home](../../README.md) · [Credential loading](../configuration.md) · [Troubleshooting](../troubleshooting.md)

- **Install ID:** `gitlab-mcp@ttpears`
- **Published package:** `@ttpears/gitlab-mcp-server`
- **Catalog manifest:** `3.0.0`

Use this plugin to discover GitLab projects, issues, merge requests, and team
activity. Requests use the identity and permissions of the supplied token.

## Choose your token mode

| Goal | Variable | PAT scope |
| --- | --- | --- |
| Read-only fallback access | `GITLAB_READ_TOKEN` | `read_api` |
| Fallback access including writes | `GITLAB_TOKEN` | `api` |

Set **one** fallback variable for the local setup below. Setting both nonempty
variables is a startup error. Token scopes do not grant access to projects that
the token's account cannot access.

The server also supports per-call credentials without a fallback token; that is
an advanced upstream configuration, not the setup used here.

## Configure and install

Read-only example in Bash, Git Bash, or WSL:

```bash
unset GITLAB_TOKEN
export GITLAB_URL='https://gitlab.com'
export GITLAB_READ_TOKEN='replace-with-read-api-token'
claude plugin marketplace add ttpears/claude-plugins
claude plugin install gitlab-mcp@ttpears --scope user
claude
```

PowerShell environment equivalent:

```powershell
Remove-Item Env:GITLAB_TOKEN -ErrorAction SilentlyContinue
$env:GITLAB_URL = 'https://gitlab.com'
$env:GITLAB_READ_TOKEN = 'replace-with-read-api-token'
```

Then run the same `claude plugin` and `claude` commands. If you also set tokens in
personal settings or a direct registration, remove the conflicting value there
too; clearing a shell variable does not edit those files.

For writes, remove `GITLAB_READ_TOKEN` and set `GITLAB_TOKEN` to an `api`-scoped
PAT instead. A fresh session must receive the new configuration.

## Verify with a read

Open `/mcp`, check the GitLab server, then ask:

> Find the GitLab project named my-project and list its open issues. Do not create, update, or close anything.

Replace `my-project` with a project visible to your token. A successful query
confirms that token access works for that project. Some advanced generic query
tools require per-call credentials even when a fallback token is set; consult
the upstream configuration rather than assuming every tool uses the same path.

## Runtime and platform

The package declares Node.js **22+**. The
[pinned manifest](https://github.com/ttpears/gitlab-mcp/blob/953fc42617dfafc58a0132a0460a1b7a076cbd66/.claude-plugin/plugin.json)
runs `sh -c`, changes to the shell's home directory, and executes
`npx -y @ttpears/gitlab-mcp-server`. Both `sh` and `npx` must resolve in the
environment that launches the plugin.

On Windows, Git Bash or WSL provides a POSIX shell. Check `command -v sh` and
`command -v npx` there. In native PowerShell, use `Get-Command sh, npx` to check
what the host process can resolve. A [direct npm registration](../direct-install.md)
does not use this plugin wrapper and can pin the server package itself.

## Useful optional settings

| Variable | Default in the pinned source | Purpose |
| --- | --- | --- |
| `GITLAB_URL` | `https://gitlab.com` | Target instance; use your own instance's base URL when self-hosted. |
| `GITLAB_TIMEOUT` | `30000` ms | Request timeout. |
| `GITLAB_MAX_PAGE_SIZE` | `50` | Page-size cap, from 1 to 100. |
| `GITLAB_MAX_CONCURRENCY` | `16` | Concurrent request limit; excess work queues. |
| `GITLAB_PIN_HOST` | `true` | Keep requests on the configured instance. |

## Migrate older settings

`GITLAB_AUTH_MODE` and `GITLAB_SHARED_ACCESS_TOKEN` were removed in 1.14.0. They
produce warnings and are ignored. Replace the old shared token variable with
the appropriate current fallback variable, remove the obsolete settings, and
restart Claude Code. Do not retain both current token variables to split traffic.

Reference: [configuration at the pinned release](https://github.com/ttpears/gitlab-mcp/blob/953fc42617dfafc58a0132a0460a1b7a076cbd66/README.md#configuration).
HTTP, OAuth, and shared deployments have additional requirements documented
upstream; adding this marketplace plugin does not deploy those services.
