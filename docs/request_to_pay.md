# Functional Overview

This chapter explains how each Request To Pay (RTP) product works functionally: actors, lifecycle, routing, and constraints. Use it to understand behavior before jumping into payloads.

## Simple RTP

Simple RTP is the shortest path from a payment request to a successful bank transfer. Your backend creates a request that describes what the user is paying (title, description, remittance), who receives the funds (payee), and optional URLs for redirect and server‑to‑server callbacks. You receive a hosted checkout link where the user completes Strong Customer Authentication (SCA) with their bank. From the user’s perspective it’s a clean, guided flow; from your side you track status via callbacks and finalise your order by reading the request status from the API.

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIGF1dG9udW1iZXJcbiAgcGFydGljaXBhbnQgQXBwIGFzIFlvdXIgQmFja2VuZFxuICBwYXJ0aWNpcGFudCBBUEkgYXMgRmxvd1BheSBBUElcbiAgcGFydGljaXBhbnQgVXNlciBhcyBQYXllclxuICBwYXJ0aWNpcGFudCBCYW5rIGFzIEJhbmsgKFNDQSlcbiAgQXBwLT4-QVBJOiBQT1NUIC9wYXltZW50LXJlcXVlc3RzXG4gIEFQSS0tPj5BcHA6IDIwMSB7IHJlcXVlc3RJZCwgbGluayB9XG4gIEFwcC0tPj5Vc2VyOiBSZWRpcmVjdCB0byBsaW5rIChIb3N0ZWQgQ2hlY2tvdXQpXG4gIFVzZXItPj5CYW5rOiBTQ0EgYW5kIGNvbnNlbnRcbiAgQmFuay0tPj5BUEk6IEF1dGhvcmlzYXRpb24gb3V0Y29tZVxuICBBUEktLT4-VXNlcjogU3VjY2Vzcy9FcnJvciBzY3JlZW5cbiAgQVBJLS0-PkFwcDogUE9TVCBjYWxsYmFja1VybCAoc3RhdHVzKVxuICBBcHAtPj5BUEk6IEdFVCAvcGF5bWVudC1yZXF1ZXN0cy97aWR9IChjb25maXJtKVxuIn0=)

State evolution: a Simple RTP typically flows created → inProgress when the user starts checkout, then authorized on a successful SCA with the provider. Settlement goes straight to the payee (fast transition to forwarded); it passes through the Technical Account (onHold) only when the ultimateDebtor differs from the debtor (payer). User cancellations, OTP expirations, or bank denials keep the request non-terminal until a successful attempt occurs or the request expires. Refunds, when issued, move the request to refunded for the affected amount.

What it enables: one‑off payments, invoice settlement, donations, deposits, and any flow where a user authorises a bank transfer to a known beneficiary.

How to enable specific flavors (copy‑paste ready):

### Fixed amount (default)

Set `amount` to the exact value you want to charge.

State evolution: most flows are linear. From created the first user attempt moves to inProgress; on a positive outcome the session is authorized and the request proceeds to forwarded. Settlement goes straight to the payee; the Technical Account (onHold) path applies only when ultimateDebtor ≠ debtor. Errors or cancellations do not change the aggregate state to terminal; a later successful attempt completes the flow. Post-settlement reversals are reflected as refunded.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"; API_KEY="sk_test_xxx"
curl -sS -X POST "$BASE_URL/payment-requests" -H "Content-Type: application/json" -H "X-API-Key: $API_KEY" -d '{
  "payer": "+39 333 1234567",
  "title": "Invoice #1001",
  "description": "Settlement for invoice 1001",
  "remittanceInformation": "INV-1001",
  "amount": 49.90,
  "currency": "EUR",
  "redirectUrl": "https://merchant.example.com/return",
  "callbackUrl": "https://merchant.example.com/api/payment/callback"
}'
```

### Open amount (donations or pay‑what‑you‑want)

Omit `amount` entirely to let the payer enter the amount at checkout. Keep `allowPartialPayments` at its default (`false`). Enforce your business rules (min/max) when processing callbacks or before fulfilling the order.

State evolution: identical to fixed-amount, with the value chosen during checkout. Expect multiple inProgress attempts as the payer can re-enter and adjust the amount; only attempts that reach authorized affect the financial outcome. Settlement behaves like Simple RTP (direct to payee; Technical Account hop only if ultimateDebtor ≠ debtor); any reversal is reflected as refunded.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"; API_KEY="sk_test_xxx"
curl -sS -X POST "$BASE_URL/payment-requests" -H "Content-Type: application/json" -H "X-API-Key: $API_KEY" -d '{
  "payer": "+39 333 1234567",
  "title": "Donate to ACME Foundation",
  "description": "Thank you for your support",
  "remittanceInformation": "DON-2024-09",
  "currency": "EUR",
  "redirectUrl": "https://merchant.example.com/thank-you",
  "callbackUrl": "https://merchant.example.com/api/payment/callback"
}'
```

