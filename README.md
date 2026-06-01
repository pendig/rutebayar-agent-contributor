# Rute Bayar AI Agent & Contributor Skill

[![skills.sh](https://skills.sh/b/pendig/rutebayar-agent-contributor)](https://skills.sh/pendig/rutebayar-agent-contributor)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

AI Agent skill for operating and contributing to [Rute Bayar](https://github.com/pendig/rute-bayar), an open source payment router CLI and webhook daemon for Indonesian payment gateways.

This skill helps AI agents, coding assistants, automation agents, and human contributors work with Rute Bayar consistently across sandbox, production, webhook forwarding, release, and provider integration workflows.

## What This Skill Helps With

- Operate Rute Bayar in sandbox or production.
- Use `rutebayar` as a safe billing tool boundary for AI Agent workflows.
- Create payment links or invoices and verify payment status.
- Set up webhook daemon, health checks, replay, and pass-through forwarding.
- Add or maintain payment provider adapters such as Xendit, Midtrans, DOKU, Flip Business, and Duitku.
- Prepare PRs, CI checks, E2E sandbox validation, releases, and Homebrew distribution.

## Install

Install with the skills.sh CLI:

```bash
npx skills add pendig/rutebayar-agent-contributor
```

The skills.sh CLI can install the skill into supported AI agent environments. Restart or reload your agent after installing if it does not pick up new skills automatically.

Manual install for agents that read local `SKILL.md` folders:

```bash
mkdir -p ~/.agent-skills/rutebayar-agent-contributor
cp -R SKILL.md agents references ~/.agent-skills/rutebayar-agent-contributor/
```

For Codex specifically, the local skill directory is usually:

```bash
~/.codex/skills/rutebayar-agent-contributor
```

## Usage

Ask your agent to use the skill:

```text
Use $rutebayar-agent-contributor to set up Rute Bayar sandbox webhook testing with Xendit forwarding.
```

```text
Use $rutebayar-agent-contributor to add a new provider adapter for Flip Business and prepare the PR checklist.
```

```text
Use $rutebayar-agent-contributor to review whether a Rute Bayar release is ready.
```

## Skill Layout

```text
SKILL.md
agents/openai.yaml
references/operator-workflows.md
references/contributor-workflows.md
references/provider-matrix.md
references/release-ci.md
```

## Safety Notes

- Do not commit sandbox or production provider credentials.
- Do not paste API keys, webhook tokens, or dashboard secrets into public issues or PRs.
- Verify provider behavior against official provider documentation before changing adapter logic.
- Use sandbox until the user explicitly confirms production.

## Related Project

- Rute Bayar: https://github.com/pendig/rute-bayar
- Website: https://rutebayar.id
- skills.sh repo page: https://skills.sh/pendig/rutebayar-agent-contributor
- skills.sh skill page: https://skills.sh/pendig/rutebayar-agent-contributor/rutebayar-agent-contributor

## License

MIT License. Copyright (c) 2026 Wahyu Adi Putra, Pena Digital.
