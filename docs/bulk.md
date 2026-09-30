You can allow users to pay multiple payees in a single operation using the additionalPayees parameter when creating a payment request.

### Mixed bulk with locked movements

Each `additionalPayees` item may optionally include its own `lockedUntil`. That movement is then held on a dedicated technical wallet and exposes its own locked payment method after authorization. Items without `lockedUntil` follow the normal bulk flow. The top-level `lockedUntil`, when present, applies only to the main (residual) movement and is not propagated to additional payees.

Locked movements are supported for bulk payment requests only. Split payment requests (`ultimateDebtor: payee`) cannot contain a top-level or per-movement `lockedUntil`.

The response identifies every movement with its `id`. Ordinary movements expose the generated wallet-payment identifier; for locked movements the `id` is the child payment-request identifier. Locked movements additionally expose `lockedUntil`, `status`, and `lockedPaymentMethod` when available. Movements are independent: releasing one locked movement does not change the others.

If the aggregate root has already been created and provisioning of a locked child fails, the API returns `409 Conflict` and retains the resources already created. The response exposes the retained aggregate through the `Location` and `X-FlowPay-Payment-Request-Id` headers. Inspect that aggregate before recovery and do not blindly recreate the request.

[![](https://mermaid.ink/img/pako:eNqFk81um0AQx18FTQ-5kGgxsGAOkUzanqoqcqxWqris2cFZFXbdZYnsWL70efpUeZIuH66hiRVOszO_-c9_PzhArjhCArVhBj8KttGsyqRjPy405kYo6XxZ9pkUJRYiF0zvF8719a2z4Fy0BCvv2R6xThyPkJfff17h6QU8fJO-u0DPwrfoS1aiM_1_setYru77qg26xGrRjjg7WvXK3-052HG26I0kp8V0spVp7W4wDi5stOCQGN2gCxXqirVLOLRdGZhHrDCDxIZrVtvIHeW_MS3YusS6BQ79mAwKJc1nVoly3_ddLdVaGXXlOk-oOZPMddq-ctA6tTyI52GQR7e7UXGrRdWevyqV7oEPnKNf5K-ZVGmOekwG1M-LYkQy-3qeWHvu6c_NmCxmRTzRHJHvyw4GVrgzY84jPglwxNX4q0GZ49emWk8lT3tqyWMmj_Zmtkz-UKo6XY5WzeYRkoKVtV01W37-Of5lNcrOaiMNJPNo1olAcoAdJF4Y3ZCQ0HlMI0r9yPNd2EMyC2KbpkEc0LmtUhIeXXju5pKbICC2jYRhHAbRPPKpC6wx6mEv85Ot3sgn-5iVHnwc_wL-ZjBd?type=png)](https://mermaid.live/edit#pako:eNqFk81um0AQx18FTQ-5kGgxsGAOkUzanqoqcqxWqris2cFZFXbdZYnsWL70efpUeZIuH66hiRVOszO_-c9_PzhArjhCArVhBj8KttGsyqRjPy405kYo6XxZ9pkUJRYiF0zvF8719a2z4Fy0BCvv2R6xThyPkJfff17h6QU8fJO-u0DPwrfoS1aiM_1_setYru77qg26xGrRjjg7WvXK3-052HG26I0kp8V0spVp7W4wDi5stOCQGN2gCxXqirVLOLRdGZhHrDCDxIZrVtvIHeW_MS3YusS6BQ79mAwKJc1nVoly3_ddLdVaGXXlOk-oOZPMddq-ctA6tTyI52GQR7e7UXGrRdWevyqV7oEPnKNf5K-ZVGmOekwG1M-LYkQy-3qeWHvu6c_NmCxmRTzRHJHvyw4GVrgzY84jPglwxNX4q0GZ49emWk8lT3tqyWMmj_Zmtkz-UKo6XY5WzeYRkoKVtV01W37-Of5lNcrOaiMNJPNo1olAcoAdJF4Y3ZCQ0HlMI0r9yPNd2EMyC2KbpkEc0LmtUhIeXXju5pKbICC2jYRhHAbRPPKpC6wx6mEv85Ot3sgn-5iVHnwc_wL-ZjBd)

The above diagram shows how bulk works.
We assume the case in which a user wants to pay Beneficiary A for 100€, Beneficiary B for 50€ and Beneficiary C for 25€ and again Beneficiary A for 75€.
The additionalPayees parameter will be populated like this. You can specify a per-payee `remittanceInformation`; for locked movements, when it is omitted, the main request `remittanceInformation` is inherited by that locked movement. Ordinary movements retain the existing bulk remittance behavior.

```json
{
  "additionalPayees": [
    {
      "iban": "IT79Q0300203280941591243326",
      "name": "Beneficiary A",
      "amount": 100,
      "remittanceInformation": "Order A-100"
    },
    {
      "iban": "IT79Q0300203280941591243327",
      "name": "Beneficiary B",
      "amount": 50
    },
    {
      "iban": "IT79Q0300203280941591243328",
      "name": "Beneficiary C",
      "amount": 25,
      "remittanceInformation": "Service fee C"
    },
    {
      "iban": "IT79Q0300203280941591243326",
      "name": "Beneficiary A",
      "amount": 75
    }
  ]
}
```

The payment request created will have an amount of 250€; the payer can now pay it with a single payment.
The wire transfer allowed with the PIS on a bulk payment is addressed to the FlowPay technical account (TA).
When the TA receives and reconciles the payment, ordinary movements follow the normal bulk dispatch flow to their beneficiaries. Locked movements are instead funded into their own dedicated locked wallets and remain there until the partner releases or refunds them, or until FlowPay automatically releases them at their individual `lockedUntil` deadlines.

In case of a bulk payment to the same beneficiary, the payment initiation effective beneficiary is the beneficiary itself, so the payer can easily recognize the transaction. Otherwise, the payment initiation effective beneficiary is FlowPay.
Wire transfers to beneficiaries are sent with the same original payer, so beneficiaries can easily identify the payer.

<div class="info">
 <div class="title">In case of PagoPA payments</div>
    The bulk service is not natively supported with PagoPA payments; please contact us for more information and workarounds.
</div>