Optional: allow the payer to edit the remittance at checkout by setting `allowRemittanceChange: true`.

### Partial settlement (multiple payments until total is reached)

Set `allowPartialPayments: true` and provide the target `amount` to be fully settled across one or more sessions.

State evolution: each successful session independently reaches authorized and then forwarded for its portion. The request remains non-terminal until the cumulative authorized amount meets the target; it can cycle back to inProgress for additional attempts. Temporary onHold/locked can appear per session when funds pause on the Technical Account. Once the target is covered, the request is effectively complete in forwarded. Refunds operate per session without reopening the request.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"; API_KEY="sk_test_xxx"
curl -sS -X POST "$BASE_URL/payment-requests" -H "Content-Type: application/json" -H "X-API-Key: $API_KEY" -d '{
  "payer": "+39 333 1234567",
  "title": "Installments for Order #A2001",
  "description": "Pay in multiple steps",
  "remittanceInformation": "A2001",
  "amount": 300.00,
  "currency": "EUR",
  "allowPartialPayments": true,
  "redirectUrl": "https://merchant.example.com/return",
  "callbackUrl": "https://merchant.example.com/api/payment/callback"
}'
```

To compute the outstanding: list sessions with `/payment-requests/{requestId}/sessions`, sum `succeeded` amounts, and subtract from the target `amount`.

### Scheduled execution (future date)

Set `executionDate` (ISO 8601). The user authenticates now; execution occurs at/after the scheduled time (bank support dependent).

State evolution: after inProgress and authorized, the request remains pending execution until the scheduled time. You will not see forwarded until the bank executes. If execution fails or is revoked, the provider emits a negative outcome and the request converges to rejected. Refunds are available only after execution.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"; API_KEY="sk_test_xxx"
EXEC_AT="2025-01-10T10:00:00Z" # choose your date/time (UTC)
curl -sS -X POST "$BASE_URL/payment-requests" -H "Content-Type: application/json" -H "X-API-Key: $API_KEY" -d '{
  "payer": "+39 333 1234567",
  "title": "Scheduled payment",
  "description": "Executes later",
  "remittanceInformation": "SCH-10JAN",
  "amount": 120.00,
  "currency": "EUR",
  "executionDate": "'"$EXEC_AT"'",
  "redirectUrl": "https://merchant.example.com/return",
  "callbackUrl": "https://merchant.example.com/api/payment/callback"
}'
```

## Bulk payments

Bulk turns many payouts into a single user authorisation. You compute a total amount and provide the allocation list with additional payees. The user performs SCA once; FlowPay collects the total and then splits it to the designated beneficiaries.

