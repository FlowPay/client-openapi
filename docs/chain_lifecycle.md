This endpoint enables the chaining of payments across various documents by automating the payment of one invoice using the amount received from another active invoice. <br> Note: The API for this endpoint does not require specific scopes; the authorization token used must have access to the documents it wants to pay.

The payment chain process works by linking a "ring" document that connects a trigger document to a target document. Once the ring document is created, it establishes a connection between the trigger document, which initiates the payment chain, and the target document, which will receive the payment once the trigger document has been settled through FlowPay.

For example, let's assume there are two documents. The first document involves a payment that Business A must make to Business B, and the second document involves a payment that Business B must make to Business C. To automate these payments, Business B creates a "ring" using its authorization token. This ring links the first document as the "trigger" (activation document) and the second document as the "target" (destination document).

Once the ring is created, checkout sessions can be generated for each document. The checkout for the trigger document proceeds as a standard payment process, while the checkout for the target document is created with a chain type. This chain-type checkout for the target document does not require an immediate payment but awaits the payment of the trigger document.

When Business B logs in and completes the checkout for the target document, no payment is made at that point. However, when the trigger document is paid, it automatically triggers the payment of the target document. If there is any residual amount after the payment of the target document, Business B will receive it.

In this way, the payment automation ensures that all documents involved are settled sequentially, using the funds received from one active document to pay another pending document, minimizing manual intervention and simplifying the cash flow management between the involved businesses.

### Using Payment Chain for Split Payment

In a split payment scenario, the payment chain can be utilized to automatically distribute funds between multiple parties, including retaining a portion of the payment for the partner before the remaining amount is forwarded to the creditor. For example, suppose Business A needs to pay Business B, but a portion of the payment is retained by Business C (acting as a partner or intermediary). Business A would create a trigger document representing the payment it owes to Business B. A ring document is then created, linking this trigger document to two target documents: one for Business B and another for Business C.

When Business A makes the payment, the payment chain ensures that the funds are first routed through Business C. The portion that Business C is entitled to (as agreed upon) is retained, and the remainder of the payment is automatically forwarded to Business B. This setup ensures that the split payment is handled seamlessly, with minimal manual intervention, and all parties receive their due amounts according to the predefined rules of the payment chain.
