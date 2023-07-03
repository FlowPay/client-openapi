This endpoint enables bulk payments on documents that could be of different types. <br> Note: API of this endpoint does not require specific scopes, the authorization token used must have access to documents it wants to bulk pay

<p align="center">
<img src="https://mermaid.ink/svg/pako:eNqFkl9rwjAUxb9KyB58qWCryOjDwO4PCGOIle2lL7fNrQs2iUtT0YnffYmtGjdhebo553dPEm72tFAMaUxrAwafOCw1iEwSuxjXWBiuJHmdt8pUbhQvMExQhv3-A0maanXlRM4hNy3XFP2yZrBUs4kTL5qrjthsmnbUND0Ki0m7Xxw7yIe9njvulhjRgArUAjizT9s7JKPmEwVmNLZlDrWtAk9_B80hr7B2wL7NzGippHkBwatd29ebq1wZ1QvIBjUDCQFxfVWXdWpJ-Xd3UDhebz1zrbkAvXtUldItcMcYDsviL5MozVD75Gg8LMrSI8FOZwNuQslq6ZNlVN5fZXrk_7HdBRa4NT4XDoaDEXpcjV8NygLfGpFfR57e5MhDJg92GtAYle5kQWOjGwxos2aX_0bjEqr6rD4zbpTuxMMPPNfQrg" width="50%">
</p>

The above diagram shows how bulk works.
We assume the case in which a user has two invoices for beneficiary 1 (Ben1), one invoice for beneficiary 2 (Ben2), and one PagoPA payment
The user wants to pay all of them in a single operation.

The user's client, which has access to all the documents, calls the bulk endpoint with the list of documents to pay, the endpoint returns a new document (Bulk) that is linked to all the documents to pay.
The client gives the bulk document to the user, which can now pay it with a single Payment Initiation (PIS) operation. The wire transfer allowed with the PIS on the bulk document is addressed to the FlowPay technical account (TA).
When the TA receives the payment, it splits the amount among the beneficiaries and sends the payments to them with wire transfers.

In case of a bulk payment to the same beneficiary, the payment initiation effective beneficiary is the beneficiary itself, so the payer can easily recognize the transaction. Otherwise, the payment initiation effective beneficiary is FlowPay.
Wire transfers to beneficiaries are sent with the same original payer, so beneficiaries can easily identify the payer.

Note: pagoPA payments recipient is FlowPay itself, so related payments are not counted in the split operations.