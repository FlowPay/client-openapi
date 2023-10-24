This endpoint enables chaining payments of various documents by automating the payment of an incoming invoice using the amount received from an active invoice <br> Note: API of this endpoint does not require specific scopes, the authorization token used must have access to documents it wants to pay.

<p align="center">

</p>


The ring document connects trigger to target
Onece you've created the ring, you can create a checkout on the ring document for the target. That checkout session will enable the payment automation to start after the trigger document is payed through FlowPay.
Example:
Step 1: Create underlying documents i.e.
Document 1:
Business A pays Business B
fingerprint: fingeprint1
Document 2: Business B pays Business C
Step 2: Create Ring
Create Ring with authorization token for business B
trigger Document 1
target Document 2
Step 3: Create checkouts
Create checkout for trigger as normal,
Create checkout for target with chain type
Step 4: Pay target checkout
This only requires the login of Business B, no actual payment is made at this point.
Step 5: Pay trigger checkout
Paying the trigger document will automatically pay the target document, Business B will receive any residual amount
