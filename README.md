# awesome-zcode

> ZCode plugins we've built and run.
>
> Maintained by [@buihongduc132](https://github.com/buihongduc132).

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![ZCode 3.4.2](https://img.shields.io/badge/ZCode-3.4.2-blue.svg)](https://zcode.z.ai)

**ZCode** (Z.ai Code) is the Electron-based agentic dev environment at
`/opt/ZCode` (package `@zcode/desktop`), the official harness for GLM-5.2. Its
extension model is plugin-based: a folder with `.zcode-plugin/plugin.json` can
contribute slash-commands, skills, hooks, and MCP servers.

This repo is a curated list of **the ZCode plugins we have actually implemented
and documented** — i.e. artifacts that ship a `.zcode-plugin/plugin.json`
manifest and get deployed into `~/.zcode/cli/plugins/local/`. It is not a
generic ecosystem roundup.

---

## Table of Contents

- [The plugins](#the-plugins)
- [1. `pi-commands` — 191 slash-commands](#1-pi-commands--191-slash-commands)
- [2. `zcode-intercom` — cross-agent MCP server](#2-zcode-intercom--cross-agent-mcp-server)
- [3. `hindsight` — durable memory for ZCode](#3-hindsight--durable-memory-for-zcode)
- [Where they live in `~/.zcode/`](#where-they-live-in-zcode)
- [Is there a `pi.dev/packages`-style registry for ZCode?](#is-there-a-pidevpackages-style-registry-for-zcode)
- [Contributing](#contributing)
- [License](#license)

---

## The plugins

| Plugin | Version | Type | Source repo | What it does |
| --- | --- | --- | --- | --- |
| [`pi-commands`](#1-pi-commands--191-slash-commands) | 0.2.0 | commands + hook | [`zcode-configuration`](https://github.com/buihongduc132/zcode-configuration) | 191 slash-commands imported from pi-coding-agent + a `UserPromptSubmit` hook that re-injects the command/skill index every prompt. |
| [`zcode-intercom`](#2-zcode-intercom--cross-agent-mcp-server) | 0.1.0 | MCP server | [`zcode-intercom`](https://github.com/buihongduc132/zcode-intercom) | Cross-agent intercom messaging with pi sessions via the shared intercom-core SDK + broker socket. |
| [`hindsight`](#3-hindsight--durable-memory-for-zcode) | 0.2.0 | MCP server + commands + skill + hooks | [`hindsight-zcode-local`](https://github.com/buihongduc132/hindsight-zcode-local) | Durable time-aware memory; reuses the **same banks** as the pi agent. |

---

## 1. `pi-commands` — 191 slash-commands

- **Manifest:** `pi-commands`, v0.2.0
- **Source:** [`buihongduc132/zcode-configuration`](https://github.com/buihongduc132/zcode-configuration) → `plugins/pi-commands/`
- **Deploys to:** `~/.zcode/cli/plugins/local/pi-commands/`
- **License:** MIT

**191 slash-commands** imported verbatim from the
[pi-coding-agent](https://github.com/buihongduc132/pi-plugins) prompt-templates.
ZCode derives each command name from the filename, so the pi-only `name:`
frontmatter key is redundant but harmless.

**The hook (added in v0.2.0):** a `UserPromptSubmit` hook
(`hooks/user-prompt-submit`) that re-injects the command/skill index on every
prompt. This fixes a runtime bug where long sessions drop the skill listing via
tail-window truncation — the index always survives at the most recent prompt.

```json
{
  "hooks": {
    "UserPromptSubmit": [
      { "matcher": "",
        "hooks": [{ "type": "command",
                    "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/user-prompt-submit\"",
                    "timeout": 5 }] }
    ]
  }
}
```

---

## 2. `zcode-intercom` — cross-agent MCP server

- **Manifest:** `zcode-intercom`, v0.1.0
- **Source:** [`buihongduc132/zcode-intercom`](https://github.com/buihongduc132/zcode-intercom) (canonical); inline `intercom/` fallback in `zcode-configuration`
- **Deploys to:** `~/.zcode/cli/plugins/local/zcode-intercom/`
- **Depends on:** [`intercom-core`](https://github.com/buihongduc132/intercom-core) SDK
- **License:** MIT

A ZCode MCP server for **cross-agent intercom messaging** with pi sessions (and
future frameworks) via the shared intercom-core SDK and the pi-intercom broker
socket. pi and zcode sessions intercom both ways.

**Tools exposed:**

| Tool | Behavior |
| --- | --- |
| `intercom_list` | List sessions (filter: pi / zcode / all) |
| `intercom_send` | Fire-and-forget message |
| `intercom_ask` | Send + block for reply (up to 5 min) |
| `intercom_reply` | Reply to a received message (threads `replyTo`) |
| `intercom_pending` | List outbound asks awaiting reply |
| `intercom_inbox` | Drain and read received messages |
| `intercom_status` | Connection / presence state |

**Build:** intercom-core is bundled via esbuild into `dist/mcp-server.cjs`, so
no `node_modules` are needed at deploy time.

---

## 3. `hindsight` — durable memory for ZCode

- **Manifest:** `hindsight`, v0.1.0 (package `hindsight-zcode-local` v0.2.0)
- **Source:** [`buihongduc132/hindsight-zcode-local`](https://github.com/buihongduc132/hindsight-zcode-local)
- **Registered by path** in `~/.zcode/cli/config.json` → `plugins.dirs`
- **License:** MIT

The ZCode counterpart to
[`hindsight-pi-local`](https://github.com/buihongduc132/hindsight-pi-local).
Both read the same `~/.hindsight/config.json` and resolve **identical bank IDs**,
so any fact pi retained is recallable from ZCode and vice-versa. Parity is
locked in by `test/pi-alignment.test.js`.

**What it bundles:**

1. **MCP tools** — `hindsight_search` (recall), `hindsight_context` (reflect),
   `hindsight_retain` (store), `hindsight_banks` (list banks + fact counts / top
   tags / entities).
2. **Slash commands** — `/hindsight-search`, `/hindsight-context`,
   `/hindsight-retain`, `/hindsight-banks`.
3. **Skill** — `hindsight-usage` (query strategy + budgets + memory-type guidance).
4. **Hooks** — per-turn auto-recall / auto-retain.

**Bank ID resolution mirrors pi exactly** (same precedence: `mappings[cwd]` →
`.hindsight.json` → `config.bankId` → strategy), with the only intentional
differences being display labels (`workspace`/`aiPeer` default to `"zcode"`
instead of `"pi"`).

---

## Where they live in `~/.zcode/`

All three plugins land under the ZCode local-plugins dir, registered in
`~/.zcode/cli/config.json` → `plugins.dirs`:

```
~/.zcode/cli/plugins/local/
├── pi-commands/        ← deployed by scripts/install.sh (copy)
├── zcode-intercom/     ← deployed by scripts/install.sh (build + copy dist)
└── (hindsight)         ← registered by absolute path; not copied
```

`hindsight` is registered by its checkout path
(`~/Documents/Projects/bhd/hindsight-zcode-local`) rather than copied, because
it ships a TypeScript source tree that ZCode launches directly.

---

## Is there a `pi.dev/packages`-style registry for ZCode?

**No.** As of July 2026 there is no first-party ZCode package marketplace
equivalent to [pi.dev/packages](https://pi.dev/packages). ZCode plugin discovery
today is GitHub-search-driven. The closest existing things:

| Closest thing | What it is | Gap |
| --- | --- | --- |
| [zai-org/zai-coding-plugins](https://github.com/zai-org/zai-coding-plugins) | Official Z.ai plugin collection, installable as a Claude-Code-style marketplace | Targets Claude Code, not ZCode-native; no per-package metrics UI |
| [skillpm.dev/registry](https://skillpm.dev/registry/) | Third-party agent-skills registry (mentions GLM/Z.ai) | Not ZCode-specific |
| **This list** | Static README | Not a searchable registry |

A real registry would need: (1) a package manifest convention (ZCode plugins
already have `.zcode-plugin/plugin.json`), (2) a publish/registry endpoint, and
(3) a discoverable UI with install commands. None exist first-party yet — which
is the gap this list fills for now.

---

## Contributing

This list tracks ZCode plugins **we have implemented** (shipped a
`.zcode-plugin/plugin.json` manifest and deployed into `~/.zcode/`). To add a
new entry, see [CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

[MIT](LICENSE) © 2026 buihongduc132
