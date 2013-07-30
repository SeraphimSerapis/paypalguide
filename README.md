# PayPal Product 101
[Tim Messerschmidt](http://timmesserschmidt.com/)  
29. Juli 2013  
Version 0.1

## Content
1. [Introduction](#introduction)
2. [Express Checkout](#express-checkout-ec)
3. [Adaptive Payments](#adaptive-payments)
4. [PayPal Button](#paypal-button)
5. [REST API](#rest-api)
6. [Mobile Payments](#mobile-payments)
7. [Log In with PayPal](#log-in-with-paypal-liwp)

## Introduction

This guide shall provide a brief overview over the different PayPal products that allow to process payments in (web-)applications. It will be updated over time to allow a quickstart using our products.

Please **always** make sure to check with the documentation as this guide doesn't try to be complete.

## Express Checkout (EC)

Express Checkout is the most famous and most important PayPal API so far. It allows for one time purchases, subscriptions, billing agreements and parallel payments. When accessing Express Checkout on mobile provides a mobile flow which we call Mobile Express Checkout (MEC). MEC can be used in combination with WebViews in native mobile applications to implement PayPal payments.

### One time purchases

Traditional eCommerce one time purchases get implemented by utilizing 3 simple API calls:

1. SetExpressCheckout
	- creates the payment itself
	- redirects the user to PayPal's login page to authenticate
2. (Optional) GetExpressCheckoutDetails
	- allows to get user details before the actual pamyent happens
3. DoExpressCheckoutPayment
	- this API call actually executes the created & authenticated payment using a token
	
#### Shortcut

Shortcut is a way of implementing PayPal Express Checkout that allows skipping the actual registration process at the merchant's page and provides a callback that provides user details after the checkout itself. This is an amazing feature that provides great conversion rates as the user is not require to enter tons of details while the merchant still is able to access needed information like the customer's name, shipping address and Email.

#### Reference Transactions (Billing Agreement)

Creating billing agreements using Reference Transaction requires a special permission that must be obtained during the onboarding process with PayPal or through PayPal's [Merchant Technical Service](http://paypal.com/mts). [MTS](http://paypal.com/mts) is also able to enable this feature for our test environment called *Sandbox*.

The flow is pretty easy and powerful as it allows to create One Click Payment experiences.

##### Creating a Billing Agreement:

1. SetExpressCheckout
	- define the amount as 0 as we don't create a particular payment yet
2. CreateBillingAgreement
	- this returns a billing agreement id that will be used for future transactions
	
##### Paying with Reference Transactions:

1. DoReferenceTransaction
	- provide the billing agreement id that was obtained via *CreateBillingAgreement*

##### Cancelling a Billing Agreement

1. BAUpdate
	- set `BillingAgreementStatus` to `Canceled`
	
#### Parallel Payments

This is pretty similar to a regular Express Checkout call except this time you'll be able to define multiple Payment Requests with multiple Receivers:

1. SetExpressCheckout
	- details for multiple payments [here](http://developer.paypal.com/webapps/developer/docs/classic/express-checkout/ht_ec-parallelPayments/)
2. (Optional) GetExpressCheckoutDetails
3. DoExpressCheckoutPayment
	- please see the [DoExpressCheckoutPayment](http://developer.paypal.com/webapps/developer/docs/classic/express-checkout/ht_ec-parallelPayments/) for details about defining multiple Payments

#### Subscriptions

Express Checkout allows to create subscriptions via creating a so called recurring payments profile. The flow is relatively similar to Reference Transactions but this time you'll need to define an amount.

PayPal allows to create up to 10 different recurring payments with just one API call. Each of these recurring payments need an assigned recurring payments profile that needs to be created seperately:

1. SetExpressCheckout
	- set the field `L_BILLINGTYPEn` for each of n (0 up to 9) to RecurringPayments
	- set `L_BILLINGAGREEMENTDESCRIPTIONn` (n being 0 up to 9) to your subscription's description
	- set `PAYMENTREQUEST_0_AMT` to 0 if there is no associated purchase - your amount otherwise.
2. (Optional) GetExpressCheckoutDetails
	- allows to get some user details if needed
3. CreateRecurringPaymentsProfile
	- this is where you define how often you want to charge the user (daily, weekly, monthly, annual), how much and if it's a *digital* or *physical* good.
	
Please check out [the documentation](http://developer.paypal.com/webapps/developer/docs/classic/express-checkout/integration-guide/ECRecurringPayments/) for further details.

## Adaptive Payments

Adaptive Payments is PayPal's number one API when it comes to marketplaces. Next to regular **1:1** payments (Simple Payments) it allows to create Parallel Payments (**1:n** - n being maximum 6 receivers) and Chained Payments (**1:1**:n - n being max. 5).

Payments can be *delayed* up to **90** days in order to allow working with the received money. Furthermore payments may be *preauthorized* and executed later - this is handy for scenarios like *pay at delivery*.

Payments being made with Adaptive Payments are **NOT** mobile optimized. We strongly encoure using our *Mobile SDKs* for this.

The developer can define who pays the PayPal's fee. The **4** different setups are:

1. Sender (customer) pays the fee
2. Receiver pays the fee in a Parallel Payment
3. Each receiver pays a part of the fee in a Chained Payment
4. Primary receiver pays the fee in a Chained Payment

More information regarding fees can be found in the [documentation](http://developer.paypal.com/webapps/developer/docs/classic/adaptive-payments/integration-guide/APIntro/#id091QF0N0MPF__id092SH0YG0Y4).

In case you want to build a crowdfunding platform this guide might come in handy: [Crowdfunding Application Guidelines](http://developer.paypal.com/webapps/developer/docs/classic/lifecycle/crowdfunding/).

Adaptive Payments used to be the base for PayPal's [Mobile Payments Library](http://developer.paypal.com/webapps/developer/docs/classic/mobile/gs_MPL/) and still powers the PayPal payments in the new [PayPal Mobile SDK for Android](http://developer.paypal.com/webapps/developer/docs/integration/mobile/android-integration-guide/) and [iOS](http://developer.paypal.com/webapps/developer/docs/integration/mobile/ios-integration-guide/).

### Simple Payment

The most basic usage of Adaptive Payments is creating a Simple Payment between a sender (your user) and a receiver (that doesn't have to be you):

1. Pay
2. (Optional) PaymentDetails

You will need to pass a list of receivers with only one receiver specified as documented [here](http://developer.paypal.com/webapps/developer/docs/classic/adaptive-payments/integration-guide/APCallsHeadersAndPaymentTypes/).

### Parallel Payment

In the Pay and ExecutePayment call a list of receivers needs to be provided as following:

1. Pay
	- `receiverList.receiver(0).amount=first_amount
&receiverList.receiver(0).email=first_receiver_email`
	- same applies to the other (up to 5 additional) receivers
2. (Optional) PaymentDetails

More details over [here](http://developer.paypal.com/webapps/developer/docs/classic/adaptive-payments/ht_ap-parallelPayment-curl-etc/).

### Chained Payment

Chained payments are similar to parallel payments except this time one receivers needs to be declared as primary receiver which will declare all other receivers as secondary receivers.

1. Pay
2. (Optional) PaymentDetails

### Delayed Payment

Chained Payments can be delayed in order to allow the primary receiver to keep the money for up to 90 days.

1. Pay
2. (Optional) PaymentDetails
3. ExecutePayment

The documentation provides [a step-by-step guide](http://developer.paypal.com/webapps/developer/docs/classic/adaptive-payments/ht_ap-delayedChainedPayment-curl-etc/) that helps implementing this.

### Preapproved Payments

Preapproved payments can be created up to a total amount and a number of transactions and executed afterwards. The parameters specifying these limitations are:

- `maxAmountPerPayment`
- `maxNumberOfPayments`
- `maxTotalAmountOfAllPayments`
- `startingDate`
- `endingDate`

Instead of providing the payment command the preapproval command must be specified when redirecting to PayPal:

`_ap-payment` changes to `_ap-preapproval` as documented in the [APCommands](http://developer.paypal.com/webapps/developer/docs/classic/adaptive-payments/integration-guide/APCommands/).

1. Preapproval
	- response will contain a preapproval key
2. Redirect the user to `https://www.sandbox.paypal.com/cgi-bin/webscr?cmd=_ap-preapproval&preapprovalkey=InsertPreapprovalKeyHere`
	- redirect to `https://www.paypal.com/...` if not using the Sandbox environment
3. (Optional) PreapprovalDetails
	- allows to query for details for the specified preapproval
4. Pay
	- include the preapproval key that was obtained earlier

A complete guide on implementing Payment Preapproval can be found [here](http://developer.paypal.com/webapps/developer/docs/classic/adaptive-payments/ht_ap-basicPreapproval-curl-etc/).

## PayPal Button

### Website Payments Standard (WPS)

WPS is the traditional way to create a frontend PayPal implementation using HTML forms. The main functionality is allowing one time purchases marked as *Buy now* or *Donate* button and subscriptions.

### JavaScript Buttons

The new JavaScript Buttons started in 2013 and add a layer on top of WPS that allows a much faster integration of the PayPal button. On top of the regular Buy now button and Subscriptions a QRCode button got implemented that allows to complete the checkout process using a smartphone.

## REST API

The new REST API got launched at SXSW in March 2013 and aims for a great developer experience using modern standards like a REST interface, [OAuth 2.0](http://oauth.net/2/) for user authentication and JSON as data format.

The API allows to process credit cards directly via an API call (which we call *DCC* - Direct Credit Card) if the merchant is [PCI compliant](http://en.wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard). On top of that a feature called vault allows to store credit card and receive them tokenized.

Also so called EC payments using PayPal accounts is being supported. This is the regular PayPal flow that requires users to log in with their PayPal username and password to confirm the payment.

There are SDKs for most modern (web-)languages like [Ruby](http://github.com/paypal/rest-api-sdk-ruby), [Node.js](http://github.com/paypal/rest-api-sdk-nodejs), [Python](http://github.com/paypal/rest-api-sdk-python), [PHP](http://github.com/paypal/rest-api-sdk-php), [.NET](http://github.com/paypal/rest-api-sdk-dotnet) and [Java](http://github.com/paypal/rest-api-sdk-java). All documentation for these SDKs can be found in the [REST API Reference](http://developer.paypal.com/webapps/developer/docs/api/).

In case you're missing your favourite language: There are [cURL](http://developer.paypal.com/webapps/developer/docs/integration/direct/make-your-first-call/) samples that help you creating your own wrapper.

The REST API is using [HATEOAS](http://en.wikipedia.org/wiki/HATEOAS) (Hypermedia as the Engine of Application State) which basically means: our API is smart and will tell you what to do next with the result of your request. This allows avoiding hardcoded API endpoints and is *pure awesome* ;-).

### Obtaining client credentials

A secret and id that identify your client against our server can be obtained in the [Applications section](http://developer.paypal.com/webapps/developer/applications/myapps) of our [Developer Portal](http://developer.paypal.com).

## Mobile Payments

As stated in the Express Checkout part PayPal Payments can be received via the Mobile Express Checkout. In case you're looking for a native integration the best way to do so is using the Mobile SDKs for Android and iOS. In countries that are not supported by the new SDK yet or if additional features are needed the Mobile Payments Library is helpful.

### Mobile SDK

The Mobile SDK for iOS got released in March 2013 and the Android library in May 2013. It allows for accepting both PayPal payments and credit card payments (without requiring PCI compliance).

The credit card flow can be either supercharged with credit card scannign via [card.io](http://www.card.io/) or entering the card's details manually.

Right now the SDK only allows one time payments. We'll be adding more functionality like enabling subscriptions, parallel and chained payments in future updates.

The newest versions of the SDK and sample apps can be found at GitHub: [iOS](http://github.com/paypal/PayPal-iOS-SDK/) & [Android](http://github.com/paypal/PayPal-Android-SDK/)

### Mobile Payments Library (MPL)

The [Mobile Payments Library](http://developer.paypal.com/webapps/developer/docs/classic/mobile/gs_MPL/) allows payments based on Adaptive Payments and is ideal for marketplaces and P2P (peer to peer) applications. There is a version for Android and iOS but we don't plan to release additional feature updates.

## Log In with PayPal (LIWP)

This API launched in 2011 an is known as *PayPal Access*. Access used OAuth 1.0a and OpenID to allow authenticating users via their existing PayPal accounts. Furthermore some profile information like the main shipping address and date of birth can be obtained in order to provide a nice onboarding experience for users.

In April 2013 Access got was relaunched as *Log In with PayPal* and enables a feature called *Seamless Checkout* which uses an obtained token that allows skipping logging again after using LIWP. The seamless checkoht works with Express Checkout version 94+ and the new REST API. Instead of sticking with OAuth 1.0a LIWP uses OpenID Connect which uses OAuth 2.0 for user authentication.