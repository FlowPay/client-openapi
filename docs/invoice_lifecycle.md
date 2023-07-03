# Invoice lifecycle
This endpoint allows you to create, update, and retrieve invoices. It also allows you to manage each step of the invoice lifecycle, from draft to get paid.

## Invoice 
An invoice is a commercial document that itemizes a transaction between a buyer and a seller. It serves as a request for payment from the seller to the buyer, indicating the products or services provided, their quantities, prices, and any applicable taxes or discounts. Invoices typically include payment terms, such as due dates and accepted payment methods.
Only legal entities can issue invoices.

In order to work seamlessly with real customers' invoice management, we provide also API to manage proforma and credit notes.

## Proforma invoice
A proforma invoice is a preliminary invoice that is issued before the completion of a transaction. It provides an estimate of the costs and terms of a potential sale, allowing the buyer to review and approve them before proceeding. Proforma invoices are often used in international trade to facilitate customs procedures, as they provide a detailed breakdown of the expected costs, including shipping charges, taxes, and duties. They are also used to provide a quote for potential buyers or to seek approval for a budgeted purchase.
In common practice, payments are often collected against proforma invoices and then the final invoice is issued, FlowPay supports this scenario.
Creating a checkout for a proforma invoice is allowed, but it's necessary to provide the final invoice details when it's issued and link it to the original proforma invoice.
Linking a proforma invoice to a paid invoice is also supported, this can be useful to keep track of the entire invoicing process.

When linked, the invoice replaces previously created proforma checkout and the payment status of each document is updated accordingly.

## Credit note
A credit note, also known as a credit memo or credit memo invoice, is a document issued by a seller to a buyer to indicate a reduction in the amount owed. It is essentially a negative invoice that reflects a credit or refund due to various reasons, such as product returns, billing errors, or pricing adjustments. Credit notes help maintain accurate accounting records and allow for the reconciliation of accounts between buyers and sellers.
You can also create a checkout for a credit note to refund the customer, but it's necessary to provide the original invoice and link it to the original invoice. 
Linking a credit note to an invoice will automatically update the invoice status and amount due.