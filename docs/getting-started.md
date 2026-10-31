# Getting started

[Home](../README.md) · [Configuration](configuration.md) · [Troubleshooting](troubleshooting.md)

The end result is a local MCP server that Claude Code can connect to and use with
your existing service account. Installing the plugin and authenticating to the
service are separate steps.

## 1. Check the terminal environment

Run these in the terminal from which you will start Claude Code:

```text
claude --version
git --version
node --version
npx --version
```

Use Node.js 22 or newer for a setup that meets every package's declared Node
requirement. The catalog is validated with Claude Code 2.1.302. If your CLI does
not recognize a documented command, check `claude plugin --help` and update Claude
Code through your installation method.

| Platform | Setup detail |
| --- | --- |
| macOS / Linux | Use the Bash examples, or adapt environment assignment to your shell. GitLab also needs POSIX `sh`. |
| Windows with Git Bash | Use the Bash examples. Check `command -v sh` and `command -v npx` in Git Bash. |
| Windows with WSL | Install and run Claude Code, Git, Node.js, and `npx` inside the same WSL distribution. Load credentials there too. |
| Native PowerShell | Use the PowerShell credential examples. GitLab's pinned manifest still needs `sh` on the PATH inherited by Claude Code. |

Do not assume that a working Node installation in Windows is also available in
WSL, or that an IDE started from a desktop shortcut inherits your terminal's
environment. See [credential loading](configuration.md#choose-a-credential-loading-method).

## 2. Choose a service and supply credentials

Open the relevant guide before installation:

| Service | What to have ready |
| --- | --- |
| [GitLab](plugins/gitlab.md) | Instance URL and a personal access token with the access you need. |
| [BookStack](plugins/bookstack.md) | Base URL, API token ID, and API token secret. |
| [MediaWiki](plugins/mediawiki.md) | Wiki URL and bot-password username/password for authenticated access. |

Set credentials in the shell that will run `claude`, or use the documented
[user settings example](configuration.md#user-settings-for-repeatable-launches).
You do not need to clone this repository to install its plugins. Clone it only
if you want its `.env.example` or want to contribute.

## 3. Register the marketplace

In your terminal:

```text
claude plugin marketplace add ttpears/claude-plugins
claude plugin marketplace list
```

The repository name is `claude-plugins`; the marketplace's declared name is
**`ttpears`**. That declared name belongs after `@` in installation commands.

## 4. Install one plugin

Choose one command; each installs that plugin for your user across projects:

```text
claude plugin install gitlab-mcp@ttpears --scope user
claude plugin install bookstack-mcp@ttpears --scope user
claude plugin install mediawiki-mcp@ttpears --scope user
```

| Scope | Use when |
| --- | --- |
| `user` | You want the plugin available across your projects on this machine. |
| `local` | You want it only for yourself in the current project. |
| `project` | Collaborators should share the project's plugin selection. Each person still supplies their own credentials. |

For a temporary trial in one repository, substitute `--scope local`. Remember
that scope when updating or uninstalling later. Do not install both a marketplace
plugin and a [direct MCP registration](direct-install.md) for the same service
unless you intentionally want separate connections.

You can also start `claude` and enter `/plugin marketplace add
ttpears/claude-plugins`, followed by `/plugin install bookstack-mcp@ttpears`.
In-session installation opens the plugin panel where you select a scope.

## 5. Verify installation and an API request

From the terminal, inspect the installed plugins:

```text
claude plugin list
claude
```

Inside Claude Code, open `/mcp` and inspect the relevant server. Then use the
read-only starter prompt in your [service guide](../README.md#choose-your-plugin).
Choose something your service account can already see in its web interface.

| Observation | What it establishes |
| --- | --- |
| Plugin listed as installed/enabled | The plugin is registered. |
| MCP server connects | Claude Code can launch and communicate with its process. |
| A search or read succeeds | The service URL, credentials, network, and permissions work for that request. |

A plugin appearing in the list does not prove that API authentication succeeded.
If a search returns nothing, try a known page or repository before treating that
as a connectivity problem.

Next: [routine operations](operations.md), [direct installation](direct-install.md),
or [troubleshooting](troubleshooting.md).