![](https://mermaid.ink/img/pako:eNpVUlFr2zAQ_iuHQtnDnBDHcWJrMHBSNgZrGCV9WbyHm3xORGXJSDKrm-a_T3ZoadGLTt_dd9-nuzMTpiLGWa3MP3FC6-HnfakBHg4Pjixg50_GSkcOvPGoABvTaf8HptOvsC8O30LZL-xhT-KkpQgJhRBjxkCyL4a8l-_WdC387eHHpth91tjQCxSHDWmqpZBoeyjepcPmA7R5D20_QNsADeDNDdyRbVBW8GbDQWX0Jw-ua1sTXEmtpCbQxpOLwBlwsukUegLpB47d4Z4a6T1qQfBI1DoIvo9SB0st9uEramOB0PVgSRgtpJLopdGjvB3nfKB-lToLWndXcUKhc7dUj62hlkrxSZ0PJ3LemkfiE0SMhFHG8kmSJFFttJ86-Uw8XrRPX1jEjlZWjHvbUcSaq9MwsvNAXzJ_ooZKxsO1oho75UtW6ksoa1H_NqZ5rQxTOJ4Yr1G5EHVtFdzfSjxabN5eLemK7HaYIONxPnIwfmZPIYrz2TzLsyzOksUyXqWLiPWMr1ezfJWu8zyfp3G-yJeXiD2PXeezNEuTdLWOF8kyTubZKmJUSW_s3XXnxtW7_AeTHNGA)

What it enables: mass invoice runs, bill aggregation, marketplace or platform settlements with a single frictionless checkout. Repeated beneficiaries in your allocation are grouped automatically. Keep `allowPartialPayments` disabled; if the sum of additional payees is less than the total, the remainder goes to the primary payee.

State evolution: authorization is single from the user’s perspective; settlement is mediated. After authorized, funds always land on the Technical Account, are processed immediately, and re-sent to beneficiaries. The request may appear onHold briefly while dispatch allocates amounts, then moves to forwarded. Subsequent reversals are reflected as refunded and are applied proportionally to the original allocation map.

## Locked payment

When outcomes depend on a later verification (delivery, inspection, return window, milestone approval), use Locked payments. You specify a `lockedUntil` date; after a successful checkout the funds are held on FlowPay’s technical perimeter and attributed to a dedicated technical wallet for that operation. Before the date, the partner may release the funds to the beneficiary or refund the payer. If no earlier decision is taken, FlowPay automatically releases the funds to the beneficiary at `lockedUntil`.

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIGF1dG9udW1iZXJcbiAgcGFydGljaXBhbnQgVXNlciBhcyBVc2VyXG4gIHBhcnRpY2lwYW50IEFQSSBhcyBGbG93UGF5IEFQSVxuICBwYXJ0aWNpcGFudCBBcHAgYXMgWW91ciBCYWNrb2ZmaWNlXG4gIEFwcC0-PkFQSTogUE9TVCAvcGF5bWVudC1yZXF1ZXN0cyAobG9ja2VkVW50aWwpXG4gIEFQSS0tPj5BcHA6IDIwMSB7IHJlcXVlc3RJZCwgbGluayB9XG4gIEFwcC0tPj5Vc2VyOiBSZWRpcmVjdCB0byBob3N0ZWQgY2hlY2tvdXRcbiAgVXNlci0-PkFQSTogQXV0aG9yaXplIGluaXRpYWwgcGF5bWVudFxuICBBUEktLT4-QXBwOiBDYWxsYmFjayAoYXV0aG9yaXplZCwgbG9ja2VkKVxuICBOb3RlIG92ZXIgQVBJOiBGdW5kcyBoZWxkIG9uIGRlZGljYXRlZCB0ZWNobmljYWwgd2FsbGV0IHVudGlsIGxvY2tlZFVudGlsXG4gIGFsdCBFYXJseSByZWxlYXNlXG4gICAgQXBwLT4-QVBJOiBQT1NUIC9jaGFyZ2VzIHdpdGggbG9ja2VkIHRva2VuSWRcbiAgICBBUEktLT4-QXBwOiBDYWxsYmFjayAoZm9yd2FyZGVkKVxuICBlbHNlIFJlZnVuZFxuICAgIEFwcC0-PkFQSTogUE9TVCAvcmVmdW5kc1xuICAgIEFQSS0tPj5BcHA6IENhbGxiYWNrIChyZWZ1bmRlZClcbiAgZWxzZSBObyBhY3Rpb24gYnkgZXhwaXJ5XG4gICAgQVBJLS0-PkFwcDogQ2FsbGJhY2sgKGZvcndhcmRlZCBhdCBsb2NrZWRVbnRpbClcbiAgZW5kIn0)

What it enables: escrow‑like experiences, milestone‑based projects, dispute/return windows, and any flow where the partner needs immediate authorization but delayed availability to the beneficiary.

State evolution: after a successful checkout the session is authorized, may pass briefly through `onHold` while funds are reconciled, and then enters `locked`. From that state the partner may release early to the beneficiary or refund the payer. If no explicit action is taken, expiry leads to `forwarded` through automatic release. Additional user attempts do not alter funds that are already locked.

## Split payment

Split payment directs portions of a single checkout to different parties — for example, retaining a platform fee while paying a vendor. You define a primary payee and one or more additional payees with explicit amounts. The platform collects once and allocates the proceeds accordingly, preserving the original payer identity for beneficiaries.

