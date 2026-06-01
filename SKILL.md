---
name: rutebayar-agent-contributor
description: Operate and extend Rute Bayar, the Go CLI and webhook daemon for Indonesian payment gateways. Use when setting up sandbox or production Rute Bayar flows, helping AI Agents create invoices and verify payments through the rutebayar CLI, testing webhooks/forwarding/reconciliation, preparing releases, or contributing a new provider adapter such as DOKU, Flip Business, Duitku, Xendit, or Midtrans.
---

# Rute Bayar Agent & Contributor

Use this skill for Rute Bayar work where correctness depends on the project's CLI, daemon, provider model, sandbox/prod differences, and contribution conventions.

## Start Here

1. Confirm the command name is `rutebayar`, not `rute-bayar`.
2. Read the local repo first when available: `README.md`, `docs/provider-integration.md`, `docs/operations-runbook.md`, and provider-specific docs under `docs/providers/`.
3. Keep credentials out of commits, PR bodies, issue comments, screenshots, and logs shown to public channels.
4. Prefer `lean-ctx -c "<command>"` for shell exploration when available.
5. For current provider behavior, verify against official provider docs before changing adapter logic.

## Choose The Workflow

- **Sandbox/prod operation, AI Agent billing, webhook, forwarding, health checks**: read `references/operator-workflows.md`.
- **Adding or modifying providers**: read `references/contributor-workflows.md`.
- **Provider capability caveats and current integration matrix**: read `references/provider-matrix.md`.
- **CI, E2E, PR, release, and Homebrew distribution**: read `references/release-ci.md`.

## Operating Rules

- Store raw outbound request/response JSON and raw inbound webhook body/headers for debuggability.
- Keep webhook forwarding pass-through: preserve provider body and headers as much as possible.
- Verify webhook authenticity when credentials support it; if verification is skipped, state why.
- Use reconciliation as the source-of-truth fallback when webhooks are late, missing, duplicated, or non-final.
- Treat SQLite as the current persistence target; recommend backups before production upgrades.
- Default to small, reviewable PRs with docs and tests updated together.

## Contribution Rules

- Keep provider-specific logic in adapter packages and provider docs; do not leak provider quirks into core domain or CLI contracts unless the public interface needs it.
- Add provider capabilities explicitly and make unsupported operations fail clearly.
- Normalize provider statuses into internal statuses and test edge cases.
- Add or update unit tests for adapter signing, payload mapping, webhook verification, webhook parsing, status mapping, forwarding, and service-level behavior.
- Update docs, site changelog/data, and release notes when user-facing behavior changes.

## Minimum Verification

Before saying a change is ready:

```bash
go test ./...
./scripts/smoke-local.sh
```

For site changes:

```bash
npm --prefix site run build
```

For provider sandbox validation, use `scripts/e2e-sandbox.sh` only when sandbox credentials are present in environment variables or GitHub Secrets.
