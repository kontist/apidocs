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

## JWT auth

You can read more about JWT token auth under [https://jwt.io/introduction](https://jwt.io/introduction).

To start using JWT auth you have to first create refresh token or auth token. Refresh token would be used next to create auth token. Auth token is valid for a specified amount of time. After it expires you should issue new auth token via refresh token.

### Create refresh token

To create refresh token (valid for `30 days`) you would have to use:

```shell
curl -X POST \
  https://api.kontist.com/api/user/refresh-token \
  -H 'Content-Type: application/json' \
  -d '{
  "email": "YOUR_EMAIL",
  "password": "YOUR_PASSWORD"
}'
```

If your credentials are valid, you should receive refresh token in JSON response:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI2NjYxYjIyZC01NzRmLTQwOWMtYjRlYy03ZmJhOTQ1ZGYwYjkiLCJzY29wZSI6InJlZnJlc2giLCJpYXQiOjE1NTg0NTM5MzUsImV4cCI6MTU2MTA0NTkzNX0.jXGbFGKxGciGGwNXhgJz6R0nd_swniBHFSOul6V4kjY"
}
```

### Create auth token

To create authToken (valid for exactly `1 hour`):

```shell
curl -X POST \
  https://api.kontist.com/api/user/auth-token \
  -H 'Content-Type: application/json' \
  -d '{
  "email": "YOUR_EMAIL",
  "password": "YOUR_PASSWORD"
}'
```

### Create auth token using refresh token

You can also create authToken using refresh token:

```shell
curl -X POST \
  https://api.kontist.com/api/user/auth-token \
  -H 'Authorization: Bearer REFRESH_TOKEN' \
  -H 'Content-Type: application/json'
```

It's usable when your access token gets invalid and you would like to get a new one without prompting the user for credentials again.

### Auth token invalidation

If you would like to invalidate token before it expires, you would have to use:

```shell
curl -X DELETE \
  https://api.kontist.com/api/user/auth-token \
  -H 'Authorization: Bearer ACCESS_TOKEN'
```

### Refresh token invalidation

Same works for refresh token:

```shell
curl -X DELETE \
  https://api.kontist.com/api/user/refresh-token \
  -H 'Authorization: Bearer ACCESS_TOKEN'
```

You can access Kontist API endpoints by adding `Authorization`: `Bearer AUTH_TOKEN` header to your requests.

# Bank Accounts

The Bank account endpoint returns all customer owned bank accounts including their balances

## Get All Bank Accounts

All customers can have various bank accounts, however most of them will only have one.

_When parsing the API result, please do not forget that you will receive a list of accounts._

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

- SEPA_CREDIT_TRANSFER
- SEPA_DIRECT_DEBIT
- DIRECT_DEBIT
- SEPA_DIRECT_DEBIT_RETURN

## Get All Transactions

<aside class="notice">
  The `to` property is going to be deprecated in the near future, please make sure your client is not relying on that.
</aside>

```shell
curl "https://api.kontist.com/api/accounts/4711/transactions/"
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l"
  -H "Accept: application/vnd.kontist.transactionlist.v2.1+json"
```

> The above command returns JSON structured like this:

```json
{
  "next": "/api/accounts/4711/transactions?page=2",
  "total": 3,
  "results": [
    {
      "id": 912182,
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
      "invoiceDownloadUrl": "https:",
      "userSelectedBookingDate": null,
      "documentType": null,
      "mandateNumber": null
    },
    {
      "id": 905800,
      "amount": -50433,
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
      "invoiceDownloadUrl": null,
      "userSelectedBookingDate": null,
      "documentType": null,
      "mandateNumber": null
    }
  ]
}
```

This endpoint retrieves all transactions of the authenticated user.

### HTTP Request

`GET https://api.kontist.com/api/accounts/{account_id}/transactions`
`Header: { Accept: "application/vnd.kontist.transactionlist.v2.1+json" }`

### Query Parameters

| Parameter | Default | Description                                    |
| --------- | ------- | ---------------------------------------------- |
| page      | 1       | If set, then indicating the page of the result |

