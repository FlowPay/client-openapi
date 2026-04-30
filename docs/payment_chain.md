# Payment Chain

Payment Chain lets a partner create an RTP whose execution depends on funds expected later. The chain subject is the RTP `payer` and, when specified, the RTP `debtor`; the payout destination is defined by `payee` and, for split or bulk scenarios, `additionalPayees`.

The payer/debtor authorizes the payment up front. FlowPay executes the RTP only after admitted funding operations have credited and reconciled the dedicated technical position for that request.

Use Payment Chain when an intermediate beneficiary expects to receive funds through FlowPay and wants to route those funds automatically to one or more final beneficiaries.

## Actors

- Chain subject: the RTP `payer` and actual `debtor`; this is the intermediate beneficiary whose funds are expected and who authorizes the payment.
- Funding operations: admitted FlowPay payments linked to the request and reconciled on the dedicated technical position.
- Final beneficiaries: the RTP `payee` and any `additionalPayees`.
- Partner: creates the RTP, links funding operations to it, and owns the business logic.
- FlowPay: manages the technical position, reconciliation, states, callbacks, and final payment execution.

## Core rules

- The partner must be enabled for Payment Chain during onboarding.
- The request is created through the standard Request To Pay API with `paymentMethod.technology` set to `chain`.
- The RTP `amount` is the target amount that must be funded.
- The RTP `payee` and `additionalPayees` define the final payment to execute.
- The dedicated IBAN, when exposed, is internal to FlowPay and can only receive admitted FlowPay funding operations.
- The execution trigger fires when the reconciled balance is greater than or equal to the RTP `amount`.
- Excess funds are liquidated to the payer/debtor on their verified IBAN.
- If the target is not reached within the operational window, available funds are liquidated to the payer/debtor.
- The request can be cancelled without financial impact only before the first admitted credit.
- After final forwarding or closure, late credits do not reactivate the request.

## Sequence