![](https://mermaid.ink/img/pako:eNp9UtFq2zAU_ZXLhcIGbha7sRMrMCjr-tQMk_WpcR4U6zoWkS1PlmmyJK-Dve4b9mX9ksl2Wygbe9LV1Tk6uufoiJkWhAxzpR-zghsLd8u0AkhW99pyBdelbiu7hsvLj6cllVxWgswJFq5YJUaW3Bwg4Qeidc_qcZ_3tZKZtMB78glu_dUtEXwYkOD_FxusBlTgUB3u4gIWZJyyAEPfWmmoAVsQVNoSWA0bgoZqbrglAbnRpTsRBIJyWUkrddX0aoyxjnFdZYU2XeeL2y3ltrCrZP18eE97O4hmijfNDeXw0oZcKuVAFXmNNXpHQ51ppQ3bKJ7t5n_xBq1_M-evw93xDSmwncajtMUwV1Pwmt68sncrxa9t-Y4L0Q_GVe9U8x6efv52Vri85s6j55Dg6ccvqIeIUjxBgh5ujRTIrGnJw3Iw1WV_7IRSdKaWlCJzpeBml2JanR2n5tWD1uULzeh2WyDLuWrcrq2F8_1G8q3h5WvXUPeAT12gyIIo7C9BdsQ9Mt-PR-NZPJv5s6tg4kdh4OEB2TQaxVE4jeN4HPpxEE_OHn7vZcejcBZehdHU94NJ5Naph-QM0GYxfN3-B5__ANaT8N0?theme=default)

What it enables: marketplaces, app stores, franchise models, partner revenue sharing with full transparency in remittances.

State evolution: similar to Bulk, the user authorizes once and funds always pass via the Technical Account, are processed immediately, and re-forwarded according to the split. The request may show a short onHold during dispatch, then advances to forwarded. Errors on outbound legs are handled by the dispatch engine and do not return the request to inProgress. Refunds mirror the original allocation proportions.

## Payment Chain

Payment Chain lets a partner create an RTP whose execution depends on funds expected later. The chain subject is the RTP `payer` and, when specified, the RTP `debtor`; the payout destination is defined by `payee` and, for split or bulk scenarios, `additionalPayees`.

![](https://mermaid.ink/img/c2VxdWVuY2VEaWFncmFtCiAgYXV0b251bWJlcgogIHBhcnRpY2lwYW50IFBhcnRuZXIgYXMgUGFydG5lciBiYWNrZW5kCiAgcGFydGljaXBhbnQgQVBJIGFzIEZsb3dQYXkgQVBJCiAgcGFydGljaXBhbnQgQ2hhaW4gYXMgQ2hhaW4gdGVjaG5pY2FsIHBvc2l0aW9uCiAgcGFydGljaXBhbnQgRnVuZGluZyBhcyBGdW5kaW5nIEZsb3dQYXkgcGF5bWVudHMKICBwYXJ0aWNpcGFudCBGaW5hbCBhcyBGaW5hbCBwYXllZShzKQogIFBhcnRuZXItPj5BUEk6IFBPU1QgL3BheW1lbnQtcmVxdWVzdHMgKHBheWVyL2RlYnRvciA9IGNoYWluIHN1YmplY3QsIHBheW1lbnRNZXRob2Q6IGNoYWluKQogIE5vdGUgb3ZlciBQYXJ0bmVyLEFQSTogcGF5ZWUgYW5kIGFkZGl0aW9uYWxQYXllZXMgZGVmaW5lIHRoZSBmaW5hbCBzaW5nbGUsIHNwbGl0LCBvciBidWxrIHBheW1lbnQKICBBUEktLT4-UGFydG5lcjogMjAxIHsgcmVxdWVzdElkLCBsaW5rL3N0YXR1cyB9CiAgUGFydG5lci0-PkFQSTogQ29tcGxldGUgYXV0aG9yaXphdGlvbiAvIFNDQSBmb3IgY2hhaW4gbWV0aG9kCiAgQVBJLS0-PlBhcnRuZXI6IHBheW1lbnQuc3RhdHVzX2NoYW5nZSAoYXV0aG9yaXplZCkKICBBUEktPj5DaGFpbjogUHJlcGFyZSBkZWRpY2F0ZWQgdGVjaG5pY2FsIHBvc2l0aW9uCiAgbG9vcCBBZG1pdHRlZCBmdW5kaW5nIG9wZXJhdGlvbnMKICAgIEZ1bmRpbmctPj5BUEk6IENvbXBsZXRlIEZsb3dQYXkgcGF5bWVudCBsaW5rZWQgdG8gcmVxdWVzdAogICAgQVBJLT4-Q2hhaW46IFJlY29uY2lsZSBpbmNvbWluZyBmdW5kcwogIGVuZAogIGFsdCBSZWNvbmNpbGVkIGJhbGFuY2UgPj0gcmVxdWVzdCBhbW91bnQKICAgIEFQSS0-PkZpbmFsOiBFeGVjdXRlIFJUUCBwYXlvdXQgdXNpbmcgcGF5ZWUvYWRkaXRpb25hbFBheWVlcwogICAgQVBJLS0-PlBhcnRuZXI6IHBheW1lbnQuc3RhdHVzX2NoYW5nZSAoZm9yd2FyZGVkKQogIGVsc2UgRnVuZGluZyB3aW5kb3cgZXhwaXJlcyBiZWxvdyBhbW91bnQKICAgIEFQSS0-PkZpbmFsOiBMaXF1aWRhdGUgYXZhaWxhYmxlIGZ1bmRzIHRvIHBheWVyL2RlYnRvciB2ZXJpZmllZCBJQkFOCiAgICBBUEktLT4-UGFydG5lcjogcGF5bWVudC5zdGF0dXNfY2hhbmdlIChmb3J3YXJkZWQgb3IgcmVqZWN0ZWQpCiAgZW5kCg)

What it enables: chained settlement, supplier or platform payouts funded by expected incoming FlowPay payments, convergence of multiple funding operations into one controlled technical position, and automatic execution of the RTP after the request amount is funded.

State evolution: the chain is created with the standard Request To Pay API and `paymentMethod` set to `chain`. After authorization it moves to `authorized`; once admitted funds arrive it uses `onHold` while FlowPay reconciles balance, amount, deadline, excesses, and operational conditions. When the reconciled balance reaches the RTP `amount`, FlowPay forwards the RTP payout to `payee` and `additionalPayees`. If the amount is not reached within the operational window, available funds are liquidated to the payer/debtor or the request is rejected if no valid liquidation path exists.

Detailed examples, lifecycle diagrams, and callback handling are documented in [Payment Chain](./payment_chain.md).

## PagoPA payment

For public‑service payments in the Italian PagoPA ecosystem, RTP integrates a dedicated branch. You provide the entity tax code and payment notice number, and an email to receive the receipt. The hosted checkout guides the user through the PagoPA‑specific steps, while your integration pattern (redirects, callbacks, receipt download) remains the same.

![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIGF1dG9udW1iZXJcbiAgcGFydGljaXBhbnQgQXBwIGFzIFlvdXIgQmFja2VuZFxuICBwYXJ0aWNpcGFudCBBUEkgYXMgRmxvd1BheSBBUEkgKFBhZ29QQSlcbiAgcGFydGljaXBhbnQgVXNlciBhcyBQYXllclxuICBBcHAtPj5BUEk6IFBPU1QgL3BheW1lbnQtcmVxdWVzdHMgKFBhZ29QQSBmaWVsZHMpXG4gIEFQSS0tPj5BcHA6IDIwMSB7IHJlcXVlc3RJZCwgbGluayB9XG4gIEFwcC0tPj5Vc2VyOiBSZWRpcmVjdCB0byBsaW5rXG4gIFVzZXItPj5BUEk6IFBhZ29QQSBmbG93IGF0IGNoZWNrb3V0XG4gIEFQSS0tPj5Vc2VyOiBSZWNlaXB0IHNjcmVlbiArIGVtYWlsXG4gIEFQSS0tPj5BcHA6IENhbGxiYWNrICsgUERGIGF2YWlsYWJsZSB2aWEgR0VUXG4ifQ==)

What it enables: PagoPA notice payments with consistent checkout and server‑side integration semantics. Bulk isn’t natively supported with PagoPA; contact FlowPay for options.

State evolution: follows PagoPA’s own flow as defined in the Simplified Flow document. The request goes created → inProgress during the PagoPA journey, then to authorized on a positive PagoPA outcome, and to forwarded once remittance to the Creditor Entity completes. Invalid notices or cancellations converge to rejected. No Technical Account hop is used unless explicitly mandated by PagoPA reconciliation policies.

### How to enable PagoPA payment

Provide the PagoPA‑specific fields (`pagopaEcFiscalCode`, `pagopaPaymentNotice`, `email`) alongside your standard request data.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"; API_KEY="sk_test_xxx"
curl -sS -X POST "$BASE_URL/payment-requests" -H "Content-Type: application/json" -H "X-API-Key: $API_KEY" -d '{
  "payer": "+39 333 1234567",
  "title": "PagoPA payment",
  "description": "PagoPA notice 1234567890",
  "remittanceInformation": "PPA-123",
  "amount": 49.90,
  "currency": "EUR",
  "redirectUrl": "https://merchant.example.com/return",
  "callbackUrl": "https://merchant.example.com/api/payment/callback",
  "pagopaEcFiscalCode": "01234567890",
  "pagopaPaymentNotice": "123456789012345678",
  "email": "payer@example.com"
}'
```

## Payment State Machine

This section explains the lifecycle of a payment request as defined in the Simplified Flow document. The payment goes through well-defined states from creation to completion or termination.

- created: request created and visible to the user.
- inProgress: user starts a session and authenticates with the provider.
- authorized: provider confirms a positive outcome for the operation.
- rejected: provider rejects/cancels the operation.
- onHold: funds are on the FlowPay technical account (TA) awaiting dispatch rules.
- locked: funds have been reconciled on the dedicated technical wallet and are waiting for partner release, refund, or automatic release at `lockedUntil`.
- forwarded: funds dispatched to the beneficiaries, including automatic release at `lockedUntil` for locked payments.
- refunded: refund executed for the transaction; in locked flows this is the explicit alternative final outcome to release.
- deleted: request removed by the partner (only if not in progress and not paid).

Final states are: deleted, forwarded, rejected, refunded.

![](https://mermaid.ink/svg/pako:eNp1VNuO0zAQ_RXLT4DaqkmTtpsHpNVeBBIItCteoDy49iQdiO3Kdgrbav9n_4MfY-I07XYpeYqP58w5c-xkx6VVwAvugwhwjaJyQg836cIwer69-c6Gw7fsygHtqg7cL-LGe_PZ2cqB9wX7EsAEYGKzQcE8QWgNdJRjWWRdNmFlHW5BFYw2NqjAMWlNCU4LthaV0NTKnuXewQ-Q4YTpsMQmnBA76lEnUj-Zd7Ym4q01CplvWslgWQBpUO7VuppY_sHKn63OZQjgaaJlbaW0TLVCjgZVUMc3A67jdoTIvbXul3AKjmoa24SRCWRLMFCiROGwd_qP-LMG97Bnx9mYaILV1EkexjzU7gMqGxOJd6iX1nlLc-p1DX2gL1I5BkpCSCP1sQpjmroWEeuTZb3m80twDdS8bXBTo0Yjtu3Bn82JvfLAjDUMDZkia6_PWjodvq3_80SnLFcInjyshfeiqtD-5wz3fmIrusEd2I_5Eu3COkVPAyWYD3jlUPEiuAYGXLf3tF3yXUtY8LACDQte0KuCUjR1WPCFeSTaWpiv1uqe6WxTrXhRitrTqlmr40d3QB2QI3dlGxN4Mc3z2IQXO_6bF2mWjLJkkubZbDxL0nwy4A-8SNLZKJtPkvGM8It0nmSPA76NsuPRPE3G83x6kWR5Ok6n2YC3N-j-wcjeVGfjRmGw7uAC4vJj93OI_4jHv6hVXpY)

# Examples

This chapter collects hands‑on Request To Pay (RTP) examples and the technical explanation of how each use case works end‑to‑end. It covers actors, data fields, settlement paths, and status lifecycles so you can reason about behavior beyond the JSON payloads.

## Simple RTP — example

- Actors: payer (user initiating payment), payee (final beneficiary), optional debtor (if different from payer).
- Flow: your backend creates a payment request and receives a hosted `link`. The user completes SCA with their bank at checkout. FlowPay sends `callbackUrl` events and redirects the user to `redirectUrl` with `?status`.
- Security: authenticate API calls with `X-API-Key`.
- Settlement: funds are transferred from the payer’s bank to the configured payee IBAN. If no payee is provided, the partner’s default configuration is used.
- Status: the request follows the Payment State Machine (`created` → `inProgress` → `authorized` → `onHold`/`locked` → `forwarded` or terminal `rejected`/`refunded`/`deleted`). Inspect attempts with `/payment-requests/{requestId}/sessions` (session statuses: `succeeded`, `pending`, `expired`, `failed`, `cancelled`).

Inline example (Sandbox):

1. Create a payment request and get `requestId` + `link`.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"
API_KEY="sk_test_xxx" # replace with yours

curl -sS -X POST "$BASE_URL/payment-requests" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "payer": "+39 333 1234567",
    "title": "Order #A1001",
    "description": "Payment for order A1001",
    "remittanceInformation": "A1001",
    "amount": 49.90,
    "currency": "EUR",
    "redirectUrl": "https://merchant.example.com/checkout/return",
    "callbackUrl": "https://merchant.example.com/api/payment/callback"
  }'
```

2. Redirect the customer to the returned `link` to complete payment.

Verify and download receipt:

```bash
REQUEST_ID="..."

curl -sS -X GET "$BASE_URL/payment-requests/$REQUEST_ID" -H "X-API-Key: $API_KEY"

curl -sS -X GET \
  -H "X-API-Key: $API_KEY" \
  -H "Accept: application/pdf" \
  "$BASE_URL/payment-requests/$REQUEST_ID" \
  -o receipt.pdf
```

3. Treat the `callbackUrl` as the primary source of truth; use the APIs for on‑demand confirmation.

## Split payment (Simplified Flow) — example

- Goal: distribute a single user payment across multiple beneficiaries (e.g., platform fee + vendor payout).
- Data model: set total `amount`, primary `payee`, and an array of `additionalPayees` with their individual `amount`s. The remainder flows to the primary payee.
- Settlement: the payer authorises once; funds are collected and then allocated. When multiple beneficiaries are involved, settlement routes via FlowPay’s Technical Account and fans out to beneficiaries (grouped by IBAN+name), preserving the original payer in remittance.
- Constraints: Sum(additionalPayees.amount) ≤ `amount`. Do not combine `additionalPayees` with `allowPartialPayments=true`.

### How to enable Split payment

Set the total `amount`, define a primary `payee`, and list `additionalPayees` with their explicit `amount`s. Keep `allowPartialPayments` set to `false`. Ensure the sum of all `additionalPayees.amount` is less than or equal to the total; the remainder goes to the primary `payee`.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"
API_KEY="sk_test_xxx"

curl -sS -X POST "$BASE_URL/payment-requests" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "payer": "+39 333 1234567",
    "title": "Order #SP-1001",
    "description": "Split payment",
    "remittanceInformation": "SP-1001",
    "amount": 100.00,
    "currency": "EUR",
    "redirectUrl": "https://merchant.example.com/return",
    "callbackUrl": "https://merchant.example.com/api/payment/callback",
    "payee": { "name": "ACME Vendor", "iban": "IT60X0542811101000000123456" },
    "additionalPayees": [
      { "amount": 5.00, "name": "Platform Fee", "iban": "IT60X0542811101000000654321" }
    ],
    "allowPartialPayments": false
  }'