<aside class="success">
  Remember - the amount returned will be negative for outgoing transactions.
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
  "invoiceDownloadUrl": "https:",
  "userSelectedBookingDate": null,
  "documentType": null,
  "mandateNumber": null
}
```

This endpoint retrieves a specific transaction.

### HTTP Request

`GET https://api.kontist.com/api/accounts/{account_id}/transactions/{transaction_id}`

# SEPA Credit Transfers

The flow of SEPA credit transfers (wire transfers) is a 2-step process, divided into 2 requests

1. Registration of all transaction-relevant data and requesting a TAN
2. Confirmation of the previously registered credit transfer with the received TAN

## SEPA Credit Transfers Creation

In order to create a credit transfer you need at least the following data

| Parameter | Mandatory | Description                                                                    |
| --------- | --------- | ------------------------------------------------------------------------------ |
| recipient | yes       | The name of the recipient                                                      |
| iban      | yes       | Recipient's IBAN                                                               |
| amount    | yes       | The transaction amount in Euro-Cents                                           |
| note      | no        | The booking text which will appear on sender's and recipient's bank statements |
| e2eId     | no        | An optional end-to-end ID such as an invoice ID or booking reference           |

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

As you have noticed, the status of the newly created SEPA credit transfer is being returned as `confirmation_required` therefore you now need to confirm it with the TAN received on your registered mobile phone number.

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

# Standing orders

A standing order is a SEPA Credit Transfer executed at regular intervals specified by the initiator.

The flow of standing orders is a 2-step process, divided into 2 requests:

1. Registration of all transaction-relevant data and requesting a TAN
2. Confirmation of the previously registered transfer with the received TAN

## Standing order creation

In order to create a standing order you need at least the following data

| Parameter           | Mandatory | Description                                                                         |
| ------------------- | --------- | ----------------------------------------------------------------------------------- |
| recipient           | yes       | The name of the recipient                                                           |
| iban                | yes       | Recipient's IBAN                                                                    |
| amount              | yes       | The transaction amount in Euro-Cents                                                |
| reoccurrence        | yes       | Interval of transactions (`MONTHLY`, `QUARTERLY`, `EVERY_SIX_MONTHS` or `ANNUALLY`) |
| firstExecutionDate  | yes       | Execution date of the first transaction                                             |
| lastExecutionDate   | no        | The date until which iterations will be repeated                                    |
| note                | no        | The booking text which will appear on sender's and recipient's bank statements      |
| e2eId               | no        | An optional end-to-end ID such as an invoice ID or booking reference                |

```shell
curl "https://api.kontist.com/api/accounts/4711/standing-orders" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "recipient": "John Smith",
    "iban": "DE33100110012626008537",
    "amount": 12012,
    "note": "Regular payment",
    "e2eId": null,
    "reoccurrence": "MONTHLY",
    "firstExecutionDate": "2019-08-27T14:32:08.604Z",
    "lastExecutionDate": null
  }'
```

> The above command returns JSON structured like this:

```json
{
  "requestId": "92bf42e2eb6405f3a61babab43d623b1",
  "status": "CONFIRMATION_REQUIRED",
  "amount": 12012,
  "recipient": "John Smith",
  "iban": "DE33100110012626008537",
  "e2eId": null
}
```

## Standing order confirmation

As you have noticed, the status of the newly created SEPA credit transfer is being returned as `CONFIRMATION_REQUIRED` therefore you now need to confirm it with the TAN received on your registered mobile phone number.

```shell
curl "https://api.kontist.com/api/accounts/4711/standing-orders/confirm" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "requestId": "92bf42e2eb6405f3a61babab43d623b1",
    "authorizationToken": "749245"
  }'
```

> The above command returns JSON structured like this:

```json
{
  "id": 20,
  "status": "ACTIVE",
  "name": "John Smith",
  "iban": "DE33100110012626008537",
  "amount": 12012,
  "description": "Regular payment",
  "lastExecutionDate": null,
  "reoccurrence": "MONTHLY",
  "e2eId": "E-81359c56883b508cc891b2d466983e3a",
  "nextOccurrence": "2019-08-27T00:00:00.000+00:00"
}
```

## Standing order update

You can update following attributes of `ACTIVE` standing order:
- `amount`
- `note`
- `e2eId`
- `reoccurrence`
- `lastExecutionDate`

