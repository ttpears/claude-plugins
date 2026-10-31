# Configuration and credentials

[Home](../README.md) · [Getting started](getting-started.md) · [Service guides](../README.md#choose-your-plugin)

All three pinned plugins start a local process and read service configuration
from environment variables. This catalog does not ask for credentials through a
plugin-specific configuration form. Use one of the methods below, then start a
new Claude Code session so the server receives the new environment.

## Choose a credential-loading method

| Method | Best fit | What persists |
| --- | --- | --- |
| Shell variables | Trying a service or choosing a different account for a session | Values belong to that shell process and its children. |
| Explicitly loaded `.env` | Reusing a local Bash configuration | The file persists; export it again in each new launch shell. |
| User settings `env` | Repeated launches, including IDE or desktop entry points that read user settings | Values are stored in your personal Claude Code settings file. |
| Direct MCP registration `--env` | A separate environment for one directly registered server | Values are stored in that MCP registration's scope. |

The shell and settings methods supply values to Claude Code's child processes,
not just one plugin. Use [direct registration](direct-install.md) with per-server
`env` when you need to isolate which connection receives which credentials.

## Shell variables

For example, BookStack in Bash, Git Bash, or WSL:

```bash
export BOOKSTACK_BASE_URL='https://bookstack.example.com'
export BOOKSTACK_TOKEN_ID='replace-with-token-id'
export BOOKSTACK_TOKEN_SECRET='replace-with-token-secret'
export BOOKSTACK_ENABLE_WRITE='false'
claude
```

In PowerShell:

```powershell
$env:BOOKSTACK_BASE_URL = 'https://bookstack.example.com'
$env:BOOKSTACK_TOKEN_ID = 'replace-with-token-id'
$env:BOOKSTACK_TOKEN_SECRET = 'replace-with-token-secret'
$env:BOOKSTACK_ENABLE_WRITE = 'false'
claude
```

Single quotes keep literal `$` characters in credentials from being interpreted
by the shell. If a credential itself contains a quote, use that shell's escaping
rules or the JSON settings method below. Set the variables before starting the
process that hosts Claude Code; an already-running app will not inherit later
changes made in another terminal.

## Explicitly load the environment example

From a local checkout, using Bash, Git Bash, or WSL:

```bash
cp .env.example .env
# Edit .env to include only the services you use.
set -a
. ./.env
set +a
claude
```

The [example](../.env.example) uses shell assignment syntax. Sourcing it runs it
as shell code, so load your own trusted file. A file named `.env` is not loaded
automatically by this marketplace. PowerShell cannot dot-source this Bash file;
use `$env:NAME = 'value'` assignments or JSON settings instead.

This repository ignores `.env` and `.env.*`, except `.env.example`. Those ignore
rules do not protect files saved in another repository or values already
committed to Git.

## User settings for repeatable launches

Merge the `env` object from
[user-settings.example.json](../examples/user-settings.example.json) into your
existing `~/.claude/settings.json`. On Windows, `~` is your user profile directory.
Replace the example values with your own; preserve unrelated settings.

```json
{
  "env": {
    "BOOKSTACK_BASE_URL": "https://bookstack.example.com",
    "BOOKSTACK_TOKEN_ID": "replace-with-token-id",
    "BOOKSTACK_TOKEN_SECRET": "replace-with-token-secret",
    "BOOKSTACK_ENABLE_WRITE": "false"
  }
}
```

The file must be valid JSON: no comments or trailing commas, and environment
values should be strings. These are stored values, not an OS keychain entry.
Keep real credentials in personal configuration rather than a shared project
settings file. If the same variable is set in more than one place, inspect
`/status` and the [Claude Code settings precedence](https://code.claude.com/docs/en/settings)
before changing several files at once.

## Service variables at a glance

| Service | Variables for the documented local setup | Important distinction |
| --- | --- | --- |
| GitLab | `GITLAB_URL` and either `GITLAB_TOKEN` or `GITLAB_READ_TOKEN` | The two fallback token variables are mutually exclusive. |
| BookStack | `BOOKSTACK_BASE_URL`, `BOOKSTACK_TOKEN_ID`, `BOOKSTACK_TOKEN_SECRET` | Token ID and secret are separate values. Writes default to disabled. |
| MediaWiki | `MEDIAWIKI_BASE_URL`, `MEDIAWIKI_USERNAME`, `MEDIAWIKI_PASSWORD` | Use the full bot username, such as `Admin@mcp`. |
| MediaWiki multi-wiki | `MEDIAWIKI_WIKIS`, `MEDIAWIKI_DEFAULT_WIKI`, and per-wiki credential variables | Use uppercase wiki-name suffixes, such as `_MAIN`. |

These are the variables needed for the setup paths in this guide, not every
option each server supports. See [GitLab](plugins/gitlab.md),
[BookStack](plugins/bookstack.md), and [MediaWiki](plugins/mediawiki.md) for
defaults, write behavior, and links to the pinned upstream documentation.

## Change accounts or rotate a token

Update the method you actually use: shell, personal settings, or a direct server's
registration. Remove the superseded value from that same place. Restart Claude
Code and repeat a read-only API check. If you use GitLab, make sure the account
switch did not leave both fallback token variables set.

To check presence without displaying values:

```bash
if [ -n "${BOOKSTACK_TOKEN_SECRET:-}" ]; then
  printf 'BookStack token secret is set\n'
else
  printf 'BookStack token secret is missing\n'
fi
```

```powershell
if ([string]::IsNullOrEmpty($env:BOOKSTACK_TOKEN_SECRET)) {
    'BookStack token secret is missing'
} else {
    'BookStack token secret is set'
}
```

These commands inspect the shell environment only. They cannot prove what an
already-running server received or whether a settings-file value will override it.
