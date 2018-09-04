---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://app.kontist.com/signup/index.html?pid=apidoc'>Sign Up for a Kontist Account</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Kontist API. Kontist allows you to access your own bank transaction data as well as the initiation and authentication of financial transactions. Please note, that you will still need to confirm the latter with an individual authorization code which you will receive through SMS.

# Authentication

You can start playing with the Kontist API using [HTTP Basic Auth](https://en.wikipedia.org/wiki/Basic_access_authentication). However this is not to be used in production as it is highly rate limited. The documentation of our oAuth flow is in preparation and will be added at a later point in time. 

`Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l`

<aside class="notice">
You must replace <code>QWxhZGRpbjpPcGVuU2VzYW1l</code> with a base64-encoded representation of <code>your_username:your_password</code>.
</aside>

# Bank Accounts

The Bank account endpoint returns all customer owned bank accounts including their balances

## Get All Bank Accounts

All customers can have various bank accounts, however most of them will only have one.

*When parsing the API result, please do not forget that you will receive a list of accounts.*

<aside class="notice">
  The `accountType` property is going to be deprecated in the near future, please make sure your client is not relying on that.
</aside>

```shell
curl "https://api.kontist.com/api/accounts/"
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l"
```

> The above command returns JSON structured like this:

```json
[
    {
        "id": 4711,
        "iban": "DE06110101001000004444",
        "balance": 17325,
        "accountType": "solaris"
    }
]
```

# Transactions

The transactions endpoint allows you to fetch all of your booked and unbooked banking transactions. These include, but are not limited to the following types

* SEPA_CREDIT_TRANSFER
* SEPA_DIRECT_DEBIT
* DIRECT_DEBIT
* SEPA_DIRECT_DEBIT_RETURN

## Get All Transactions

<aside class="notice">
  The `to` property is going to be deprecated in the near future, please make sure your client is not relying on that.
</aside>

```shell
curl "https://api.kontist.com/api/accounts/4711/transactions/"
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l"
```

> The above command returns JSON structured like this:

```json
{
  "next": "/api/accounts/4711/transactions?page=2",
  "total": 3,
  "results": [
    {
      "id": 912182,
      "from": 4711,
      "amount": 659,
      "type": null,
      "category": null,
      "name": "Vae Zrniv4chmon1; St Louis; Fr",
      "iban": null,
      "purpose": null,
      "e2eId": null,
      "bookingDate": "2018-08-08T16:26:52+00:00",
      "valutaDate": null,
      "bookingType": null,
      "originalAmount": "760",
      "foreignCurrency": "CHF",
      "paymentMethod": "credit_card",
      "invoiceNumber": 12,
      "invoicePreviewUrl": "https:",
      "invoiceDownloadUrl": "https:"
    },
    {
      "id": 905800,
      "from": 4711,
      "amount": 50433,
      "type": null,
      "category": null,
      "name": "Karl Lagerfeld",
      "iban": "DE35120300001012765235",
      "purpose": "E-16ab84e3177fdb4f692e02fd5b8ce808",
      "e2eId": "E-16ab84e3177fdb4f692e02fd5b8ce808",
      "bookingDate": "2018-08-06T22:00:00+00:00",
      "valutaDate": "2018-08-06T22:00:00+00:00",
      "bookingType": "SEPA_CREDIT_TRANSFER",
      "originalAmount": null,
      "foreignCurrency": null,
      "paymentMethod": "bank_account",
      "invoiceNumber": null,
      "invoicePreviewUrl": null,
      "invoiceDownloadUrl": null
    }
  ]
}
```

This endpoint retrieves all transactions of the authenticated user.

### HTTP Request

`GET https://api.kontist.com/api/accounts/{account_id}/transactions`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
page | 1 | If set, then indicating the page of the result

<aside class="success">
Remember - the amount will never be returned as negative . You will have to also check whether the <code>from</code> (outgoing transaction) or <code>to</code> (incoming transaction) field contains the current account ID.
</aside>

## Get a Specific Transaction

<aside class="notice">
  The `to` property is going to be deprecated in the near future, please make sure your client is not relying on that.
</aside>

```shell
curl "https://api.kontist.com/api/accounts/4711/transactions/905800"
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l"
```

> The above command returns JSON structured like this:

```json
{
  "id": 905800,
  "from": 4711,
  "amount": 50433,
  "type": null,
  "category": null,
  "name": "Karl Lagerfeld",
  "iban": "DE35120300001012765235",
  "purpose": "E-16ab84e3177fdb4f692e02fd5b8ce808",
  "e2eId": "E-16ab84e3177fdb4f692e02fd5b8ce808",
  "bookingDate": "2018-08-06T22:00:00+00:00",
  "valutaDate": "2018-08-06T22:00:00+00:00",
  "bookingType": "SEPA_CREDIT_TRANSFER",
  "originalAmount": null,
  "foreignCurrency": null,
  "paymentMethod": "bank_account",
  "invoiceNumber": 12,
  "invoicePreviewUrl": "https:",
  "invoiceDownloadUrl": "https:"
}
```

This endpoint retrieves a specific transaction.

### HTTP Request

`GET https://api.kontist.com/api/accounts/{account_id}/transactions/{transaction_id}`
# SEPA Credit Transfers

The flow of  SEPA credit transfers (wire transfers) is a 2-step process, divided into 2 requests

1. Registration of all transaction-relevant data and requesting a TAN
2. Confirmation of the previously registered credit transfer with the received TAN

## SEPA Credit Transfers Creation

In order to create a credit transfer you need at least the following data

Parameter | Mandatory | Description
--------- | ------- | -----------
recipient | yes | The name of the recipient
iban|yes|Recipient's IBAN
amount|yes|The transaction amount in Euro-Cents
note|yes|The booking text which will appear on sender's and recipient's bank statements
e2eId|no|An optional end-to-end ID such as an invoice ID or booking reference

```shell
curl "https://api.kontist.com/api/accounts/4711/transfer" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
        "recipient": "Karl Brenner",
        "iban": "DE89370400440532013000",
        "amount": 1250,
        "note": "Thank you for paying my lunch",
        "e2eId": null
      }'
```

> The above command returns JSON structured like this:

```json
{
    "status": "confirmation_required",
    "recipient": "Karl Brenner",
    "iban": "DE89370400440532013000",
    "amount": 1,
    "note": "Thank you for paying my lunch",
    "meta": {
        "authorizationTokenLength": 6
    },
    "links": {
        "self": "/api/accounts/4711/transfer/f55641811042b9e85989bd57c3718346ctrx"
    }
}
```

## SEPA credit transfers Confirmation

As you have noticed, the status of the newly created SEPA credit  transfer is being returned as `confirmation_required` therefore you now need to confirm it with the TAN received on your registered mobile phone number.

<aside class="notice">
In addition to the TAN, refered to as <code>authorizationToken</code>, you are required to provide the exact same payload which you provided when requesting the TAN.
</aside>

```shell
curl "https://api.kontist.com/api/accounts/4711/transfer/f55641811042b9e85989bd57c3718346ctrx" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X PUT \
  -d '{
  		"recipient": "Karl Brenner",
  		"iban": "DE89370400440532013000",
  		"amount": 1250,
  		"note": "Thank you for paying my lunch",
  		"e2eId": null,
  		"authorizationToken": "012345"
	}'
```

> The above command returns JSON structured like this:

```json
{
    "status": "accepted",
    "recipient": "Karl Brenner",
    "iban": "DE89370400440532013000",
    "amount": 1250,
    "note": "Thank you for paying my lunch",
    "meta": {
        "authorizationTokenLength": 6
    },
    "links": {
        "self": "/api/accounts/4711/transfer/f55641811042b9e85989bd57c3718346ctrx"
    }
}
```


