# ttpears claude-plugins

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) that bundles the MCP servers published by [@ttpears](https://github.com/ttpears) into a single installable catalog. Each plugin is a thin manifest over a published npm package — installing a plugin wires its MCP server into Claude Code via `npx`.

## Plugins

Verified against published npm releases on **2026-10-31**. Each plugin source is pinned to the commit recorded by its npm release, so an unreleased upstream manifest does not silently change the catalog.

| Plugin | Package | Manifest version | Node.js | Purpose |
| --- | --- | --- | --- | --- |
| `gitlab-mcp` | [`@ttpears/gitlab-mcp-server`](https://www.npmjs.com/package/@ttpears/gitlab-mcp-server) | `3.0.0` | 22+ | GitLab GraphQL discovery and team activity tools |
| `bookstack-mcp` | [`bookstack-mcp`](https://www.npmjs.com/package/bookstack-mcp) | `5.7.1` | 18+ | BookStack wiki search, reading, and optional writes |
| `mediawiki-mcp` | [`mediawiki-mcp`](https://www.npmjs.com/package/mediawiki-mcp) | `2.5.4` | 20+ | MediaWiki reading and editing across named wikis |

## Requirements

- Claude Code with plugin support. This catalog is validated with **2.1.302**; that is the tested version, not a minimum-version claim.
- Node.js and `npx` on the PATH visible to Claude Code. Node.js **22 or newer** meets all three packages' declared requirements.
- Git to fetch the marketplace and upstream repositories.
- A POSIX `sh` on PATH for GitLab's upstream launch wrapper. On Windows, use Git Bash or WSL and ensure `sh` and `npx` are available in that environment.
- Credentials and an endpoint for each service you install.

## Install

Add the marketplace:

```
/plugin marketplace add ttpears/claude-plugins
```

Install one or more plugins:

```
/plugin install gitlab-mcp@ttpears
/plugin install bookstack-mcp@ttpears
/plugin install mediawiki-mcp@ttpears
```

From your shell, you can instead run `claude plugin marketplace add ttpears/claude-plugins` and `claude plugin install gitlab-mcp@ttpears --scope user`. Use `user` for all projects, `local` for yourself in the current project, or `project` to share the enabled plugin with collaborators. Check `/mcp` in your session to confirm that each server starts and authenticates.

## Updates

Refresh the catalog, then update the plugins you installed:

```bash
claude plugin marketplace update ttpears
claude plugin update gitlab-mcp@ttpears
claude plugin update bookstack-mcp@ttpears
claude plugin update mediawiki-mcp@ttpears
```

Only run update commands for installed plugins, then start a new session. Inside Claude Code, the Marketplaces tab in `/plugin` also offers **Update marketplace** and an auto-update toggle. Auto-update is off by default for third-party marketplaces. See [Claude Code's update documentation](https://code.claude.com/docs/en/discover-plugins#keep-plugins-updated).

## Configuration

Each MCP server reads credentials and endpoints from environment variables inherited by its `npx` process. From a checkout of this repository, copy `.env.example` to `.env`, fill in the services you use, then load it in the shell that starts Claude Code:

```bash
cp .env.example .env
# Edit .env, then load it (Bash / Git Bash / WSL):
set -a
. ./.env
set +a
claude
```

Copying `.env` alone does not load it. The commands above explicitly export its values before launch. Local `.env` files are ignored by Git. You can also export variables directly or put them inside the `env` object in your user-level `~/.claude/settings.json`.

Check each plugin's upstream repo for the full list of required variables:

- [gitlab-mcp](https://github.com/ttpears/gitlab-mcp#configuration)
- [bookstack-mcp](https://github.com/ttpears/bookstack-mcp#configuration)
- [mediawiki-mcp](https://github.com/ttpears/mediawiki-mcp#configuration)

## Manual install with `claude mcp add`

Choose direct registration instead of installing the marketplace plugin for the same service to avoid duplicate servers. The examples below pin the npm releases verified above. Repeat `--env` for each variable and put all Claude options before `--`, which separates the spawned command. `--scope` picks where the entry is written: `local` (default, current project), `user` (all projects), or `project` (`.mcp.json` for team use). These are Bash examples; enter each complete command on one line in PowerShell.

If you already export credentials before every Claude Code session, you can omit the `--env` flags and let the server inherit them. Otherwise, replace the placeholders below; `--env` stores their values in the selected MCP registration. See the [official MCP configuration reference](https://code.claude.com/docs/en/mcp).

### gitlab-mcp

Set `GITLAB_URL` and **exactly one** of `GITLAB_TOKEN` or `GITLAB_READ_TOKEN`. Both token variables together cause a startup error. Use a `read_api` PAT for read-only access or an `api` PAT when writes are needed. Optional settings include `GITLAB_TIMEOUT` and `GITLAB_MAX_PAGE_SIZE`.

`GITLAB_AUTH_MODE` and `GITLAB_SHARED_ACCESS_TOKEN` were removed in 1.14.0 and are ignored with a warning. Remove old settings when migrating; use `GITLAB_READ_TOKEN` for shared read-only access or `GITLAB_TOKEN` for full-access fallback credentials.

```bash
claude mcp add gitlab \
  --scope user \
  --env GITLAB_URL=https://gitlab.com \
  --env GITLAB_TOKEN=glpat-your-pat \
  -- npx -y @ttpears/gitlab-mcp-server@3.0.0
```

### bookstack-mcp

Required: `BOOKSTACK_BASE_URL`, `BOOKSTACK_TOKEN_ID`, `BOOKSTACK_TOKEN_SECRET` (from a BookStack API token). Optional: `BOOKSTACK_ENABLE_WRITE=true` to allow create/update/delete tools, `BOOKSTACK_INSECURE_SKIP_TLS_VERIFY=true` for self-signed certs.

```bash
claude mcp add bookstack \
  --scope user \
  --env BOOKSTACK_BASE_URL=https://your-bookstack.com \
  --env BOOKSTACK_TOKEN_ID=your-token-id \
  --env BOOKSTACK_TOKEN_SECRET=your-token-secret \
  --env BOOKSTACK_ENABLE_WRITE=false \
  -- npx -y bookstack-mcp@5.7.1
```

### mediawiki-mcp

Single-wiki — required: `MEDIAWIKI_BASE_URL`, `MEDIAWIKI_USERNAME`, `MEDIAWIKI_PASSWORD` (from `Special:BotPasswords`).

```bash
claude mcp add mediawiki \
  --scope user \
  --env MEDIAWIKI_BASE_URL=https://wiki.example.com \
  --env MEDIAWIKI_USERNAME=Admin@mcp \
  --env MEDIAWIKI_PASSWORD=your-bot-password \
  -- npx -y mediawiki-mcp@2.5.4
```

Multi-wiki — register wikis as `Name:URL` pairs in `MEDIAWIKI_WIKIS`, then provide `MEDIAWIKI_USERNAME_<NAME>` and `MEDIAWIKI_PASSWORD_<NAME>` per wiki (uppercase). `MEDIAWIKI_DEFAULT_WIKI` picks the wiki used when a tool call doesn't specify one.

```bash
claude mcp add mediawiki \
  --scope user \
  --env MEDIAWIKI_WIKIS=Main:https://wiki.example.com,Dev:https://dev.wiki.example.com \
  --env MEDIAWIKI_DEFAULT_WIKI=Main \
  --env MEDIAWIKI_USERNAME_MAIN=Admin@mcp \
  --env MEDIAWIKI_PASSWORD_MAIN=your-bot-password \
  --env MEDIAWIKI_USERNAME_DEV=Admin@mcp \
  --env MEDIAWIKI_PASSWORD_DEV=your-bot-password \
  -- npx -y mediawiki-mcp@2.5.4
```

## How it's wired

The catalog is `.claude-plugin/marketplace.json`. Each entry points at an upstream GitHub repository through a `github` source and a full release commit `sha`. Claude Code reads that commit's `.claude-plugin/plugin.json`, which declares the MCP server's launch command. Display names, repository links, licenses, and keywords are supplied here so users can see them before installation.

- The source pin fixes the plugin manifest and its configuration at a reviewed commit.
- The upstream launch commands currently use **unversioned npm package names**. A manifest pin does not pin the runtime server package; `npx` handles package resolution and caching. Do not assume every spawn downloads a new release.
- For an exact runtime package version, use the versioned direct registrations above.
- Updating the catalog and updating an installed plugin are separate operations, as shown in [Updates](#updates).

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Plugin not found | Add the marketplace first and use the full `plugin@ttpears` ID. |
| `npx` or `sh` not found | Check PATH in the same shell that starts Claude Code. GitLab's manifest needs both. |
| GitLab refuses to start | Set exactly one token variable and remove obsolete authentication settings. |
| BookStack reads work but writes fail | Enable `BOOKSTACK_ENABLE_WRITE` and check the API token's BookStack permissions. |
| MediaWiki login fails | Supply both bot credential parts, including a username such as `Admin@mcp`. |
| Old plugin behavior after updating | Update the installed plugin as well as the catalog, then start a new session. |

## Refreshing the catalog

1. Check the published release with `npm view <package> version gitHead engines --json`.
2. At its `gitHead`, inspect `.claude-plugin/plugin.json` and `package.json`. Confirm matching plugin names and versions, the launch command, and environment requirements. A newer default-branch version may not yet be published.
3. Set `source.sha` to that full commit and refresh this README's version table, direct install examples, and credential guidance. Keep plugin version authority in upstream `plugin.json` rather than duplicating it in the catalog entry.
4. Run `claude plugin validate . --strict` and `git diff --check`. Also validate the upstream checkout with `claude plugin validate /path/to/plugin --strict`.
5. Test installation and authenticated access before releasing. CI validates the catalog; it does not authenticate to services or exercise their tools.

## Adding a new plugin

1. Publish the npm package and an upstream `.claude-plugin/plugin.json` with a matching name, version, and `mcpServers` entry.
2. Add the GitHub source and reviewed release commit to `.claude-plugin/marketplace.json`. Update this README and `.env.example`.
3. Run the checks above and open a pull request. Users refresh the catalog and installed plugins using [Updates](#updates).

See the [marketplace reference](https://code.claude.com/docs/en/plugins/marketplace-reference) for source and metadata fields.
