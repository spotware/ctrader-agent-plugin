# cTrader plugin for AI agents

The `ctrader` plugin connects your AI agent to [cTrader](<https://ctrader.com/>) by Spotware Systems Ltd. Your agent reads account and market data through the cTrader MCP server and answers with analytics, while every trading action stays with you in cTrader.

With the plugin, your agent can:

- review balances, equity, open positions, pending orders and trade history;
- compare bid and ask prices, inspect historical candles and calculate summaries from your connected account;
- diagnose connection problems and find orders by their client identifiers;
- prepare a link to a new-order form, or to an existing position or pending order.

A link opens cTrader, where you review the order, close the position, cancel the pending order or edit its settings yourself. Preparing a link executes nothing. The plugin does not place trades, sell products or process payments.

## What the plugin connects to

The plugin declares one MCP server: the cTrader MCP server at <https://mcp.ctrader.com/mcp>, reached over Streamable HTTP. The first time your agent connects, a browser window opens where you sign in to cTrader and grant access; connect with read-only data access. Your agent then sends its tool calls, such as a request for a balance or for prices, to that server, and the server answers from your cTrader account.

The plugin contains no scripts and no hooks. Its skill, [`skills/ctrader-analytics/`](skills/ctrader-analytics/), tells your agent how to read the server's data and present it.

## Privacy

The [Spotware Privacy Policy](<https://www.spotware.com/privacy-policy/>) describes how Spotware handles the data of your cTrader account, and the [Spotware Terms of Service](<https://www.spotware.com/terms-of-service/>) apply to your use of cTrader.

## Install

### Claude Code

```bash
claude plugin marketplace add spotware/ctrader-agent-plugin
claude plugin install ctrader@ctrader-agent-plugin
```

Grok Build reads the plugins you install for Claude Code, so the same two commands make the plugin available there.

### Gemini CLI

```bash
gemini extensions install https://github.com/spotware/ctrader-agent-plugin
```

### GitHub Copilot CLI

```bash
copilot plugin install spotware/ctrader-agent-plugin
```

### Kiro

Open the Powers panel, choose **Add Custom Power**, then **Import power from a repository URL**, and enter `https://github.com/spotware/ctrader-agent-plugin`.

### Other agents

The repository root follows the [Agent Plugins](<https://agent-plugins.org/>) specification: `plugin.json` describes the plugin and `mcp.json` names its server. An agent that reads Agent Plugins loads the plugin from the repository root.

## Learn more

For cTrader accounts and the cTrader MCP server, see the [cTrader Help Centre](<https://help.ctrader.com/>). [CHANGELOG.md](CHANGELOG.md) lists the changes in each version of the plugin.

## Releases

Each release is one commit on `main`, tagged `v<version>`, and `main` changes only at a release.

## Contributing

Issues and pull requests are welcome as proposals. See [CONTRIBUTING.md](CONTRIBUTING.md) for how Spotware publishes this repository, how to report an issue, and how to propose a change. By participating, you agree to abide by the [Code of Conduct](CODE_OF_CONDUCT.md).

## License

This plugin is proprietary to Spotware Systems Ltd and forms part of the cTrader platform. The [Spotware End User License Agreement](<https://www.spotware.com/eula/>) governs its use. See [LICENSE](LICENSE) and [COPYRIGHT.md](COPYRIGHT.md) for the full terms.

## Security

To report a security vulnerability, follow [SECURITY.md](SECURITY.md). Do not open a public issue for a security report.

---

cTrader is a registered trademark of Spotware Systems Ltd.