```

Notes:

- Use `remittanceInformation` to correlate the whole order across all beneficiaries.
- Refunds operate per successful session; maintain your allocation map for proportional refunds if needed.

## Locked payment — example

- Goal: escrow‑like behavior with optional early release and deterministic automatic release at expiry.
- Data model: add `lockedUntil` (ISO 8601) at creation time. The beneficiary must be identified before authorization; final payout coordinates may already be known at creation time or completed later before release.
- Lifecycle: after successful collection, funds are reconciled and the request enters `locked`. Before expiry you may retrieve the dedicated locked payment method and release the funds through `POST /charges`, or refund the payer through `POST /refunds`. If no explicit action is taken, FlowPay automatically releases the funds to the beneficiary at `lockedUntil`.

### How to enable Locked payment

Add `lockedUntil` (ISO 8601). After reconciliation, use `GET /payment-requests/{requestId}` to retrieve `lockedDetails.lockedPaymentMethod.id`. The webhook payload exposes the same technical method as `payload.paymentMethod.id`. Before expiry, either release the funds with `POST /charges` or refund the payer with `POST /refunds`.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"
API_KEY="sk_test_xxx"
LOCK_UNTIL=$(date -u -v+3d +"%Y-%m-%dT%H:%M:%SZ" 2>/dev/null || date -u -d "+3 days" +"%Y-%m-%dT%H:%M:%SZ")

curl -sS -X POST "$BASE_URL/payment-requests" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "payer": "+39 333 1234567",
    "title": "Escrow order #L-2001",
    "description": "Locked until verification",
    "remittanceInformation": "L-2001",
    "amount": 59.00,
    "currency": "EUR",
    "redirectUrl": "https://merchant.example.com/return",
    "callbackUrl": "https://merchant.example.com/api/payment/callback",
    "payee": { "name": "ACME Vendor", "iban": "IT60X0542811101000000123456" },
    "lockedUntil": "'"$LOCK_UNTIL"'"
  }'
```