```shell
curl "https://api.kontist.com/api/accounts/4711/standing-orders/20" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X PATCH \
  -d '{
    "amount": 500,
    "note": "Updated standing order",
    "reoccurrence": "QUARTERLY",
    "e2eId": "123456789",
    "lastExecutionDate": "2022-12-31T20:11:02.109Z"
  }'
```

> The above command returns JSON structured like this:

```json
{
  "requestId": "1568357736285",
  "status": "CONFIRMATION_REQUIRED",
  "amount": 500,
  "recipient": "John Smith",
  "iban": "DE33100110012626008537",
  "e2eId": "123456789"
}
```

You need to confirm update of a standing order with the TAN received on your registered mobile phone number in the same way as creation of new standign order:

```shell
curl "https://api.kontist.com/api/accounts/4711/standing-orders/confirm" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "requestId": "1568357736285",
    "authorizationToken": "123953"
  }'
```

> The above command returns JSON structured like this:

```json
{
  "id": 20,
  "status": "ACTIVE",
  "iban": "DE33100110012626008537",
  "name": "John Smith",
  "description": "Updated standing order",
  "amount": 500,
  "lastExecutionDate": "2022-12-31T20:11:02.109+00:00",
  "e2eId": "123456789",
  "reoccurrence": "QUARTERLY",
  "nextOccurrence": "2019-08-27T00:00:00.000+00:00"
}
```

## Standing order cancelation

To cancel standing order you need to make following request:

```shell
curl "https://api.kontist.com/api/accounts/4711/standing-orders/20/cancel" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X PATCH
```

> The above command returns JSON structured like this:

```json
{
  "requestId": "1568357935921",
  "status": "CONFIRMATION_REQUIRED"
}
```

As you have noticed, standing order cancelation should be confirmed as well:

```shell
curl "https://api.kontist.com/api/accounts/4711/standing-orders/confirm" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "requestId": "1568357935921",
    "authorizationToken": "472948"
  }'
```

> The above command returns JSON structured like this:

```json
{
  "id": 20,
  "status": "CANCELED",
  "iban": "DE33100110012626008537",
  "name": "John Smith",
  "description": "Updated standing order",
  "amount": 500,
  "lastExecutionDate": "2022-12-31T20:11:02.109+00:00",
  "e2eId": "123456789",
  "reoccurrence": "QUARTERLY",
  "nextOccurrence": "2019-08-27T00:00:00.000+00:00"
}
```

## Fetch standing orders

Following endpoint allows you to fetch all of your standing orders:

```shell
curl "https://api.kontist.com/api/accounts/4711/standing-orders" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 20,
    "status": "CANCELED",
    "iban": "DE33100110012626008537",
    "name": "John Smith",
    "description": "Updated standing order",
    "amount": 500,
    "lastExecutionDate": "2022-12-31T20:11:02.109+00:00",
    "e2eId": "123456789",
    "reoccurrence": "QUARTERLY",
    "nextOccurrence": "2019-08-27T00:00:00.000+00:00"
  },
  {
    "id": 6,
    "status": "ACTIVE",
    "iban": "DE35120300001012765235",
    "name": "John Doe",
    "description": "Donation",
    "amount": 100,
    "lastExecutionDate": null,
    "e2eId": "E-f36e4f7e16f95f5960ec9fe00d3c00502",
    "reoccurrence": "MONTHLY",
    "nextOccurrence": "2019-10-01T00:00:00.000+00:00"
  }
]
```

# Timed orders

A timed order is a regular SEPA Credit Transfer which is pre-authorized to be executed once at a specific date.

The flow of timed orders is a 2-step process, divided into 2 requests:

1. Registration of all transaction-relevant data and requesting a TAN
2. Confirmation of the previously registered transfer with the received TAN

## Timed order creation

In order to create a timed order you need at least the following data

| Parameter | Mandatory | Description                                                                    |
| --------- | --------- | ------------------------------------------------------------------------------ |
| recipient | yes       | The name of the recipient                                                      |
| iban      | yes       | Recipient's IBAN                                                               |
| amount    | yes       | The transaction amount in Euro-Cents                                           |
| date      | yes       | The transaction execution time                                                 |
| note      | no        | The booking text which will appear on sender's and recipient's bank statements |
| e2eId     | no        | An optional end-to-end ID such as an invoice ID or booking reference           |