![](https://mermaid.ink/img/c2VxdWVuY2VEaWFncmFtCiAgYXV0b251bWJlcgogIHBhcnRpY2lwYW50IFBhcnRuZXIgYXMgUGFydG5lciBiYWNrZW5kCiAgcGFydGljaXBhbnQgQVBJIGFzIEZsb3dQYXkgQVBJCiAgcGFydGljaXBhbnQgQ2hhaW4gYXMgQ2hhaW4gdGVjaG5pY2FsIHBvc2l0aW9uCiAgcGFydGljaXBhbnQgRnVuZGluZyBhcyBGdW5kaW5nIEZsb3dQYXkgcGF5bWVudHMKICBwYXJ0aWNpcGFudCBGaW5hbCBhcyBGaW5hbCBwYXllZShzKQogIFBhcnRuZXItPj5BUEk6IFBPU1QgL3BheW1lbnQtcmVxdWVzdHMgKHBheWVyL2RlYnRvciA9IGNoYWluIHN1YmplY3QsIHBheW1lbnRNZXRob2Q6IGNoYWluKQogIE5vdGUgb3ZlciBQYXJ0bmVyLEFQSTogcGF5ZWUgYW5kIGFkZGl0aW9uYWxQYXllZXMgZGVmaW5lIHRoZSBmaW5hbCBzaW5nbGUsIHNwbGl0LCBvciBidWxrIHBheW1lbnQKICBBUEktLT4-UGFydG5lcjogMjAxIHsgcmVxdWVzdElkLCBsaW5rL3N0YXR1cyB9CiAgUGFydG5lci0-PkFQSTogQ29tcGxldGUgYXV0aG9yaXphdGlvbiAvIFNDQSBmb3IgY2hhaW4gbWV0aG9kCiAgQVBJLS0-PlBhcnRuZXI6IHBheW1lbnQuc3RhdHVzX2NoYW5nZSAoYXV0aG9yaXplZCkKICBBUEktPj5DaGFpbjogUHJlcGFyZSBkZWRpY2F0ZWQgdGVjaG5pY2FsIHBvc2l0aW9uCiAgbG9vcCBBZG1pdHRlZCBmdW5kaW5nIG9wZXJhdGlvbnMKICAgIEZ1bmRpbmctPj5BUEk6IENvbXBsZXRlIEZsb3dQYXkgcGF5bWVudCBsaW5rZWQgdG8gcmVxdWVzdAogICAgQVBJLT4-Q2hhaW46IFJlY29uY2lsZSBpbmNvbWluZyBmdW5kcwogIGVuZAogIGFsdCBSZWNvbmNpbGVkIGJhbGFuY2UgPj0gcmVxdWVzdCBhbW91bnQKICAgIEFQSS0-PkZpbmFsOiBFeGVjdXRlIFJUUCBwYXlvdXQgdXNpbmcgcGF5ZWUvYWRkaXRpb25hbFBheWVlcwogICAgQVBJLS0-PlBhcnRuZXI6IHBheW1lbnQuc3RhdHVzX2NoYW5nZSAoZm9yd2FyZGVkKQogIGVsc2UgRnVuZGluZyB3aW5kb3cgZXhwaXJlcyBiZWxvdyBhbW91bnQKICAgIEFQSS0-PkZpbmFsOiBMaXF1aWRhdGUgYXZhaWxhYmxlIGZ1bmRzIHRvIHBheWVyL2RlYnRvciB2ZXJpZmllZCBJQkFOCiAgICBBUEktLT4-UGFydG5lcjogcGF5bWVudC5zdGF0dXNfY2hhbmdlIChmb3J3YXJkZWQgb3IgcmVqZWN0ZWQpCiAgZW5kCg)

## Public lifecycle

Payment Chain reuses the existing payment request states. The public lifecycle remains focused on the request status; funding reconciliation and balance checks are handled by FlowPay while the request is waiting for execution.

- `created`: the RTP has been created. No funds have been reconciled yet.
- `authorized`: the chain payment method is authorized and the technical position can receive admitted funding.
- `onHold`: funds are present or pending against final execution. Balance, deadline, and exceptions are monitored here.
- `forwarded`: the RTP payout has been forwarded to `payee` and `additionalPayees`, or default liquidation has been executed.
- `rejected`: the request is refused, cancelled before funding, or cannot be executed.

![](https://mermaid.ink/img/c3RhdGVEaWFncmFtLXYyCiAgWypdIC0tPiBjcmVhdGVkCiAgY3JlYXRlZCAtLT4gYXV0aG9yaXplZDogY2hhaW4gbWV0aG9kIGF1dGhvcml6ZWQgLyBTQ0EgY29sbGVjdGVkCiAgY3JlYXRlZCAtLT4gcmVqZWN0ZWQ6IGNhbmNlbGxlZCBiZWZvcmUgZnVuZGluZwogIGF1dGhvcml6ZWQgLS0-IG9uSG9sZDogZmlyc3QgYWRtaXR0ZWQgZnVuZHMgcmVjb25jaWxlZAogIG9uSG9sZCAtLT4gb25Ib2xkOiBiYWxhbmNlIHVwZGF0ZWQgLyBiZWxvdyBSVFAgYW1vdW50CiAgb25Ib2xkIC0tPiBmb3J3YXJkZWQ6IGJhbGFuY2UgPj0gUlRQIGFtb3VudCBhbmQgUlRQIHBheW91dCBzZW50CiAgb25Ib2xkIC0tPiBmb3J3YXJkZWQ6IGZ1bmRpbmcgd2luZG93IGV4cGlyZXMgYW5kIGRlZmF1bHQgbGlxdWlkYXRpb24gc2VudAogIG9uSG9sZCAtLT4gcmVqZWN0ZWQ6IG5vdCBleGVjdXRhYmxlIC8gbm8gdmFsaWQgbGlxdWlkYXRpb24gcGF0aAogIGZvcndhcmRlZCAtLT4gWypdCiAgcmVqZWN0ZWQgLS0-IFsqXQo)

## Example: chain with a single final payment

In this example, the payer/debtor is the chain subject. The final payment is the RTP payout to `payee`.

```bash
BASE_URL="https://api.sandbox.flowpay.it/v2/"
API_KEY="sk_test_xxx"

curl -sS -X POST "$BASE_URL/payment-requests" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -H "Idempotency-Key: chain-order-9001" \
  -d '{
    "payer": "4a1d7c8e-3b2f-4a40-a74f-13df37a1d230",
    "debtor": "4a1d7c8e-3b2f-4a40-a74f-13df37a1d230",
    "payee": {
      "name": "Supplier SRL",
      "iban": "IT60X0542811101000000123456"
    },
    "title": "Payment chain for order #9001",
    "description": "Execute supplier payout when funding is received",
    "remittanceInformation": "CHAIN-9001",
    "amount": 100.00,
    "currency": "EUR",
    "paymentMethod": {
      "technology": "chain"
    },
    "callbackUrl": "https://merchant.example.com/api/payment/callback"
  }'
```

Representative response:

```json
{
  "paymentRequestId": "3bb35be2-9f2d-4217-b41e-e83b89312d8c",
  "status": "created",
  "amount": 100.00,
  "currency": "EUR",
  "payer": "4a1d7c8e-3b2f-4a40-a74f-13df37a1d230",
  "payeeId": "8d73f75d-583d-4a81-aea2-c97dfccfb331",
  "paymentMethod": {
    "technology": "chain"
  }
}
```

## Example: chain with split or bulk payout

A split or bulk chain uses the same payout fields as the corresponding RTP. The final distribution is expressed with `payee` and `additionalPayees`.

```json
{
  "payer": "4a1d7c8e-3b2f-4a40-a74f-13df37a1d230",
  "debtor": "4a1d7c8e-3b2f-4a40-a74f-13df37a1d230",
  "amount": 250.00,
  "currency": "EUR",
  "remittanceInformation": "CHAIN-9001",
  "paymentMethod": {
    "technology": "chain"
  },
  "payee": {
    "name": "Supplier SRL",
    "iban": "IT60X0542811101000000123456"
  },
  "additionalPayees": [
    {
      "name": "Platform SPA",
      "iban": "IT79Q0300203280941591243327",
      "amount": 50.00,
      "remittanceInformation": "Platform fee #9001"
    }
  ]
}
```

In this example, FlowPay executes the RTP when the chain is funded. The supplier receives the residual amount (`amount` minus `additionalPayees`), and the additional payee receives the configured amount.

## Funding

Funding must come from FlowPay operations enabled for the use case. FlowPay links admitted funding operations to the request and reconciles them on the dedicated technical position.

When the reconciled amount reaches or exceeds the RTP `amount`, FlowPay schedules or sends the payout defined by the RTP.

If the last funding operation exceeds the RTP amount, the difference is treated as excess and liquidated to the payer/debtor on their verified IBAN.

## Callbacks and status reads

Use the configured `callbackUrl` exactly as for other Request To Pay products. The stable callback event remains `payment.status_change`; the `status` field carries the public payment request status. Always treat the API read model as authoritative, and make webhook handlers idempotent.
