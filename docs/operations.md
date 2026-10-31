# Updates and routine operations

[Home](../README.md) · [Direct installation](direct-install.md) · [Troubleshooting](troubleshooting.md)

Use the commands for the installation route and scope you actually chose.
`claude plugin` manages marketplace plugins; `claude mcp` manages direct server
registrations. The names are different: `bookstack-mcp@ttpears` is a plugin ID,
while `bookstack` is the direct server name used in our examples.

## Inspect what is installed

```text
claude plugin marketplace list
claude plugin list
```

For a direct registration, use `claude mcp get bookstack`. Inside a session, use
`/mcp` to inspect the running connection. These operations answer different
questions: a catalog may be registered without any plugin installed, and a plugin
may be installed while its server fails to authenticate.

## Update marketplace plugins

Refresh the catalog, then update each installed plugin. Example for a user-scope
BookStack installation:

```text
claude plugin marketplace update ttpears
claude plugin update bookstack-mcp@ttpears --scope user
```

Repeat the plugin command with `gitlab-mcp@ttpears` or `mediawiki-mcp@ttpears` as
needed. Use your installation's scope. Start a new Claude Code session and repeat
a read-only service check after an update.

Inside Claude Code, the Marketplaces tab in `/plugin` offers **Update marketplace**
and an auto-update toggle. Third-party marketplaces default to auto-update off.
Refreshing a listing from the CLI is distinct from updating an installed plugin;
the explicit commands above make both steps visible. See the
[official update behavior](https://code.claude.com/docs/en/discover-plugins#keep-plugins-updated).

## Understand the three layers of versioning

| Layer | Where it is selected | What changing it does |
| --- | --- | --- |
| Marketplace catalog | This repository's default branch | Changes available entries, display metadata, and source pins. |
| Plugin manifest | Each entry's `source.sha` in `.claude-plugin/marketplace.json` | Selects the upstream plugin's files, configuration, and declared version. |
| npm server package | The command and arguments inside that upstream manifest | Selects what server executable is resolved when it starts. |

The catalog pins each upstream repository to a release commit. Its current
upstream commands still request unversioned packages (`npx -y bookstack-mcp`, for
example). An immutable source commit therefore does **not** mean the npm runtime
is fixed to that release. `npx` also handles its own resolution and caching;
do not assume every restart downloads a new package.

To require an exact server version, use a [direct registration](direct-install.md)
with `package@version`. The version table in the README describes the catalog's
manifest snapshot, not a measurement of a running server's executable.

## Update a direct registration

A plugin update command does not change your direct MCP command. Select the new
package version, check that release's configuration requirements, and replace
your registration at the same scope. For example:

```text
claude mcp remove bookstack --scope user
claude mcp add --scope user --transport stdio bookstack -- npx -y bookstack-mcp@5.7.1
```

The version shown is the documented snapshot; substitute the exact release you
intend to use. This example inherits credentials. If your old registration used
`--env`, recreate its required environment fields too. For `.mcp.json`, edit the
package version in its `args` array and preserve the other servers.

## Disable or remove a plugin

Temporarily disable and later re-enable an installed plugin:

```text
claude plugin disable bookstack-mcp@ttpears --scope user
claude plugin enable bookstack-mcp@ttpears --scope user
```

Uninstall it from the chosen scope:

```text
claude plugin uninstall bookstack-mcp@ttpears --scope user
```

Use `local` or `project` instead if that is where you installed it. A copy enabled
at another scope or a separate direct registration may still provide the service.
Inspect `/mcp` again in a fresh session.

Removing the entire marketplace is broader than uninstalling one plugin: Claude
Code removes plugins installed from that marketplace. Use the marketplace
management UI or consult `claude plugin marketplace remove --help` when that is
your intent.

Neither uninstalling a plugin nor removing a direct registration revokes an API
token in GitLab, BookStack, or MediaWiki. Token retirement is a separate action
in the service that issued it.

## Switch installation routes

To move from a plugin to a direct exact-version server, first disable the plugin
at its active scope, add the direct registration, then verify a read-only request.
To move back, remove the direct server, reinstall or enable the plugin, and
verify again. Keep your credentials in the method appropriate to the new route.
There is no need to delete unrelated plugin caches or user settings.
