# Contributor Workflows

Use this reference when adding or changing Rute Bayar providers, CLI behavior, webhook parsing, forwarding, refunds, reconciliation, or persistence.

## Repo Orientation

Important locations:

```text
cmd/rute-bayar/main.go
internal/cli/
internal/daemon/
internal/domain/
internal/provider/
internal/provider/<provider>/
internal/providerfactory/
internal/paymentsvc/
internal/webhooksvc/
internal/forwardingsvc/
internal/storage/sqlite/
docs/
scripts/
.github/workflows/
site/
```

Provider packages are currently under:

```text
internal/provider/xendit/
internal/provider/midtrans/
internal/provider/doku/
```

## Provider Adapter Contract

Every provider should declare and test:

- create payment
- get payment status
- refund payment or explicit unsupported behavior
- verify webhook
- parse webhook event
- map provider status to internal status
- capability declaration

Keep provider-specific request signing, payload shape, webhook signature logic, and raw response parsing inside the provider adapter.

## Add A New Provider

1. Read official provider docs and capture the exact APIs to use.
2. Add `internal/provider/<provider>/adapter.go`.
3. Add capability declarations.
4. Register the provider in `internal/provider/registry.go` and factory wiring.
5. Add onboarding flags and provider test command support.
6. Add pay create and pay status mappings.
7. Add webhook verification and parsing.
8. Add refund/reconcile only when official APIs and credentials are clear.
9. Add SQLite migrations only if existing generic tables are insufficient.
10. Update docs and site changelog when user-facing behavior changes.

## Status Mapping

Map provider-specific statuses into neutral internal statuses. Use current project vocabulary and update `docs/status-mapping.md` if new statuses are introduced.

Common internal statuses include:

```text
pending
paid
settled
failed
expired
cancelled
authorized
captured
refunded
partial_refunded
```

Test ambiguous and edge statuses. If the provider has both authorization and settlement phases, keep the distinction visible unless product semantics require simplification.

## Webhook Handling

Provider webhook flow:

1. Verify signature/token using provider credentials when configured.
2. Persist raw body and raw headers.
3. Deduplicate/idempotently process repeated delivery.
4. Parse into a normalized event.
5. Update payment/refund state.
6. Dispatch forwarding from app/daemon layer, not provider adapter.

Never transform forwarded body for convenience. Forwarding is pass-through.

## JSON Debugging Standard

All provider implementations should preserve:

- outbound request JSON
- outbound response JSON
- inbound webhook body JSON
- inbound webhook headers JSON
- forwarding request JSON
- forwarding response JSON

This is intentional for debugging. Do not censor logs unless a specific task asks for a redaction feature.

## Tests To Add

For provider work, add focused unit tests for:

- request signing and auth headers
- create payment payload mapping
- response parsing and redirect/payment URL extraction
- pay status parsing
- status mapping
- webhook signature/token verification
- webhook event parsing
- unsupported capability errors
- refund/reconcile behavior when implemented

Then run:

```bash
go test ./...
./scripts/smoke-local.sh
```

Run sandbox E2E only with real sandbox credentials:

```bash
./scripts/e2e-sandbox.sh
```

## Docs To Update

Update the smallest docs set that matches the change:

- `docs/provider-integration.md`
- `docs/providers/<provider>-integration.md`
- `docs/cli-onboarding.md`
- `docs/end-to-end-smoke.md`
- `docs/operations-runbook.md`
- `docs/release-readiness.md`
- `README.md`
- `site/src/data/changelog.ts` for user-visible releases

## PR Hygiene

- Use a branch name with `codex/` unless the user asks otherwise.
- Keep PR body non-empty and include verification commands.
- Check review comments with GitHub CLI before merging.
- Resolve review threads only after fixes are committed or the comment is clearly non-actionable.
- Do not self-approve as a substitute for fixing review comments.
