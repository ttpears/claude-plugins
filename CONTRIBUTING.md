# Contributing to the marketplace

[Home](README.md) · [Operations and versioning](docs/operations.md)

This repository owns catalog entries, source pins, and installation documentation.
Server code and plugin manifests belong to the upstream projects. Make a server
behavior change there first; update this catalog when the published release and
its manifest are ready to distribute.

## Repository map

| Path | Purpose |
| --- | --- |
| `.claude-plugin/marketplace.json` | Marketplace identity, plugin discovery metadata, and pinned GitHub sources. |
| `README.md` | First installation path and catalog snapshot. |
| `docs/` | Platform, credential, service, lifecycle, and troubleshooting guides. |
| `.env.example` | Bash-compatible credential template with placeholders. |
| `examples/` | JSON examples intended to be merged into a user's own configuration. |
| `.github/workflows/validate.yml` | Claude Code 2.1.302 strict catalog validation and commit whitespace checks. |

## Set up a documentation or catalog change

```bash
git clone https://github.com/ttpears/claude-plugins.git
cd claude-plugins
git switch -c docs/describe-your-change
```

Use Node.js 22+ for the pinned validator. There is no application build or local
dependency installation required for ordinary catalog edits.

## Refresh an existing release pin

1. Read published package metadata, not only the upstream branch tip:

   ```text
   npm view @ttpears/gitlab-mcp-server version gitHead engines --json
   npm view bookstack-mcp version gitHead engines --json
   npm view mediawiki-mcp version gitHead engines --json
   ```

2. Inspect the release commit's `.claude-plugin/plugin.json`, `package.json`, and
   setup documentation. Confirm that the plugin name matches this catalog's
   entry, the versions agree, and the package exists in npm. If `gitHead` is
   missing, verify a release tag and its package metadata before selecting a
   commit. Do not guess a commit from the version string.
3. Review launch-command changes, required runtimes, environment variables, and
   permission defaults. A newer default branch can contain an unpublished
   manifest; it is not automatically the appropriate catalog source.
4. Change the entry's `source.sha` to the full reviewed commit. Preserve its
   `source: github` and `repo` fields. Plugin versions remain authoritative in the
   upstream manifest; do not add a competing catalog `version` field.
5. Update the README snapshot date/table, service guide version and pinned links,
   direct registration examples, and any changed environment guidance. The
   [project MCP example](examples/mcp-project.example.json) has an exact package
   version too. Keep paired file and inline JSON examples consistent.

The catalog's `source.sha` pins plugin files. It does not rewrite an upstream
unversioned npm command. If an upgrade requires a fixed server runtime, change
the upstream plugin definition or document a direct exact-version registration.

## Add a plugin

Publish the server package and its upstream `.claude-plugin/plugin.json` first.
The manifest must define the intended name and MCP launch configuration. Then
add a catalog entry with a matching name, GitHub repository, full release SHA,
and useful display metadata.

Add a service guide with required variables, platform requirements, write
behavior, an installation command, and a read-only verification prompt. Update
the README navigation and catalog table, `.env.example`, and relevant examples.
Do not copy server-specific runtime components into this catalog merely to
override an upstream manifest; make the ownership of those components explicit.

## Validate the change

With the tested CLI installed:

```text
claude plugin validate . --strict
git diff --check
```

Or invoke the exact CI validator through npm:

```text
npx --yes @anthropic-ai/claude-code@2.1.302 plugin validate . --strict
```

For a changed pin, check out the selected upstream commit and run
`claude plugin validate /path/to/upstream --strict` there too. These checks do not
make authenticated service requests.

For documentation edits, check every local link and section anchor, parse JSON
examples, and syntax-check Bash/PowerShell snippets using the relevant shell.
Confirm command flags against `claude plugin <command> --help` or
`claude mcp <command> --help`. Do not run lifecycle examples against your everyday
configuration just to verify their spelling.

## Manual acceptance check

When changing install behavior or releasing a new source pin, use a deliberate
test configuration and a service account you control. Add the marketplace,
install the selected plugin, start a session with the documented credentials,
and make a read-only request to a known resource. Check write-tool availability
against the documented setting; changing content is not required to prove a
documentation fix.

Record what was actually tested and any unavailable service access in the PR.
The existing CI validates catalog structure and whitespace, not GitHub-source
availability, server runtime behavior, or authentication to all three services.

## Submit a useful pull request

Describe the user-visible problem and resulting behavior, name any changed
release pins, and include validation results. For a command correction, provide
the corrected form and the CLI version used to check it. For a server limitation,
link the relevant upstream release or source rather than adding an unsupported
compatibility promise.

Format and source references:
[marketplace schema](https://code.claude.com/docs/en/plugins/marketplace-reference),
[plugin manifest](https://code.claude.com/docs/en/plugins-reference), and
[plugin CLI commands](https://code.claude.com/docs/en/plugins/cli-reference).
