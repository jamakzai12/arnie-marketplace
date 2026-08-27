# Arnie CLI and agent skills

Arnie is a GTM workspace where tables are workflows. The public package gives
coding agents one normal command-line tool plus 24 skills for building and
operating prospecting, enrichment, research, and outbound workflows.

The CLI keeps the transport hidden. Users sign in through the browser; they do
not copy an API key or configure MCP.

## Requirements

- Node.js 20 or newer
- An Arnie account at [arnies.ai](https://www.arnies.ai)

## Install the CLI and skills

Install the CLI package first:

```bash
npm install --global github:jamakzai12/arnie-marketplace
arnie --version
```

Then register the same 24 skills with the coding agent you use.

Codex:

```bash
codex plugin marketplace add jamakzai12/arnie-marketplace --ref main
codex plugin add arnie@arnie
```

Claude Code:

```bash
claude plugin marketplace add jamakzai12/arnie-marketplace
claude plugin install arnie@arnie
```

Start a new Codex task or restart Claude Code so the skills are loaded.

Only after the CLI and skills are installed, sign in and prove the account:

```bash
arnie login
arnie whoami
arnie tools
```

Normal calls use:

```bash
arnie call <tool> '<json>'
```

The plugin contains skills only. Codex and Claude Code use the same `arnie`
CLI and browser sign-in.

## Update

```bash
npm install --global github:jamakzai12/arnie-marketplace
codex plugin marketplace upgrade arnie
codex plugin add arnie@arnie
```

For Claude Code:

```bash
claude plugin marketplace update arnie
claude plugin update arnie@arnie
```

## Included skills

The package ships all 24 Arnie skills: the four core operating skills, workflow
execution and authoring skills, prospecting motions, list-quality checks,
personalization, and outcome analysis. Both clients load the same skill files.

## Support

- Product: [arnies.ai](https://www.arnies.ai)
- Public package: [github.com/jamakzai12/arnie-marketplace](https://github.com/jamakzai12/arnie-marketplace)
- Email: hello@arnies.ai

Copyright Arnie. This distribution is proprietary and is provided for use with
the Arnie service.
