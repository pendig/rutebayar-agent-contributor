# Provider Matrix

Use this reference to avoid repeating decisions already made in Rute Bayar.

## Current Providers

| Provider | Create payment | Status | Webhook verify | Refund | Notes |
| --- | --- | --- | --- | --- | --- |
| Xendit | Payment Sessions `POST /sessions` | Supported | `X-Callback-Token` when configured | Implemented | Per-payment webhook URL override is not available for Payment Sessions. Use dashboard webhook URL plus Rute Bayar forwarding. |
| Midtrans | Core API, QRIS/bank transfer style flows | Supported | `order_id + status_code + gross_amount + server_key` signature | Implemented | Some flows support per-transaction notification override via `X-Override-Notification`. |
| DOKU | Checkout `POST /checkout/v1/payment` | Check Status API | HMAC-SHA256 `Signature` header | Disabled | Refund needs Refund API/disbursement setup. Back Office notification URL must match active channels. |

## DOKU Notes

Use Checkout for hosted payment creation. Supported method filters include:

- `checkout`
- `bank_transfer` with bank such as `bca`, `mandiri`, `bri`, `bni`
- `qris`
- `credit_card`
- `ewallet` with bank such as `ovo`, `dana`
- `convenience_store` with `alfa`
- raw DOKU method names such as `VIRTUAL_ACCOUNT_BNI`

DOKU webhook verification uses:

```text
Client-Id
Request-Id
Request-Timestamp
Request-Target
Digest
Signature
```

The HMAC target path matters. Preserve `/webhooks/doku` unless the user configured a custom path during onboarding.

## Xendit Notes

Payment Session create maps:

```text
reference_id <- external reference
session_type = PAY
mode = PAYMENT_LINK
amount, currency, country
items[], customer
```

Keep `items[].category` and `items[].type` compatible with sandbox validation.

## Midtrans Notes

Webhook signature validation uses:

```text
SHA512(order_id + status_code + gross_amount + server_key)
```

For QRIS sandbox proof, the user may need Midtrans simulator interaction. Do not claim full real payment proof unless callback/status evidence exists.

## Future Providers

Planned providers include:

- Flip Business
- Duitku

For new providers, create an issue checklist before broad implementation when official docs or account capabilities are unclear.
