# BookStack MCP

[Home](../../README.md) · [Credential loading](../configuration.md) · [Troubleshooting](../troubleshooting.md)

- **Install ID:** `bookstack-mcp@ttpears`
- **Published package:** `bookstack-mcp`
- **Catalog manifest:** `5.7.1`

Use this plugin to find and read BookStack documentation. Write tools are
available only when explicitly enabled, and all API access remains subject to
the account's BookStack permissions.

## Prepare the API token

Create a BookStack API token for an account with API access and access to the
books/pages you need. Keep both values: the **token ID** and **token secret** are
different fields. Set the BookStack base URL, including any deployment subpath,
rather than a single page URL or an `/api` endpoint.

## Configure and install

Bash, Git Bash, or WSL:

```bash
export BOOKSTACK_BASE_URL='https://bookstack.example.com'
export BOOKSTACK_TOKEN_ID='replace-with-token-id'
export BOOKSTACK_TOKEN_SECRET='replace-with-token-secret'
export BOOKSTACK_ENABLE_WRITE='false'
claude plugin marketplace add ttpears/claude-plugins
claude plugin install bookstack-mcp@ttpears --scope user
claude
```

PowerShell environment equivalent:

```powershell
$env:BOOKSTACK_BASE_URL = 'https://bookstack.example.com'
$env:BOOKSTACK_TOKEN_ID = 'replace-with-token-id'
$env:BOOKSTACK_TOKEN_SECRET = 'replace-with-token-secret'
$env:BOOKSTACK_ENABLE_WRITE = 'false'
```

Then run the same `claude plugin` and `claude` commands. For persistent settings,
see the [user settings example](../configuration.md#user-settings-for-repeatable-launches).

## Verify a search and a page read

Open `/mcp`, inspect the BookStack server, then ask:

> Search BookStack for onboarding, list matching page titles, and summarize one page I can access. Do not edit any content.

Choose a search term that exists in your instance. If you can connect but a known
page is missing, compare the token account's visibility with your browser login.

## Enable writes when needed

Change `BOOKSTACK_ENABLE_WRITE` to the string `true` in your credential-loading
method, then start a new session. Enabling the server's write tools does not
increase the underlying account's permissions. Creating, editing, deleting, and
administrative operations still require the corresponding BookStack access.

Keep write tools disabled for workflows that only need search and reading. To
turn them off again, restore `false` and restart the server through a new session.

## Runtime and optional settings

The package declares Node.js **18+**; Node.js 22+ is the common recommendation for
using this catalog's three servers together. The
[pinned manifest](https://github.com/ttpears/bookstack-mcp/blob/e662ee4fabf84dfc4f18b0a0cdf3ab4cd4920c2f/.claude-plugin/plugin.json)
launches `npx -y bookstack-mcp@5.7.1` directly. Both the GitHub source and npm
runtime are pinned. This catalog selects the reviewed launcher/asset correction
for the already published 5.7.1 runtime.

| Variable | Default | Purpose |
| --- | --- | --- |
| `BOOKSTACK_ENABLE_WRITE` | `false` | Register write operations. |
| `BOOKSTACK_TIMEOUT_MS` | `30000` | Timeout per HTTP attempt. |
| `BOOKSTACK_RETRY_TIMEOUT_MS` | `60000` | Total budget for an API request, including queueing and retries. |
| `BOOKSTACK_MAX_CONCURRENCY` | `4` | Concurrent in-flight requests per base URL. |

For private certificate authorities, configure the Node process to trust your CA
as described in [Troubleshooting](../troubleshooting.md#network-and-certificate-failures).
The upstream server also has a TLS-verification bypass option; it disables
certificate checks and should not be the default fix for a connection failure.

Reference: [README at the pinned source](https://github.com/ttpears/bookstack-mcp/blob/e662ee4fabf84dfc4f18b0a0cdf3ab4cd4920c2f/README.md)
and [environment example](https://github.com/ttpears/bookstack-mcp/blob/e662ee4fabf84dfc4f18b0a0cdf3ab4cd4920c2f/.env.example).
The marketplace uses local stdio; upstream HTTP/OAuth instructions describe a
different deployment.
