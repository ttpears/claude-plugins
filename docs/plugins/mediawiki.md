# MediaWiki MCP

[Home](../../README.md) · [Credential loading](../configuration.md) · [Troubleshooting](../troubleshooting.md)

- **Install ID:** `mediawiki-mcp@ttpears`
- **Published package:** `mediawiki-mcp`
- **Catalog manifest:** `2.5.4`

Use this plugin to search, read, and edit one or more named wikis. The upstream
server targets MediaWiki 1.42+ and uses REST and Action APIs. The examples here
configure authenticated access with MediaWiki bot passwords.

## Create bot credentials

On each wiki, open `Special:BotPasswords` for the account you want to use and
create a bot password with the grants needed for your workflow. Record both the
full bot username, such as `Admin@mcp`, and its generated password. The bot name
alone and your ordinary web-login password are not that credential pair.

The bot's grants and account permissions determine what it can do. Use the wiki's
base URL, such as `https://wiki.example.com`, rather than a page URL. See the
upstream configuration if the wiki is deployed below a URL subpath.

## Single wiki

Bash, Git Bash, or WSL:

```bash
export MEDIAWIKI_BASE_URL='https://wiki.example.com'
export MEDIAWIKI_USERNAME='Admin@mcp'
export MEDIAWIKI_PASSWORD='replace-with-bot-password'
claude plugin marketplace add ttpears/claude-plugins
claude plugin install mediawiki-mcp@ttpears --scope user
claude
```

PowerShell environment equivalent:

```powershell
$env:MEDIAWIKI_BASE_URL = 'https://wiki.example.com'
$env:MEDIAWIKI_USERNAME = 'Admin@mcp'
$env:MEDIAWIKI_PASSWORD = 'replace-with-bot-password'
```

Then run the same `claude plugin` and `claude` commands.

## Multiple wikis

Use a comma-separated set of `Name:URL` pairs instead of the single-wiki settings.
Choose simple names such as `Main` and `Dev`, then use their uppercase forms in
credential variable suffixes:

```bash
export MEDIAWIKI_WIKIS='Main:https://wiki.example.com,Dev:https://dev.wiki.example.com'
export MEDIAWIKI_DEFAULT_WIKI='Main'
export MEDIAWIKI_USERNAME_MAIN='Admin@mcp'
export MEDIAWIKI_PASSWORD_MAIN='replace-with-main-bot-password'
export MEDIAWIKI_USERNAME_DEV='Admin@mcp'
export MEDIAWIKI_PASSWORD_DEV='replace-with-dev-bot-password'
```

PowerShell uses the same names with `$env:` assignments:

```powershell
$env:MEDIAWIKI_WIKIS = 'Main:https://wiki.example.com,Dev:https://dev.wiki.example.com'
$env:MEDIAWIKI_DEFAULT_WIKI = 'Main'
$env:MEDIAWIKI_USERNAME_MAIN = 'Admin@mcp'
$env:MEDIAWIKI_PASSWORD_MAIN = 'replace-with-main-bot-password'
$env:MEDIAWIKI_USERNAME_DEV = 'Admin@mcp'
$env:MEDIAWIKI_PASSWORD_DEV = 'replace-with-dev-bot-password'
```

Remove old single-wiki settings from the configuration method you used when
switching modes, so there is one clear source of wiki configuration. Each wiki
has its own bot credential pair. The default wiki selects a target for operations
that use a default; some searches and listings fan out when no wiki is specified.
Name a wiki explicitly when you want a request restricted to one instance.

## Verify wiki registration and a read

Open `/mcp`, inspect the MediaWiki server, then ask:

> List the configured wikis and their authentication status. Search only the Main wiki for onboarding and summarize the matching titles. Do not edit any pages.

Replace `Main` with your configured wiki name. The server's `list-wikis` tool is
useful for distinguishing a missing wiki from a login problem. A server process
starting successfully does not prove each wiki authenticated.

If reading works and editing fails, check bot grants, account rights, and page
protection on that wiki. There is no BookStack-style `BOOKSTACK_ENABLE_WRITE`
switch for this connection.

## Runtime and deployment

The package declares Node.js **20+**; Node.js 22+ meets the catalog's combined
requirements. The
[pinned manifest](https://github.com/ttpears/mediawiki-mcp/blob/7d49e4632df036c1ac53565ef1b78c88e9e64273/.claude-plugin/plugin.json)
launches `npx -y mediawiki-mcp` directly. This catalog does not start an HTTP
listener or configure an OAuth broker.

Reference: [configuration at the pinned release](https://github.com/ttpears/mediawiki-mcp/blob/7d49e4632df036c1ac53565ef1b78c88e9e64273/README.md#configuration),
[bot-password instructions](https://github.com/ttpears/mediawiki-mcp/blob/7d49e4632df036c1ac53565ef1b78c88e9e64273/README.md#creating-a-bot-password),
and [environment example](https://github.com/ttpears/mediawiki-mcp/blob/7d49e4632df036c1ac53565ef1b78c88e9e64273/.env.example).
