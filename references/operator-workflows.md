# Operator Workflows

Use this reference for sandbox, production, webhook, forwarding, and AI Agent billing operations.

## Command Baseline

The public binary is `rutebayar`.

```bash
rutebayar --help
rutebayar db migrate
rutebayar provider list
rutebayar provider accounts
```

Install options:

```bash
brew tap pendig/tap
brew install rutebayar
curl -fsSL https://raw.githubusercontent.com/pendig/rute-bayar/main/scripts/install.sh | bash
go install github.com/pendig/rute-bayar/cmd/rute-bayar@latest
```

If Go install creates `rute-bayar`, rename or symlink it to `rutebayar`.

## Sandbox Setup

Use sandbox for real provider API tests without production money movement.

```bash
export RUTE_BAYAR_ENV=sandbox
export RUTE_BAYAR_DB_PATH=./rute-bayar.sqlite3
rutebayar db migrate
```

Onboard providers:

```bash
rutebayar onboard xendit \
  --secret-key "$XENDIT_SECRET_KEY" \
  --webhook-token "$XENDIT_WEBHOOK_TOKEN" \
  --environment sandbox

rutebayar onboard midtrans \
  --merchant-id "$MIDTRANS_MERCHANT_ID" \
  --client-key "$MIDTRANS_CLIENT_KEY" \
  --server-key "$MIDTRANS_SERVER_KEY" \
  --environment sandbox

rutebayar onboard doku \
  --client-id "$DOKU_CLIENT_ID" \
  --secret-key "$DOKU_SECRET_KEY" \
  --webhook-path /webhooks/doku \
  --environment sandbox
```

Test provider connectivity:

```bash
rutebayar provider test xendit --environment sandbox
rutebayar provider test midtrans --environment sandbox
rutebayar provider test doku --environment sandbox
```

For DOKU, a dummy check-status probe may return `404` while still proving signed auth reaches DOKU correctly.

## Payment Creation And Status

Use unique references. In CI or repeated sandbox tests, include UTC timestamp plus run id to avoid provider duplicate-reference errors.

```bash
rutebayar pay create \
  --provider xendit \
  --method payment_link \
  --reference rb-xendit-$(date -u +%Y%m%d%H%M%S) \
  --amount 15000

rutebayar pay status --provider xendit --reference <reference>
```

Midtrans supports per-transaction notification override for applicable methods:

```bash
rutebayar pay create \
  --provider midtrans \
  --method qris \
  --bank gopay \
  --reference rb-midtrans-qris-001 \
  --amount 15000 \
  --notification-url https://<public-domain>/webhooks/midtrans
```

DOKU Checkout supports notification URL override, but DOKU Back Office channel settings must still be compatible:

```bash
rutebayar pay create \
  --provider doku \
  --method checkout \
  --reference rb-doku-001 \
  --amount 15000 \
  --notification-url https://<public-domain>/webhooks/doku

rutebayar pay status --provider doku --reference rb-doku-001
```

Xendit Payment Sessions do not support per-payment webhook URL override. Configure the global webhook URL in Xendit Dashboard and use Rute Bayar forwarding for application-specific delivery.

## Webhook Daemon

Run locally:

```bash
rutebayar webhook serve --addr :8080 --environment sandbox
curl -i http://localhost:8080/healthz
```

Endpoints:

```text
/webhooks/xendit
/webhooks/midtrans
/webhooks/doku
```

For public callback testing:

```bash
wrangler tunnel quick-start http://localhost:8080
curl -i https://<trycloudflare-domain>/healthz
```

Set provider dashboards to:

```text
https://<public-domain>/webhooks/xendit
https://<public-domain>/webhooks/midtrans
https://<public-domain>/webhooks/doku
```

## Forwarding

Forwarding is pass-through and managed by CLI. Use it when Rute Bayar should receive, verify, persist, and optionally retry webhooks before sending the raw provider callback to another app.

```bash
rutebayar webhook forward list
rutebayar webhook forward add --provider xendit --target-url https://example.com/webhooks/payments
rutebayar webhook forward attempts list --limit 20
rutebayar webhook forward attempts show <attempt_id>
rutebayar webhook forward attempts retry <attempt_id>
```

If forwarding fails:

- Check daemon logs.
- Check the target URL accepts the provider body and headers.
- Check event filters and enabled state.
- Replay from persisted webhook events when needed.

## Replay And Diagnostics

Webhook events persist raw payloads and headers.

```bash
rutebayar webhook replay --event-id <id> --db ./rute-bayar.sqlite3
rutebayar webhook replay --provider midtrans --event-id <id>
```

Useful SQLite checks:

```bash
sqlite3 ./rute-bayar.sqlite3 \
  "SELECT id, provider_id, event_type, processing_status, signature_valid, received_at FROM webhook_events ORDER BY received_at DESC LIMIT 20;"
```

## Production Deployment

Recommended production shape:

```text
Internet -> Reverse proxy/TLS -> rutebayar webhook daemon -> SQLite
```

Use a locked-down runtime user and separate paths:

```text
/etc/rute-bayar/rute-bayar.env
/var/lib/rute-bayar/rute-bayar.sqlite3
/var/log/rute-bayar/
```

Production checks:

- HTTPS public endpoint exists.
- `/healthz` returns success.
- SQLite path is not web-accessible.
- Provider credentials and webhook tokens match production dashboards.
- Backups run before upgrades and regularly during production traffic.
- Logs are access-controlled because raw payloads are intentionally retained.

## AI Agent Billing Pattern

Rute Bayar is useful as a bounded billing tool for AI Agents:

```text
AI Agent computes billable product/usage
AI Agent calls rutebayar pay create with provider, reference, amount
User pays through hosted provider flow
Provider sends webhook to Rute Bayar
Rute Bayar verifies, stores raw JSON, reconciles if needed
Rute Bayar forwards event to product or AI orchestration layer
AI Agent checks rutebayar pay status or receives forwarded event
```

Agent guardrails:

- Use deterministic references that include product, tenant, and run id.
- Do not print secrets in conversation or PRs.
- Prefer sandbox until the user explicitly confirms production.
- Check `pay status` before granting entitlement if webhook state is unclear.
- Use forwarding for orchestration callbacks instead of provider-specific webhook handling in each agent.
