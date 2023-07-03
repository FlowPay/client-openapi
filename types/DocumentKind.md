Type of document:
- `bill`: Document representing a bill. It can be an ecommerce bill or a any other kind of bill (e.g. a bill from a restaurant).
- `bulk`: Document used to manage bulk payments. This type of document can contain multiple other documents.
- `invoice`: Document used to manage invoice lifecycle. This type of document can handle invoices, pro-forma invoices and credit notes.
- `transfer`: Kind of document that allow clients to manage payment requests with low constraints. <br> Clients can use transfers to fast prototyping payment initiation use cases or to avoid specific document's lifecycle managment. <br><b>Note: </b> In order to use transfers in production, a more in-depth due diligence is required.


