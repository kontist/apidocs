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

## Device Binding

To have access to Kontist API endpoints that require strong customer authentication, you need to pass two-factor authentication (2FA).

To make it seamless for you we provide device binding authentication that leverages digital signature algorithm.

To pass device binding authentication you need to generate a pair of private and public keys by using elliptic curve algorithm  (secp256r1). To create and verify your device you need to pass your public key and then sign OTP from SMS received on your phone.

After that, you can use your device id and your private key to get your confirmed auth token by creating and verifying device challenge.

### Create device

To initiate the device binding you need to create a device. After a successful request, you will receive an SMS with OTP that you will need during device verification.

```shell
curl "https://api.kontist.com/api/user/devices" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
        "name": "iPhone XS",
        "key": "0402e86575939cd541f016b69b1bc6ee97736f7a6d32c0ad375695ffdc03acf21a3b54224fd164ad6f9cfdfb42b74f49f3d34a41f95d62e893be4977c7ec154f29"
      }'
```

> The above command returns JSON structured like this:

```json
{
  "deviceId": "4e310a55-1b1a-4efb-b9a5-fd04491bdd21",
  "challengeId": "4e310a55-1b1a-4efb-b9a5-fd04491bdd21"
}
```

#### HTTP Request

`POST https://api.kontist.com/api/user/devices`

#### Request body

| Parameter | Mandatory | Description                                                                    |
| --------- | --------- | ------------------------------------------------------------------------------ |
| name      | yes       | The name of the device                                                         |
| key       | yes       | The hex-encoded public key without header                                      |

#### Response
| Field       | Description                                                                    |
| ----------- | ------------------------------------------------------------------------------ |
| deviceId    | ID of the device                                                               |
| challengeId | ID of the challenge                                                            |

### Verify device

To verify device you need to provide a signature of OTP received on your mobile phone.

```shell
curl "https://api.kontist.com/api/user/devices/4e310a55-1b1a-4efb-b9a5-fd04491bdd21/verify" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
        "challengeId": "4e310a55-1b1a-4efb-b9a5-fd04491bdd21",
        "signature": "30440220220B71BA03178A43B6CFA766F1B520CA1A626777F76B21253F9EC5039F4A0EB3022043CF2685C8F695F434862EADD1D5F5D6F68C29E875F755D058070A71E8338E11"
      }'
```

> The above command returns `204 No Content` in case of success.

#### HTTP Request

`POST https://api.kontist.com/api/user/devices/{device_id}/verify`

#### Request body

| Parameter   | Mandatory | Description                                                                    |
| ----------- | --------- | ------------------------------------------------------------------------------ |
| challengeId | yes       | ID of the challenge recieved during device creation                            |
| signature   | yes       | The hex-encoded signature for the OTP recived in SMS                           |

### Create device challenge

After the device is created and verified, you need to create a device challenge. As a response, you will receive `stringToSign` that should be used during verification of device challenge.

```shell
curl "https://api.kontist.com/api/user/devices/4e310a55-1b1a-4efb-b9a5-fd04491bdd21/challenges" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -X POST
```

> The above command returns JSON structured like this:

```json
{
  "id": "5f7c36e2-e0bf-4755-8376-ac6d0711192e",
  "stringToSign": "9e2d45df-9b00-49d3-9064-29b86374fe67"
}
```

#### HTTP Request

`POST https://api.kontist.com/api/user/devices/{device_id}/challenges`

#### Response
| Field         | Description                                                                    |
| ------------- | ------------------------------------------------------------------------------ |
| id            | ID of the challenge                                                            |
| stringToSign  | Challenge string that should be signed by device private key                   |


### Verify device challenge

To verify device challenge you need to provide a signature of `stringToSign` received after challenge creation.

```shell
curl "https://api.kontist.com/api/user/devices/4e310a55-1b1a-4efb-b9a5-fd04491bdd21/challenges/5f7c36e2-e0bf-4755-8376-ac6d0711192e/verify" \
  -H "Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l" \
  -H "Content-Type: application/json" \
  -X POST \
  -d '{
        "signature": "FF93DF062DB808E35AB9D28D80E0B261C7313C69785471954C05A474156BA7A7F07F2F0E7E805513754A8119BBF172E1E6D0103901249CE8DE012E5E61FDA36AD06405341043"
      }'
```

> The above command returns JSON structured like this:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI4ODNjNTc4ZS01M2QwLTRhYmEtOTBiNC02MmRmZmFkNTE5NTMiLCJzY29wZSI6ImF1dGgiLCJjbmYiOnsia2lkIjoiMmExNjRlYzYtZTJkNC00OTI4LTk5NDItZDU5YWI2Yzc4ZDU5In0sImlhdCI6MTU2NzQwOTExNSwiZXhwIjoxNTY3NDEyNzE1fQ.m35NDpQMAB5DMebXUxEzWupP3i-iAwoyVy2sGF1zp_8"
}
```


#### HTTP Request

`POST https://api.kontist.com/api/user/devices/{device_id}/challenges/{challenge_id}/verify`

#### Request body

| Parameter   | Mandatory | Description                                                                    |
| ----------- | --------- | ------------------------------------------------------------------------------ |
| signature   | yes       | The hex-encoded signature for the `stingToSign`                                |

#### Response
| Field         | Description                                                                                                      |
| ------------- | ---------------------------------------------------------------------------------------------------------------- |
| token         | Auth token with confirmation claim that should be used for endpoints that require strong customer authentication |

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
