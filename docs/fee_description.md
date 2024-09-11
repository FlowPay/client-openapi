# Fee structure

FlowPay provides a fully automated fee management system that simplifies the process for partners, eliminating the need to integrate specific APIs for fee collection.

Fees calculation rules are determined during the initial system setup, tailored to the specific contractual requirements of each partner. These fees can be charged either to the creditor or the debtor, depending on the case. When the creditor is responsible for the fee, a split payment is automatically generated. Conversely, if the debtor bears the cost, a bulk payment is triggered.

Each rule is defined for a specific payment method (pis, sdd, card) and type of document. Additionally, minimum amounts can be established, below which fees are not applied, this allows the creation of exceptions to the general rule or progressive fee structures.
Rules also determine which party will be responsible for the fees, ensuring a clear and transparent process.
In cases where documents involve multiple due dates subject to fees, the system does not allow partial payments tied to individual deadlines. Instead, the user is required to make a single full payment, in line with the overall terms of the document.

It is also possible to specify whether the fee is calculated as a percentage of the total amount of the document to be paid or based on a fixed amount per unit.

# Lifecycle

a
When the user selects the payment method to be used, the fee calculation is done automatically by the payment gateway retrieving the most up-to-date rule applicable to that specific payment. Once the calculation is performed, the gateway automatically inserts the fee document into an aggregated document (Bulk).

In accordance with the documentation related to bulk payments, if a user accesses a checkout linked to a document that has subsequently been included in a Bulk document, the payment gateway will automatically retrieve the final Bulk document. This ensures that the payment process is always aligned with the most recent information, simplifying the management of fees and their application to multiple payments.
