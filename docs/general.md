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

## Contributi

FlowPay accoglie con favore i contributi da parte dei partner. Il file OpenAPI è disponibile pubblicamente su GitHub: [FlowPay/client-openapi](https://github.com/FlowPay/client-openapi).

I partner possono proporre modifiche effettuando un _fork_ del repository e inviando una _pull request_.

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

## Become part of the FlowPay ecosystem

To use FlowPay APIs you need to register your tenant, create your first application and obtain a valid access token. If you don't have an account, you need to register.

The onboarding procedure takes a few minutes, to start you need to click the _Register_ button at the bottom of the <a href="https://developer.flowpay.it">developer portal</a>.

The first step of registration is to verify a bank account, this step is also one of the two requirements for verifying the company's identity, therefore the linked account must belong to the company you intend to register.

Once the bank account has been verified, the system will automatically retrieve the company's details. Finally, you need to provide your personal information as a contact person for the company. If you want, you can also proceed with your identity verification and obtain your personal FlowPay account, this adds a level of security and helps to track tokens granted for your company and could be mandatory to grant a token for some advanced services.

In addition to bank account ownership, a second identity verification step is required, which may vary depending on the services requested.
In most cases, digital identity verification or verification of a certified email address is sufficient.
Failure of the second identity verification step will not prevent registration, but it will be necessary to provide all requested information before the client can operate in the production environment.

## Register your application

Within the developer portal access the "Applications" section and press the "+" button, the creation screen requires the following information:

- **Application logo**: it can be any image, it will be used in the consent request screen to allow the user to identify the application to which he is granting access.The supported formats are: jpg or png with 1:1 aspect ratio

- **Application name**: unique name assigned to the application, in addition to allowing its identification, the application name is present in the user invoice if it is used for the creation of RTP. The name is also used within the invoice following the API supply to group the costs relating to the use of the individual applications.

- **Homepage URL**: must contain the URL to the homepage of your service or company. The URL is shown during the consent screen to allow the user to identify the company to which he is granting consent.

- **Privacy URL**: must point to a page containing how the data obtained from FlowPay is processed and the purpose of accessing this data.

## Be enabled as a third-party application

Once the application has been created, it is already enabled for the sandbox environment but is not yet able to act as a third-party application for the production environment.

# Sandbox environment

FlowPay provides a sandbox environment to allow partners to test the APIs before going into production. The sandbox is a safe space where you can experiment with the APIs without affecting real accounts or transactions.

## Available sandbox types

FlowPay provides two types of sandbox environments to allow partners to test API integration:

1. **Public sandbox**  
   Available to all developers who register an application through the portal. It provides an initial experience with FlowPay APIs, useful to explore functionalities and simulate standard flows. However, it comes with several functional limitations.

2. **Private sandbox**  
   Upon request, FlowPay can activate a dedicated sandbox for a specific partner. This environment enables testing of advanced features such as onboarding flows, bulk payment, payment chain, and more realistic behaviors.

## Public sandbox limitations

The public sandbox environment has the following functional limitations:

- **Payment status update callbacks are not triggered**, as the APIs do not receive responses from banks.
- **AIS data (balances and transactions) is not returned** for real bank accounts.
- Only **fake AIS data** is available on preconfigured test accounts.
- **Bulk payment service is not available**.
- **Payment chain service is not available**, since it requires real payments.
- **User onboarding is not supported**.

## Requesting a private sandbox

If you need to test full API capabilities, you can request a private sandbox by submitting a contact form or opening a ticket via the support portal.

# Pagination

All endpoints that return a list of items use cursor-based pagination.

The response includes the following fields:

- `data`: the list of items returned in the current page
- `next_cursor`: a string value to be used as the `cursor` query parameter to retrieve the next page
- `has_more`: a boolean indicating whether more results are available

To paginate through results, pass the `cursor` parameter with the value of `next_cursor` from the previous response. If `has_more` is false or `next_cursor` is null, there are no additional pages.

# Rate limits

Requests are limited to 100 requests per minute per source IP, if you exceed this limit you will receive a 429 error.

There is also a burst limit of 10 requests per second.
