# awesome-zcode

ZCode plugins we've built.

| Plugin | Version | What it does | Source |
| --- | --- | --- | --- |
| `pi-commands` | 0.2.0 | 191 slash-commands + `UserPromptSubmit` hook (re-injects the command/skill index every prompt) | [zcode-configuration](https://github.com/buihongduc132/zcode-configuration) → `plugins/pi-commands/` |
| `zcode-intercom` | 0.1.0 | Cross-agent MCP server: `intercom_list` / `send` / `ask` / `reply` / `pending` / `inbox` / `status` | [zcode-intercom](https://github.com/buihongduc132/zcode-intercom) |
| `hindsight` | 0.2.0 | Durable-memory MCP server (`hindsight_search` / `context` / `retain` / `banks`) + 4 slash-commands + `hindsight-usage` skill + hooks. Shares banks with pi. | [hindsight-zcode-local](https://github.com/buihongduc132/hindsight-zcode-local) |

A plugin = a folder with `.zcode-plugin/plugin.json` deployed to `~/.zcode/cli/plugins/local/`.
