<!-- include css file -->
<link rel="stylesheet" href="./theme/messages.css">
<link rel="stylesheet" href="./theme/darkmode.css">
<link rel="stylesheet" href="./theme/beta.css">

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.postman.com/run-collection/postman_collection.json)

# API Support

If you have any questions or need help with the APIs, you can open a ticket on our support portal. Click on the button below to open a ticket.

<script src=https://youtrack.flowpay.it/static/simplified/form/form-entry.js?auto=true></script>
<div id=form-button style="">
    <button> Do you need help? Open a ticket! </button>
</div>
<script>  
    YTFeedbackForm.renderFeedbackButton(
        document.currentScript.previousElementSibling,
        { 
            backendURL: 'https://youtrack.flowpay.it', 
            formUUID: '5365c66c-6295-4631-9a5e-7afc2d5b7abf', 
            theme: 'auto', 
            language: 'en'
        }  
    );
</script>

# FlowPay's commitment to the payments ecosystem

As a Payment Service Provider (PSP), FlowPay is regulated by the Bank of Italy and we are authorised to provide payment services to our customers.

We believe that the best API a payment institution can provide is one that is tailored to the payment use case, **allowing customers to focus on their business and not the payment process**.

On the end user side, FlowPay ensures that **users own their data, can access it at any time and fully manage it**. They can choose which data to share with third parties in the most transparent way possible.

## Contributions

FlowPay welcomes contributions from partners. The OpenAPI file is publicly available on GitHub: [FlowPay/client-openapi](https://github.com/FlowPay/client-openapi).

Partners can propose changes by forking the repository and submitting a pull request.

# Introduction

The APIs provided are REST and accessible via HTTPS, some endpoints are restricted, so you need to register your application and obtain a valid access token to use them.

## Account Information Service (AIS)

AIS (Account Information Service) is a financial service that allows third parties to access a user's account information from different banks or financial institutions. AIS works by using APIs provided to securely connect to the user's bank account and retrieve the necessary information.

As a payment institution authorised by the Bank of Italy, FlowPay can offer AIS to its customers, allowing them to **access account information, balances and transaction history of the tenants' bank accounts** for which they are authorised.
These services enable many use cases such as account aggregation, personal financial management, credit scoring and many others.

In line with its principles, **FlowPay provides a seamless and compliant way to access tenants' bank details**, taking on the burden of negotiating PSD2 consent with the user and **providing a single API to access all banks**. All PSD2 consents are collected directly by FlowPay, so **there is no need for the client to implement a consent acquisition or renewal process**.
If desired, a client can initiate a consent acquisition process themselves and manage the user experience.

## Payment Initiation Service (PIS)

PIS (Payment Initiation Service) is a financial service that allows third-party providers to initiate a payment transaction from a user's bank account. PIS works by using APIs provided to securely connect to the user's bank account and initiate the payment.

FlowPay is an authorised PIS Provider (PISP), which means that it can mediate between the user and the bank to authorise the payment.

APIs allow users to initiate any traditional payment type:

- Simple account-to-account payment: user can initiate a SEPA Credit Transfer (SCT) payment from one of its bank accounts.
- Future date payment: the payer can schedule a payment for a future date.

In addition, FlowPay extends traditional payment methods by providing value-added services such as

- **Bulk payment**: payer can initiate a single payment with a single Strong Customer Authentication (SCA) to pay multiple payment requests or documents at once.
- **Payment chain**: user can authorise a payment to be executed when a previous payment has been successfully received.
- **Locked payment**: the user can authorise a payment to be executed if a previous payment has been successfully received. The check is performed by the client application that initiated the payment request.

Each of these services uses a FlowPay technical account, but the payment retains the original payer and payee information.

# Onboarding

A partner who intends to develop an integration to access tenants' data must first register its application and obtain the `client_id` and `client_secret` pair.

The developer portal can be reached at https://developer.flowpay.it, to access it's necessary to have a company account registered with FlowPay services.

# Sandbox environment

FlowPay provides a sandbox environment to allow partners to test the APIs before going into production. The sandbox is a safe space where you can experiment with the APIs without affecting real accounts or transactions.

# Mock environment

FlowPay provides a mock environment designed to help developers quickly prototype and validate their integration without connecting to real systems. This environment is built using [Prism](https://github.com/stoplightio/prism), which serves the OpenAPI specification as live endpoints, and uses [faker.js](https://fakerjs.dev/) to generate random but realistic data.

## How it works

The mock server validates all requests against the OpenAPI specification and returns mocked responses that match the expected output schema. Each field in the response is populated with context-aware fake data, such as realistic names, IBANs, dates, or UUIDs.
The mock environment is available at: `https://api.mock-flowpay.it/v3`; all endpoints mirror those defined in the OpenAPI specification.

The mock server automatically checks:

- required query parameters and headers
- request body structure and content
- response conformance to schema

This allows you to focus on building your application without worrying about backend logic or data consistency.

You can use the mock environment to:

- Test your application's integration with FlowPay APIs
- Front-end integration without backend logic
- Early validation of request formats
- Automated tests with predictable structure

Example request to list payment requests:

```
GET https://api.mock-flowpay.it/v3/platform/payment-requests
Authorization: Bearer test-token
```

Response:

```json
{
  "data": [
    {
      "id": "req_82ae0947",
      "amount": 1250,
      "currency": "EUR",
      "created_at": "2024-06-01T14:30:00Z",
      "description": "Mocked request",
      "status": "created"
    }
  ],
  "next_cursor": "abc123",
  "has_more": false
}
```

## Validating your requests

You can deliberately send incorrect requests (e.g. missing required fields) to confirm how the system returns validation errors. This helps ensure that your integration meets the expected structure before switching to a real sandbox or production environment.

Example invalid request (missing `Authorization` header):

```
GET https://api.mock-flowpay.it/v3/platform/payment-requests
```

Response:

```json
{
  "type": "https://stoplight.io/prism/errors#MISSING_HEADER",
  "title": "Authorization header is required",
  "status": 400,
  "detail": "Missing required header: Authorization"
}
```

# Pagination

FlowPay APIs implement offset-based pagination on all list endpoints, following a standard response structure.

## Query parameters

When requesting a paginated resource, the following query parameters are supported:

- `limit` (integer): the maximum number of items to return. Default is 50, maximum is 100.
- `offset` (integer): the number of items to skip before starting to return results.

## Response structure

Each paginated response follows the `PaginatedResult` format:

- `total`: total number of available items.
- `limit`: maximum number of items returned in this page (as requested).
- `offset`: number of items skipped from the beginning of the collection.
- `count`: number of items actually returned in this response.
- `items`: array of objects representing the current page results.

## Example

### Request

```
GET /platform/payment-requests?limit=20&amp;offset=0
Authorization: Bearer {token}
```

### Response

```json
{
  "total": 147,
  "limit": 20,
  "offset": 0,
  "count": 20,
  "items": [
    {
      "id": "req_12345",
      "amount": 1000,
      "currency": "EUR",
      "created_at": "2024-06-01T10:00:00Z",
      "status": "created"
    }
  ]
}
```

This structure allows clients to calculate pagination UI and control navigation across multiple pages using `offset` and `limit`.

# Rate limits

Requests are limited to 100 requests per minute per source IP, if you exceed this limit you will receive a 429 error.

There is also a burst limit of 10 requests per second.
