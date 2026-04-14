# ttpears claude-plugins

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) that bundles the MCP servers published by [@ttpears](https://github.com/ttpears) into a single installable catalog. Each plugin is a thin manifest over a published npm package — installing a plugin wires its MCP server into Claude Code via `npx`.

## Plugins

| Plugin | Package | Purpose |
| --- | --- | --- |
| `gitlab-mcp` | [`@ttpears/gitlab-mcp-server`](https://www.npmjs.com/package/@ttpears/gitlab-mcp-server) | GitLab GraphQL client with team activity tools |
| `bookstack-mcp` | [`bookstack-mcp`](https://www.npmjs.com/package/bookstack-mcp) | BookStack wiki search, read, write, and admin |
| `mediawiki-mcp` | [`mediawiki-mcp`](https://www.npmjs.com/package/mediawiki-mcp) | MediaWiki multi-wiki client over the REST API |
| `mysql-mcp` | [`@ttpears/mysql-mcp`](https://www.npmjs.com/package/@ttpears/mysql-mcp) | MySQL analytics and user behavior analysis |

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
/plugin install mysql-mcp@ttpears
```

Update the catalog later with `/plugin marketplace update ttpears`.

## Configuration

Each MCP server reads its credentials and endpoints from environment variables. Set these in your shell (or `~/.claude/settings.json`) before starting Claude Code — the spawned `npx` process inherits them.

Check each plugin's upstream repo for the full list of required variables:

- [gitlab-mcp](https://github.com/ttpears/gitlab-mcp#configuration)
- [bookstack-mcp](https://github.com/ttpears/bookstack-mcp#configuration)
- [mediawiki-mcp](https://github.com/ttpears/mediawiki-mcp#configuration)
- [mysql-mcp](https://github.com/ttpears/mysql-mcp#configuration)

## How it's wired

This repo contains only `.claude-plugin/marketplace.json`. Each plugin entry points at its upstream GitHub repository via a `github` source; Claude Code fetches the plugin's `.claude-plugin/plugin.json` from there, which in turn declares an `mcpServers` entry that runs `npx -y <package>`. That means:

- New releases of a plugin's npm package are picked up automatically on next `npx` spawn.
- Bumping which commit of an upstream repo is pinned here only matters when the plugin manifest itself changes (version metadata, env hints, etc.).
- Marketplace-level changes (adding or removing a plugin) ship by committing to this repo.

## Adding a new plugin

1. In the upstream repo, add `.claude-plugin/plugin.json` with the plugin name, version, and an `mcpServers` entry.
2. Append a new entry to `.claude-plugin/marketplace.json` in this repo pointing at that GitHub repo.
3. Commit and push. Users pick it up with `/plugin marketplace update ttpears`.
