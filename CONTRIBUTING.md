# Contributing to awesome-zcode

This list tracks ZCode plugins **we have implemented and documented** — i.e.
artifacts that ship a `.zcode-plugin/plugin.json` manifest and get deployed into
`~/.zcode/cli/plugins/local/` (or registered by path in
`~/.zcode/cli/config.json` → `plugins.dirs`).

It is **not** a generic ecosystem roundup. Infra repos, config-as-code repos,
SDKs, and MCP aggregates that are not themselves ZCode plugins do not belong
here.

## What qualifies

A repo/path qualifies as a ZCode plugin entry if it has:

```
<plugin-root>/.zcode-plugin/plugin.json
```

with a `name`, `version`, and at least one of: `commands`, `skills`, `hooks`,
`mcpServers`.

## Entry format

Add a row to **The plugins** table in `README.md`:

```markdown
| [`<name>`](#<n>-name--one-line) | <version> | <type> | [<source-repo>](repo-url) | **Short bold lead.** One sentence. |
```

Then add a dedicated `## <n>. \`<name>\`` subsection below with:

- **Manifest** — plugin name + version
- **Source** — link to the repo (and subpath if the plugin lives in a subfolder)
- **Deploys to** / **Registered by** — where it lands in `~/.zcode/`
- **License**
- **What it does** — commands / tools / skills / hooks it contributes

### Type values

Pick the most specific:

| Type | Meaning |
| --- | --- |
| commands | Slash-commands only |
| MCP server | Contributes an `mcpServers` entry |
| commands + hook | Slash-commands + a hook |
| MCP server + commands + skill + hooks | Multiple contributions (hindsight-style) |

## PR checklist

- [ ] The entry ships a real `.zcode-plugin/plugin.json` (link to it in the PR).
- [ ] Version in the table matches `plugin.json` `version`.
- [ ] Dedicated subsection has manifest / source / deploy-path / license.
- [ ] Listed tools/commands/skills match what's actually in the manifest.
- [ ] No live secrets in linked content.