Early release example:

```bash
REQUEST_ID="..." # from create response or callback

LOCKED_TOKEN_ID=$(curl -sS -X GET "$BASE_URL/payment-requests/$REQUEST_ID" \
  -H "X-API-Key: $API_KEY" | jq -r '.lockedDetails.lockedPaymentMethod.id')

curl -sS -X POST "$BASE_URL/charges" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "tokenId": "'"$LOCKED_TOKEN_ID"'",
    "payee": {
      "name": "ACME Vendor",
      "iban": "IT60X0542811101000000123456"
    },
    "remittanceInformation": "L-2001 RELEASE"
  }'
```

Refund example (alternative final outcome):

```bash
SESSION_ID="..." # locked collection session id from sessions or callback

curl -sS -X POST "$BASE_URL/refunds" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "sessionId": "'"$SESSION_ID"'",
    "amount": 59.00,
    "reason": "other"
  }'
```

Notes:

- Locked payments are all-or-nothing: the release amount is implicit in the locked token and matches the original locked amount; refunds are documented for the full amount only.
- If the final payee coordinates were not already available, provide them in the `payee` field of the locked release payload before expiry.
- Treat `callbackUrl` as the primary orchestration signal and use `GET /payment-requests/{requestId}` as the recovery path if a webhook is missed.

