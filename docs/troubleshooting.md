# Troubleshooting

[Home](../README.md) · [Configuration](configuration.md) · [Operations](operations.md)

Start with the stage that failed. This avoids replacing working credentials when
the actual issue is a missing executable or an incorrect plugin ID.

## Identify the failed stage

| Stage | Check | Where to look next |
| --- | --- | --- |
| Add marketplace | `claude plugin marketplace list` | GitHub connectivity, Git installation, repository source. |
| Install plugin | `claude plugin list` | Full install ID, selected scope, plugin-panel errors. |
| Start MCP process | `/mcp` in Claude Code | Node.js, `npx`, GitLab's `sh`, inherited environment. |
| Authenticate | Read one known project or page | Credential pair, URL, token expiry, service permissions. |
| Perform a particular action | Compare a successful read with the failed action | Write-tool settings and account rights. |

## Plugin not found or not enabled

Use the catalog's declared marketplace name, `ttpears`, after `@`:

```text
claude plugin marketplace list
claude plugin marketplace update ttpears
claude plugin install bookstack-mcp@ttpears --scope user
claude plugin list
```

The GitHub repository is named `claude-plugins`; `bookstack-mcp@claude-plugins`
is not this catalog's install ID. If a plugin is already installed but disabled,
enable it at its active scope. If a project or organization controls plugin
availability, inspect the relevant settings rather than reinstalling repeatedly.

## A process will not start

Check executable resolution in the launch shell:

```bash
command -v claude
command -v node
command -v npx
command -v sh
```

```powershell
Get-Command claude, node, npx
# GitLab's upstream plugin wrapper also requires:
Get-Command sh
```

Run `node --version` and compare with the [catalog requirements](../README.md#catalog-snapshot).
GitLab's plugin wrapper requires `sh` as well as `npx`; BookStack and MediaWiki
invoke `npx` directly. A Node installation in another WSL distribution does not
make it available to this process.

For `npx` download errors, check the configured npm registry and network in that
same environment. A corporate registry that lacks a package or selected version
can fail even when GitHub access works. Do not treat that as a service token
failure.

## Credentials are set but the server cannot authenticate

Confirm where you set them. Terminal exports affect processes started from that
terminal; they do not update a running IDE or desktop app. A `.env` file needs to
be [loaded explicitly](configuration.md#explicitly-load-the-environment-example).

Use the presence-only checks in [Configuration](configuration.md#change-accounts-or-rotate-a-token)
instead of printing token values. If user or project settings contain the same
variables, check `/status` and resolve the duplicate source. Restart Claude Code
after changing the effective configuration.

### GitLab token errors

- For the documented local setup, supply one of `GITLAB_TOKEN` or
  `GITLAB_READ_TOKEN`. Both nonempty values produce a configuration error.
- Remove `GITLAB_AUTH_MODE` and `GITLAB_SHARED_ACCESS_TOKEN`; they no longer select
  an authentication mode.
- Check expiry, `api` or `read_api` scope, instance URL, and the account's project
  visibility. A token for one self-hosted instance does not authenticate to another.
- A request for per-call credentials on an advanced generic tool can reflect the
  upstream tool's credential policy; it does not necessarily mean the fallback
  token failed for ordinary tools.

See [GitLab setup](plugins/gitlab.md).

### BookStack authentication or write errors

Check both token ID and token secret, the base URL, and API access for that
account. A successful read does not imply permission to update a page.
`BOOKSTACK_ENABLE_WRITE=false` intentionally leaves write tools disabled. To
enable writes, change it to `true`, restart, and confirm the token account has
the needed BookStack permissions. See [BookStack setup](plugins/bookstack.md).

### MediaWiki login or wiki-selection errors

Use the full bot username (`Admin@mcp`) and generated bot password together.
Check the credentials on the wiki they belong to. For multi-wiki configuration,
compare names in `MEDIAWIKI_WIKIS` with uppercase credential suffixes and inspect
`list-wikis` through Claude Code.

Specify the wiki explicitly when a search should not fan out. If a read succeeds
but an edit fails, check bot grants, account rights, and page protection. See
[MediaWiki setup](plugins/mediawiki.md).

## Network and certificate failures

Check the service base URL, DNS, VPN/private-network access, and proxy settings
from the same host environment as the server. Browser access alone is not proof
that Node trusts the same certificate chain or uses the same proxy.

For an internal CA, Node supports `NODE_EXTRA_CA_CERTS` pointing to a PEM file
containing trusted additional CA certificates. Set it before starting Claude
Code, using a path accessible in that environment. A Windows path will need an
appropriate WSL equivalent if the server runs in WSL. Consult
[Node's certificate configuration](https://nodejs.org/api/cli.html#node_extra_ca_certsfile).

Do not use a global TLS-verification disable as a routine repair. For a BookStack
server with the upstream bypass option already enabled, record that fact when
diagnosing certificate behavior.

## Duplicate tools or old behavior

Check whether the service is installed as both a plugin and a direct MCP server,
or enabled at several scopes. Use `claude plugin list` and `/mcp` to identify
which connections are present. Remove or disable only the unwanted registration.

For an update, refresh the marketplace **and** update the installed plugin at the
correct scope, then start a new session. Remember that the catalog's source pin
and the npm server version are [separate layers](operations.md#understand-the-three-layers-of-versioning).
An exact-version direct registration remains on its named package version until
you edit it.

## Reporting a problem

Report catalog entries, broken links, or these setup instructions to
[claude-plugins issues](https://github.com/ttpears/claude-plugins/issues).
Report API failures, server crashes, or tool behavior to the relevant
[GitLab](https://github.com/ttpears/gitlab-mcp/issues),
[BookStack](https://github.com/ttpears/bookstack-mcp/issues), or
[MediaWiki](https://github.com/ttpears/mediawiki-mcp/issues) project.

Include:

- OS and shell, including whether the process runs in WSL.
- Claude Code and Node.js versions.
- Installation route, plugin/server name, and scope.
- Catalog source commit or exact npm package version, when known.
- The failing stage, a minimal read-only reproduction, and relevant error text.
- Whether another read works with the same account.

Redact tokens, passwords, authorization headers, private page content, and
internal hostnames as appropriate. State variable names and whether they are set;
do not include credential values or dump your complete settings file.
