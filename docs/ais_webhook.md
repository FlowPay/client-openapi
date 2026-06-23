# AIS consent webhooks

FlowPay sends AIS consent webhooks to the `callbackUrl` configured on the consent session. A webhook is queued only after FlowPay persists a local consent status transition with a matching event type.

Events:

- `consent.authorized`: the consent was finalized successfully.
- `consent.denied`: the consent flow failed or was denied.
- `consent.canceled`: a pending or initiated consent was canceled.
- `consent.revoked`: an authorized consent was explicitly revoked.
- `consent.expired`: an authorized consent is no longer valid according to the banking connector.

No webhook is emitted for `pending` or `initiated`.

The `consent.authorized` event is normally queued when the user returns from the SCA flow and FlowPay receives the redirect result. If that redirect is missed, FlowPay can still discover an already-authorized consent during a later consent detail or AIS data-read call, then persist `authorized` and queue the same event.

Delivery is retried when the partner endpoint returns a non-2xx status or the HTTP request fails. Return any 2xx response to acknowledge the webhook. Retries use exponential backoff with jitter and preserve the same `eventId` across attempts; use `X-FlowPay-Event-Id` for idempotency.

Each request is signed with Ed25519. Verify `X-FlowPay-Signature` against the exact raw request body using:

```text
X-FlowPay-Timestamp + "." + rawBody
```

Fetch the public key identified by `X-FlowPay-Key-Id` from `/.well-known/jwks.json`.