## Bulk payments — example

- Goal: let a user pay many targets with a single SCA (single checkout), reducing friction.
- Data model: same allocation primitives as split (`additionalPayees`) but used for larger lists; total `amount` equals the sum of all components intended for beneficiaries.
- Settlement: FlowPay’s Technical Account receives the total, then issues outgoing wires by allocation, grouping repeated entries (same IBAN+name). The original payer is preserved in remittance data to ease reconciliation.

### How to enable Bulk payments

Compute the total `amount` as the sum of all components, set the primary `payee`, and provide the allocation list in `additionalPayees`. Keep `allowPartialPayments` set to `false`. Repeated beneficiaries are allowed; FlowPay groups them by IBAN+name during settlement.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"
API_KEY="sk_test_xxx"

curl -sS -X POST "$BASE_URL/payment-requests" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "payer": "+39 333 1234567",
    "title": "Bulk order #B-9001",
    "description": "Bulk payout to A,B,C",
    "remittanceInformation": "B-9001",
    "amount": 250.00,
    "currency": "EUR",
    "redirectUrl": "https://merchant.example.com/return",
    "callbackUrl": "https://merchant.example.com/api/payment/callback",
    "payee": { "name": "Beneficiary A", "iban": "IT79Q0300203280941591243326" },
    "additionalPayees": [
      { "amount": 50, "name": "Beneficiary B", "iban": "IT79Q0300203280941591243327" },
      { "amount": 25, "name": "Beneficiary C", "iban": "IT79Q0300203280941591243328" },
      { "amount": 75, "name": "Beneficiary A", "iban": "IT79Q0300203280941591243326" }
    ],
    "allowPartialPayments": false
  }'
