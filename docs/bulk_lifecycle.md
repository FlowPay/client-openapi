You can allow users to pay multiple payees in a single operation using the additionalPayees parameter when creating a payment request.

<p align="center">
<img src="https://mermaid.ink/svg/pako:eNqFk8GOmzAQhl8FTQ97YSPAARwOlcK2PVXVKhu1UsXFwUPWKtipMauwUS59nj7VPkkNJA10Nyone-abf_6xzQFyxRESqA0z-EGwrWZVJh37caExN0JJ5_NqiKQosRC5YLpdOre3750l56IjWHnPWsQ6cXzPe_n1-xWeXsHDN-m7K3QQvkVfsxJf6H-TfcVqfT9k7aIPrJddi4uj9aD8zZ6DbWeT_khymkwno0xzdyfj4MJWCw6J0Q26UKGuWLeFQ1eVgXnECjNI7HLDartyR_GvTAu2KbHugMPQJoNCSfOJVaJsh7qbldooo25c5wk1Z5K5TldXnrTOJQ_i-dTIj3b7UXKnRdWdvyqVHoB3nCMp8tdMqjRHPSbnEcmLYkQy-3qeWHfu6Y_tmCyCgk40R-T_ZU8G1rg3Y873iDfHEVfjzwZljl-aajOVPM_UkcdMHu3N7Jj8rlR1vhytmu0jJAUra7trdvzyc_yNapS91UYaSBbeoheB5AB7SPyYzOKAkJBGIfUjErrQ2igls4Xvk8DO5BEa0vDownPf1pvN4zgMojiK6ZxGlMYusMaoh1bmZ1ODjY_2KSt9cnH8A6xMMAI" width="50%">
</p>

The above diagram shows how bulk works.
We assume the case in which a user wants to pay Beneficiary A for 100€, Beneficiary B for 50€ and Beneficiary C for 25€ and again Beneficiary A for 75€.
The additionalPayees parameter will be populated like this:

```json
{
  "additionalPayees": [
    {
      "iban": "IT79Q0300203280941591243326",
      "name": "Beneficiary A",
      "amount": 100
    },
    {
      "iban": "IT79Q0300203280941591243327",
      "name": "Beneficiary B",
      "amount": 50
    },
    {
      "iban": "IT79Q0300203280941591243328",
      "name": "Beneficiary C",
      "amount": 25
    },
    {
      "iban": "IT79Q0300203280941591243326",
      "name": "Beneficiary A",
      "amount": 75
    }
  ]
}
```

The payment request created will have an amount of 250€, payer can now pay it with a single payments.
The wire transfer allowed with the PIS on a bulk payment is addressed to the FlowPay technical account (TA).
When the TA receives the payment, it splits the amount among the beneficiaries, groups them by their IBANs and names, and sends the payments to them with wire transfers.

In case of a bulk payment to the same beneficiary, the payment initiation effective beneficiary is the beneficiary itself, so the payer can easily recognize the transaction. Otherwise, the payment initiation effective beneficiary is FlowPay.
Wire transfers to beneficiaries are sent with the same original payer, so beneficiaries can easily identify the payer.

<div class="info">
 <div class="title">In case of pagoPA payments</div>
    The bulk service is not natively supported with pagoPA payment, please contact us for more information and workarounds.
</div>
