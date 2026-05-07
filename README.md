# ttpears claude-plugins

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) that bundles the MCP servers published by [@ttpears](https://github.com/ttpears) into a single installable catalog. Each plugin is a thin manifest over a published npm package — installing a plugin wires its MCP server into Claude Code via `npx`.

## Plugins

| Plugin | Package | Purpose |
| --- | --- | --- |
| `gitlab-mcp` | [`@ttpears/gitlab-mcp-server`](https://www.npmjs.com/package/@ttpears/gitlab-mcp-server) | GitLab GraphQL client with team activity tools |
| `bookstack-mcp` | [`bookstack-mcp`](https://www.npmjs.com/package/bookstack-mcp) | BookStack wiki search, read, write, and admin |
| `mediawiki-mcp` | [`mediawiki-mcp`](https://www.npmjs.com/package/mediawiki-mcp) | MediaWiki multi-wiki client over the REST API |

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

Update the catalog later with `/plugin marketplace update ttpears`.

## Configuration

Each MCP server reads its credentials and endpoints from environment variables. Set these in your shell (or `~/.claude/settings.json`) before starting Claude Code — the spawned `npx` process inherits them.

Check each plugin's upstream repo for the full list of required variables:

- [gitlab-mcp](https://github.com/ttpears/gitlab-mcp#configuration)
- [bookstack-mcp](https://github.com/ttpears/bookstack-mcp#configuration)
- [mediawiki-mcp](https://github.com/ttpears/mediawiki-mcp#configuration)

## Manual install with `claude mcp add`

If you'd rather wire the servers up directly (no marketplace), each one runs from npm via `npx`. Repeat `--env` for each variable, put all flags **before** the server name, and use `--` to mark the start of the spawned command. `--scope` picks where the entry is written: `local` (default, current project), `user` (all your projects), or `project` (`.mcp.json` in the repo root, intended for team check-in).

### gitlab-mcp

Required: `GITLAB_URL`, `GITLAB_TOKEN` (a personal access token with `api` scope). Optional: `GITLAB_READ_TOKEN` (read-only PAT to split write traffic), `GITLAB_AUTH_MODE` (`hybrid` default), `GITLAB_TIMEOUT`, `GITLAB_MAX_PAGE_SIZE`.

```bash
claude mcp add gitlab \
  --scope user \
  --env GITLAB_URL=https://gitlab.com \
  --env GITLAB_TOKEN=glpat-your-pat \
  -- npx -y @ttpears/gitlab-mcp-server
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
  -- npx -y bookstack-mcp
```

### mediawiki-mcp

Single-wiki — required: `MEDIAWIKI_BASE_URL`, `MEDIAWIKI_USERNAME`, `MEDIAWIKI_PASSWORD` (from `Special:BotPasswords`).

```bash
claude mcp add mediawiki \
  --scope user \
  --env MEDIAWIKI_BASE_URL=https://wiki.example.com \
  --env MEDIAWIKI_USERNAME=Admin@mcp \
  --env MEDIAWIKI_PASSWORD=your-bot-password \
  -- npx -y mediawiki-mcp
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
  -- npx -y mediawiki-mcp
```

## How it's wired

This repo contains only `.claude-plugin/marketplace.json`. Each plugin entry points at its upstream GitHub repository via a `github` source; Claude Code fetches the plugin's `.claude-plugin/plugin.json` from there, which in turn declares an `mcpServers` entry that runs `npx -y <package>`. That means:

- New releases of a plugin's npm package are picked up automatically on next `npx` spawn.
- Bumping which commit of an upstream repo is pinned here only matters when the plugin manifest itself changes (version metadata, env hints, etc.).
- Marketplace-level changes (adding or removing a plugin) ship by committing to this repo.

## Adding a new plugin

1. In the upstream repo, add `.claude-plugin/plugin.json` with the plugin name, version, and an `mcpServers` entry.
2. Append a new entry to `.claude-plugin/marketplace.json` in this repo pointing at that GitHub repo.
3. Commit and push. Users pick it up with `/plugin marketplace update ttpears`.