```

Notes:

- Combine with attachments for richer payer context (e.g., PDF invoice).
- Rely on `callbackUrl` + `GET /payment-requests/{id}` for authoritative status.

## Attachments and PDF receipt — example

- Two kinds of attachments: `attachments` (visible to payer at checkout) and `privateAttachments` (compliance‑only, not shown to payer).
- Upload options: multipart/form‑data (preferred) or JSON with base64 data.
- Receipt: `GET /payment-requests/{requestId}` with `Accept: application/pdf` returns a PDF with details and a QR for device handoff; if paid, it acts as a receipt.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"
API_KEY="sk_test_xxx"

# Upload (multipart)
FILE_ID=$(curl -sS -X POST "$BASE_URL/files" -H "X-API-Key: $API_KEY" -F "file=@./invoice.pdf;type=application/pdf" | jq -r .id)

# Create RTP with attachments
curl -sS -X POST "$BASE_URL/payment-requests" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "payer": "+39 333 1234567",
    "title": "Order #A1002",
    "description": "Payment with attachment",
    "remittanceInformation": "A1002",
    "amount": 129.00,
    "currency": "EUR",
    "redirectUrl": "https://merchant.example.com/checkout/return",
    "callbackUrl": "https://merchant.example.com/api/payment/callback",
    "attachments": ["'"$FILE_ID"'"]
  }'

# Download receipt (PDF)
REQUEST_ID="..."
curl -sS -X GET -H "X-API-Key: $API_KEY" -H "Accept: application/pdf" "$BASE_URL/payment-requests/$REQUEST_ID" -o receipt.pdf
```
