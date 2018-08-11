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
        "bankName": "solarisBank"
        "balance": 17325
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
      "to": null,
      "amount": 659,
      "type": null,
      "category": null,
      "name": "Vae Zrniv4chmon1; St Louis; Fr",
      "iban": null,
      "purpose": null,
      "e2e_id": null,
      "booking_date": "2018-08-08T16:26:52+00:00",
      "valuta_date": null,
      "booking_type": null,
      "original_amount": "760",
      "foreign_currency": "CHF",
      "paymentMethod": "credit_card"
    },
    {
      "id": 905800,
      "from": 4711,
      "to": null,
      "amount": 50433,
      "type": null,
      "category": null,
      "name": "Karl Lagerfeld",
      "iban": "DE35120300001012765235",
      "purpose": "E-16ab84e3177fdb4f692e02fd5b8ce808",
      "e2e_id": "E-16ab84e3177fdb4f692e02fd5b8ce808",
      "booking_date": "2018-08-06T22:00:00+00:00",
      "valuta_date": "2018-08-06T22:00:00+00:00",
      "booking_type": "SEPA_CREDIT_TRANSFER",
      "original_amount": null,
      "foreign_currency": null,
      "paymentMethod": "bank_account"
    }
  ]
}
```

This endpoint retrieves all transactions of the authenticated user.

### HTTP Request

`GET https://api.kontist.com/api/accounts/{accpunt_id}/transactions`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
page | 1 | If set, then indicating the page of the result

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

