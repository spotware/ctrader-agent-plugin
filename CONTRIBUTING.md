# Contributing to the cTrader plugin

Thanks for your interest in improving the cTrader plugin for AI agents. This guide explains how Spotware publishes this repository, how to report an issue, and how to propose a change. By participating in this project you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md).

## How Spotware Publishes This Repository

Spotware develops the plugin outside this repository and publishes each release here. Each release is one commit on `main`, tagged `v<version>`, and `main` changes only at a release, so `main` always holds the latest release. [CHANGELOG.md](CHANGELOG.md) lists what each release changed.

## Repository Layout

The repository root is the plugin. Each AI agent reads the manifest written for it:

```text
plugin.json                  # Agent Plugins manifest
mcp.json                     # Agent Plugins MCP server configuration
.claude-plugin/plugin.json   # Claude Code plugin manifest
.claude-plugin/marketplace.json
.mcp.json                    # MCP server configuration for Claude Code and Grok Build
.grok-plugin/plugin.json     # Grok Build plugin manifest
gemini-extension.json        # Gemini CLI extension manifest
skills/ctrader-analytics/
  SKILL.md                   # the skill: YAML frontmatter and instructions
  LICENSE.txt                # the license, the same text as the root LICENSE
  references/*.md            # reference files the skill reads when it needs them
assets/                      # the logos
```

The manifests state the same facts for different agents: the plugin name, version and description, and the cTrader MCP server. The plugin contains no scripts and no hooks.

## Reporting Issues

Use one of the [issue templates](.github/ISSUE_TEMPLATE/) when filing an issue:

- **Bug Report**: unexpected behavior of the skill, or a manifest that an agent does not load.
- **Feature Request**: a new capability or an improvement.
- **Documentation Issue**: inaccurate, missing, unclear, or outdated documentation.
- **Question**: how to install or use the plugin. Questions about the **cTrader platform** itself (charts, accounts, broker behavior) or about your cTrader account belong in the [cTrader Help Centre](<https://help.ctrader.com/>).

## Proposing a Change

Open an issue that describes the change and the reason for it. For a change to the skill, quote the passage it concerns or paste the text you propose. For a change to a fact that the manifests state, such as the description, edit one manifest; Spotware applies the change to every manifest that carries the fact.

The plugin reads account and market data and prepares links that you complete in cTrader. A proposal that makes it place, change, close or cancel orders is out of scope.

You can also open a pull request to show the change in full. A release replaces the content of `main`, so Spotware does not merge pull requests. When Spotware accepts a change, it ships the change in the next release, credits the contributor in the changelog entry of that release, and closes the pull request with a link to that release. A pull request that Spotware does not accept is closed with the reason.

## Security

To report a security vulnerability, follow the process in [SECURITY.md](SECURITY.md). Do **not** open a public GitHub issue for security disclosures.