```shell
curl "https://api.kontist.com/api/accounts/4711/timed-orders" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
    "recipient": "John Smith",
    "iban": "DE33100110012626008537",
    "amount": 12012,
    "note": "Bill payment",
    "e2eId": null,
    "date": "2019-08-27T14:32:08.604Z"
  }'
```

> The above command returns JSON structured like this:

```json
{
  "id": "9939482a-b1a2-4cff-8b48-ae340e398659",
  "execute_at": "2019-08-27T12:00:00.000Z",
  "executed_at": null,
  "status": "CONFIRMATION_REQUIRED",
  "scheduled_transaction": {
    "id": "04775196-a148-45ca-8f59-e3bb260a3f38",
    "status": "scheduled",
    "reference": "R-0230af41c289989f97715193772e229f",
    "description": "Bill payment",
    "recipient_iban": "DE33100110012626008537",
    "recipient_name": "John Smith",
    "recipient_bic": "",
    "end_to_end_id": "E-0230af41c289989f97715193772e229f",
    "batch_id": null,
    "created_at": "2019-08-26T14:32:11.426Z",
    "amount": { "value": 12012, "currency": "EUR", "unit": "cents" }
  }
}
```

## Timed order confirmation

As you have noticed, the status of the newly created SEPA credit transfer is being returned as `CONFIRMATION_REQUIRED` therefore you now need to confirm it with the TAN received on your registered mobile phone number.

```shell
curl "https://api.kontist.com/api/accounts/4711/timed-orders/9939482a-b1a2-4cff-8b48-ae340e398659/confirm" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{ "token":"931434" }'
```

> The above command returns JSON structured like this:

```json
{
  "id": "9939482a-b1a2-4cff-8b48-ae340e398659",
  "execute_at": "2019-08-27T12:00:00.000Z",
  "executed_at": null,
  "status": "SCHEDULED",
  "scheduled_transaction": {
    "id": "04775196-a148-45ca-8f59-e3bb260a3f38",
    "status": "scheduled",
    "reference": "R-0230af41c289989f97715193772e229f",
    "description": "Bill payment",
    "recipient_iban": "DE33100110012626008537",
    "recipient_name": "John Smith",
    "recipient_bic": "",
    "end_to_end_id": "E-0230af41c289989f97715193772e229f",
    "batch_id": null,
    "created_at": "2019-08-26T14:32:11.426Z",
    "amount": { "value": 12012, "currency": "EUR", "unit": "cents" }
  }
}
```

## Timed order cancelation

To cancel timed order in `SCHEDULED` state:

```shell
curl "https://api.kontist.com/api/accounts/4711/timed-orders/9939482a-b1a2-4cff-8b48-ae340e398659/cancel" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X PATCH
```

> The above command returns JSON structured like this:

```json
{
  "id": "9939482a-b1a2-4cff-8b48-ae340e398659",
  "execute_at": "2019-08-27T12:00:00.000Z",
  "executed_at": null,
  "status": "CANCELED",
  "scheduled_transaction": {
    "id": "04775196-a148-45ca-8f59-e3bb260a3f38",
    "status": "canceled",
    "reference": "R-496ef67d09d9f725e66810d15336b595",
    "description": "Bill payment",
    "recipient_iban": "DE33100110012626008537",
    "recipient_name": "John Smith",
    "recipient_bic": "",
    "end_to_end_id": "E-496ef67d09d9f725e66810d15336b595",
    "batch_id": null,
    "created_at": "2019-08-26T14:39:41.699Z",
    "amount": { "value": 12012, "currency": "EUR", "unit": "cents" }
  }
}
```

Timed order will not be displayed anymore in fetch timed orders response.

## Fetch timed orders

To fetch list of timed order in `SCHEDULED` state:

```shell
curl "https://api.kontist.com/api/accounts/4711/timed-orders" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": "9939482a-b1a2-4cff-8b48-ae340e398659",
    "recipientName": "John Smith",
    "recipientIban": "DE33100110012626008537",
    "date": "2019-08-27T12:00:00.000Z",
    "purpose": "Bill payment",
    "amount": -12012,
    "e2eId": "E-d447d47fb16d5e0dffe7c7894cc643cf",
    "status": "scheduled",
    "type": "TIMED_ORDER"
  }
]
```
