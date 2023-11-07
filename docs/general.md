[![Run in Postman](https://run.pstmn.io/button.svg)](/postman_collection)
<br><br><br>
If you can't find what you're looking for, it doesn't mean we can't do it. Write to us and tell us about your idea.

# API Support

If you have any questions or need help with the APIs, you can open a ticket on our support portal. Click on the button below to open a ticket.

<script src=https://youtrack.flowpay.it/static/simplified/form/form-entry.js?auto=false></script>
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

FlowPay is both a payment institution and a start-up.

As a Payment Service Provider (PSP), we are regulated by the Bank of Italy and we are authorised to provide payment services to our customers. As a start-up, we are always looking for new ideas and new ways to improve our services and help you grow your business.

We believe that the best API a payment institution can provide is one that is tailored to the payment use case, **allowing customers to focus on their business and not the payment process**.

On the end user side, FlowPay ensures that **users own their data, can access it at any time and fully manage it**. They can choose which data to share with third parties in the most transparent way possible.

# Introduction

FlowPay's services compose a multi-tenant platform, where each tenant is a user of FlowPay and can be a legal entity or an individual.
Users can delegate third-party applications, called _clients_, to act on their behalf using the OAuth2 protocol.

The APIs provided are REST and accessible via HTTPS, some endpoints are protected by the OAuth2, so you need to register your application and obtain a valid access token to use them.

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
- Recurring payment: the payer can schedule a recurring payment with a fixed frequency.

In addition, FlowPay extends traditional payment methods by providing value-added services such as

- **Bulk payment**: payer can initiate a single payment with a single Strong Customer Authentication (SCA) to pay multiple payment requests or documents at once.
- **Payment chain**: user can authorise a payment to be executed when a previous payment has been successfully received.
- **Locked payment**: the user can authorise a payment to be executed if a previous payment has been successfully received. The check is performed by the client application that initiated the payment request.

Each of these services uses the FlowPay technical account, but the payment retains the original payer and payee information.

# Onboarding

Tenants can share their FlowPay resources with third-party applications by simply granting permission based on the OAuth2 protocol.

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

Once the application has been created, it is already enabled for the sandbox environment but is not yet able to act as a third-party application for the production environment. It is however possible to use client credentials to access the APIs for your own tenant in production.

# Sandbox environment

The sandbox is a shared environment where it is possible to test most of the basic flows and functionalities of the APIs, without using real data and accounts.
We enable you to test the authentication as a third party by providing you with two fake businesses, each with a fake bank account and fake account data, so you can test AIS and PIS flows.

## Limitations of the public sandbox environment

The sandbox uses sandbox open banking APIs and doesn't have access to the FlowPay technical account, so some features are not available or are limited.

Open banking sandbox APIs don't allow us to monitor the status of the payment, so the webhook for the payment status change is not triggered, and the payment status is not updated.
In addition, AIS data is not provided by the bank in the sandbox environment, so if you try to complete the AIS flow, you will be able to connect the bank account, but won't receive any balance and transactions data from it. We do however provide fake AIS data on the fake bank accounts that come with the sandbox users.

It is not possible to use the bulk payment service, as it requires the FlowPay technical account to be used.

Also, due to the nature of the sandbox environment, it is not possible to use the payment chain service, as it requires a real payment to be executed.

Onboarding users isn't allowed in the sandbox environment also.

## Request a dedicated sandbox environment

If you need to test the full functionality of the APIs, you can request a dedicated sandbox environment, and test all the features of the APIs, including the onboarding flow.

# Oauth2 authentication flows

FlowPay uses the OAuth2 protocol to authenticate third-party applications and users.
Oauth2 is the industry standard for authentication and authorization, and is used by most of the major companies in the world.

The scopes

FlowPay supports the following authentication flows:

## Authorization code flow

## Client credentials flow

The Client Credentials grant type is used by clients to obtain an access token outside of the context of a user.

This is typically used by clients to access resources about themselves rather than to access a user's resources.

The client makes a request to the token endpoint by sending the following parameters using the "application/x-www-form-urlencoded" format per Appendix B with a character encoding of UTF-8 in the HTTP request entity-body:

- grant_type
  - REQUIRED. Value MUST be set to "client_credentials".
- scope
  - OPTIONAL. The scope of the access request as described by Section 3.3.
- client_id
  - REQUIRED. The client identifier as described by Section 2.2.
- client_secret
  - REQUIRED. The client secret as described by Section 2.3.1.

## How token works and tenant filtering

(negli header )

# Pagination

# Rate limits

Requests are limited to 100 requests per minute per source IP, if you exceed this limit you will receive a 429 error.

There is also a burst limit of 10 requests per second.

<h3>Tenant</h3>

<p>A tenant can be a company or a single person, is identified by a unique identifier (tenant ID ) and owns its data,
    for transparency and security reasons each tenant can manage its data via the dedicated <a
        href="https://account.flowpay.it">Account portal</a>
</p>
