---
title: Switch API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='http://github.com/tripit/slate'>Documentation Powered by Slate</a>

search: true
---

# Introduction

Welcome to the Switch API documentation. 

The API follows a [REST](http://en.wikipedia.org/wiki/Representational_state_transfer) architecture, where endpoints are built around the concept of resources, actions are represented by the respective HTTP verb and response status are represented using HTTP status codes.

```
API Endpoint
https://api.switchpayments.com/v1

```

All requests must be made over HTTPS.

## Resources

> Resources List

```json
{
    "url": "https://api.switchpayments.com/v1",
    "resources": {
        "cards": {
            "url": "https://api.switchpayments.com/v1/cards"
        },
        "tokens": {
            "url": "https://api.switchpayments.com/v1/tokens"
        },
        "customers": {
            "url": "https://api.switchpayments.com/v1/customers"
        },
        "payments": {
            "url": "https://api.switchpayments.com/v1/payments"
        }
    }
}
```

The API root URL provides a list of the available top-level resources and their respective URL.

## Responses

Every API call response is a JSON encoded top-level object.

> Resource Collection

```json
{
    "pagination": {},
    "filters": {},
    "collection": []
}
```

- **Resource Collections** - The response is an *envelope* as demonstrated on the right.
    - `pagination` - An object containing information regarding the pagination of the results.
    - `filters`- An object containing the parameters and respective values that effectively filtered the results collection.
    - `collection` - An array containing the resource item objects for the given page of results.
- **Resource Items** - The object returned represents the item requested.

## Pagination

When a resource collection result is paginated, additional _query parameters_ can be passed in order to request a specific results page or to set the number of results per page.

### QUERY PARAMETERS

Parameter | Default | Description
--------- | ------- | -----------
page      | -       | The desired result page.
per_page  | 30      | The number of results per page.

## Errors

Errors with the API are reported first using the appropriate [HTTP status code](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes) and, if applicable, additional details are provided in the response body in a JSON encoded object.

Error Code | Description  | Meaning
---------- | -----------  | -------
400        | Bad Request  | Missing or invalid request parameters.
401        | Unauthorized | No credentials were provided or credentials provided are invalid.
403        | Forbidden    | You don't have the necessary permissions to access the given resource.
404        | Not Found    | The requested item doesn't exist.
500        | Server Error | Something unexpected happened on the server side.

### Bad Request 

> Bad Request Response

```json
{
    "message": "Generic error detail",
    "parameters": {
        "foo": ["Detail 1 for foo", "Detail 2 for foo"],
        "bar": ["Detail 1 for bar"]
    }
}
```

When a request fails because of a bad request, a generic message describing the error is always provided and, when the error is with specific parameters, for each one, an array of detailed messages is provided.

# Authentication

> Example Request

```shell
$ curl -vX GET "https://api.switchpayments.com/v1/customers" -u "username:password"
```

All the API endpoints require authentication and the respective credentials should be provided via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication):

`Authorization: Basic {credentials}`

You must replace {credentials} with the base64 encoded string in the form of "username:password", where username is your _Account ID_ and password your _Private Key_.

<aside class="notice">
Your credentials should NEVER be shared as they unlock sensitive resources, such as payments.
</aside>

# Payments

### The payment object

> Example Object

```json
{
    "id": "5affd688507a7ed35fac7c324862fdcc7d55099e549ee32f",
    "amount": 100,
    "amount_captured": 80,
    "amount_refunded": 0,
    "capture_on_creation": false,
    "captured": true,
    "captures": [
      {
        "amount": 80,
        "created_at": "2014-12-28 15:14:13",
        "success": false
      },
      {
        "amount": 80,
        "created_at": "2014-12-28 15:14:15",
        "success": true
      }
    ],
    "card": {
        "cvc": "007",
        "expiration_month": 3,
        "expiration_year": 2016,
        "name": "John Doe",
        "number": "1111222233334444"
    },
    "created_at": "2014-12-27 16:49:51",
    "currency": "EUR",
    "description": "",
    "refunds": [],
    "success": true,
    "updated_at": "2014-12-27 16:49:51",
    "voided": false,
    "voids": []
}
```

Attribute           | Type     | Description
------------------- | -------- | -----------
id                  | string   | Unique identifier of the payment.
amount              | float    | Requested payment amount.
amount_captured     | float    | Amount captured.
amount_refunded     | float    | Amount refunded.
capture_on_creation | boolean  | If 'false', then payment will be an authorization that will have to be either captured or voided.
captured            | boolean  | Whether or not the payment was captured.
captures            | array    | Capture requests for given payment.
card                | object   | If 'capture_on_creation' is 'true', then this must be a valid Card ID string. If not, then payment is an authorization and the card's details, including CVC, are required and a dict containg them is expected.
created_at          | string   | Date and time of the creation of the payment.
currency            | string   | Currency in which the payment was made.
description         | string   | Description of the payment.
refunds             | array    | Refunds made for this payment.
success             | boolean  | Whether or not the payment (either process payment or authorization) were successful.
updated_at          | string   | Date and time of the last time an update was made to the payment.
voided              | boolean  | In the case of payment being an Authorization, whether or not it was voided.
voids               | array    | Voids requested for this payment.

## Create a new payment

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v1/payments" \
       -u "username:password" \
       -d '{"amount": 42, "capture_on_creation": true, "currency": "EUR", "card": "8135032a4dc488116a25dcca63b251af727dcca6549ee2a2"}'
```

`POST https://api.switchpayments.com/v1/payments`

### Request Parameters

Parameter           | Type     | Required | Description
------------------- | -------- | -------- | -----------
amount              | float    | yes      | A positive integer in the smallest currency unit.
capture_on_creation | boolean  | yes      | If 'true' then payment is immediately captured and if 'false' then an Authorization is requested. 
card                |          | no       | Either the ID of an existing card or a [dictionary](#create-a-new-card) containing a credit card's details, depending on 'capture_on_creation' being 'true' and 'false', respectively.
currency            | string   | yes      | Three-letter ISO currency code representing the currency in which the payment should be made.
description         | string   | no       | A description of the payment.

### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the payment was created, with the respective [object](#payments). If something goes wrong, an [error](#errors) is returned.

## Capture a payment

You can only capture a payment that has an _active_ authorization.

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v1/payments/cd501d5d9a68fea10f2926562c0593c24634d68854ad4e64/capture" \
       -u "username:password" \
       -d '{"amount": 40}'
```

`POST https://api.switchpayments.com/v1/payments/{id}/capture`

### Request Parameters

Parameter | Required | Description
--------- | -------- | -----------
amount    | yes      | The amount to be captured (must be equal or less of the payment's total).

### Returns

If the request succeeded, then `HTTP 201` is returned. If something goes wrong, an [error](#errors) is returned.

## Void a payment

You can only void a payment that has an _active_ authorization.

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v1/payments/cd501d5d9a68fea10f2926562c0593c24634d68854ad4e64/void" \
       -u "username:password" \
       -d '{}'
```

`POST https://api.switchpayments.com/v1/payments/{id}/void`

### Returns

If the request succeeded, then `HTTP 201` is returned. If something goes wrong, an [error](#errors) is returned.

## Refund a payment

You can only make refunds of a payment that has been _captured_ and until the total refund value amounts to the payment's total value.

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v1/payments/cd501d5d9a68fea10f2926562c0593c24634d68854ad4e64/refund" \
       -u "username:password" \
       -d '{"amount": 40, "description": "Client did not like the product"}'
```

`POST https://api.switchpayments.com/v1/payments/{id}/refund`

### Request Parameters

Parameter   | Required | Description
----------- | -------- | -----------
amount      | yes      | The amount to be refunded.
description | no       | Description of the refund.

### Returns

If the request succeeded, then `HTTP 201` is returned. If something goes wrong, an [error](#errors) is returned.

## List all payments

### HTTP Request

> Example Request

```shell
$ curl -vX GET "https://api.switchpayments.com/v1/payments" \
       -u "username:password"
```

`GET https://api.switchpayments.com/v1/payments`

# Cards

### The card object

> Example Object

```json
{
    "id": "8135032a4dc488116a25dcca63b251af727dcca6549ee2a2",
    "expiration_month": 12,
    "created_at": "2014-12-27 16:47:30",
    "funding": "credit",
    "name": "John Doe",
    "expiration_year": 2042,
    "brand": "visa",
    "status": "ok",
    "last_4_digits": "0022"
}
```

Attribute        | Type     | Description
---------------- | -------- | -----------
id               | string   | Unique identifier of the card.
brand            | string   | Card brand.
funding          | string   | Card funding type.
last_4_digits    | string   | Last four digits of the card number.
name             | string   | Cardholder name.
expiration_month | int      | Card expiration month.
expiration_year  | int      | Card expiration year.
created_at       | string   | Date and time of the creation of the card.
status           | string   | Status of the card ("ok" means it is ready to be used).

## Create a new card

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v1/cards" \
       -u "username:password" \
       -d '{"card": "4360eeff94bda5bf0040cd9b907cbadc1008e36154ad319f"}'
```

`POST https://api.switchpayments.com/v1/cards`

### Request Parameters

Parameter | Required | Description
--------- | -------- | -----------
card      | yes      | Can be a [token](#create-a-new-token) or a dictionary containing a credit card's details (see bellow).

### Credit Card Details

Parameter        | Required | Description
---------------- | -------- | -----------
number           | yes      | The card's number.
expiration_month | yes      | The card's expiration month.
expiration_year  | yes      | The card's expiration year.
cvc              | yes      | The card's security code.
name             | yes      | The cardholder name.
customer_id      | no       | The ID of an existing [customer](#customers). If this value is provided, the card's owner will be the given customer.

### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the card was created, with the respective [object](#cards). If something goes wrong, an [error](#errors) is returned.

## Update a card

### HTTP Request

> Example Request

```shell
$ curl -vX PUT "https://api.switchpayments.com/v1/cards/8135032a4dc488116a25dcca63b251af727dcca6549ee2a2" \
       -u "username:password" \
       -d '{"customer_id": "ef40a7a20144b23b3b82a82f86e3005e0fb6e07654ad265e"}'
```

`PUT https://api.switchpayments.com/v1/cards/{id}`

### Request Parameters

Parameter   | Required | Description
----------- | -------- | -----------
customer_id | yes      | The ID of an existing [customer](#customers). If this value is provided, the card's owner will be the given customer. If the value is empty, then the card will have no customer associated with it.

### Returns

If the request succeeded, then `HTTP 200` is returned. If something goes wrong, an [error](#errors) is returned.

## List all cards

### HTTP Request

> Example Request

```shell
$ curl -vX GET "https://api.switchpayments.com/v1/cards" \
       -u "username:password"
```

`GET https://api.switchpayments.com/v1/cards`

# Customers

### The customer object

> Example Object

```json
{
    "id": "ef40a7a20144b23b3b82a82f86e3005e0fb6e07654ad265e",
    "description": "The best client in the world!",
    "created_at": "2015-01-07 12:28:14",
    "updated_at": "2015-01-07 12:28:14",
    "email": "john@example.com",
    "cards": {
        "url": "/v1/customers/ef40a7a20144b23b3b82a82f86e3005e0fb6e07654ad265e/cards",
        "total_items": 0,
        "default": null
    },
    "name": "John Doe"
}
```

Attribute        | Type     | Description
---------------- | -------- | -----------
id               | string   | Unique identifier of the customer.
email            | string   | The customer's email address.
name             | string   | The customer's full name.
description      | string   | A description about the customer.
created_at       | string   | Date and time of the creation of the customer.
updated_at       | string   | Date and time of the last update to the customer's details.
cards            | object   | A dictionary with a detailed summary regarding customer's cards (e.g. total cards, default card) and the URL to the respective resource collection.

## Create a new customer

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v1/customers" \
       -u "username:password" \
       -d '{"email": "john@example.com", "name": "John Doe", "description": "The best client in the world!"}'
```

`POST https://api.switchpayments.com/v1/customers`

### Request Parameters

Parameter   | Required | Description
----------- | -------- | -----------
email       | yes      | The customer's email address.
name        | no       | The customer's full name.
description | no       | A description of the customer.
card        | no       | The ID of an existing card (that cannot belong to a customer already) or a [dictionary](#create-a-new-card) containing a credit card's details. This will be set as the user's default card.
token       | no       | A card [token](#tokens). This will be set as the user's default card.

<aside class="notice">
Regarding the default card, either a card OR token should be provided and not both.
</aside>

### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the card was created, with the respective [object](#customers). If something goes wrong, an [error](#errors) is returned.

## Update a customer

### HTTP Request

> Example Request

```shell
$ curl -vX PUT "https://api.switchpayments.com/v1/customers/ef40a7a20144b23b3b82a82f86e3005e0fb6e07654ad265e" \
       -u "username:password" \
       -d '{"default_card": "8135032a4dc488116a25dcca63b251af727dcca6549ee2a2"}'
```

`PUT https://api.switchpayments.com/v1/customers/{id}`

### Request Parameters

Parameter    | Required | Description
------------ | -------- | -----------
email        | yes      | The customer's email address.
name         | no       | The customer's full name.
description  | no       | A description of the customer.
default_card | no       | The ID of an existing [card](#cards) (should either don't have a customer of belong to the given customer) that will be set as the customer's default card.

### Returns

If the request succeeded, then `HTTP 200` is returned. If something goes wrong, an [error](#errors) is returned.

## List a customer's cards

### HTTP Request

> Example Request

```shell
$ curl -vX GET "https://api.switchpayments.com/v1/customers/ef40a7a20144b23b3b82a82f86e3005e0fb6e07654ad265e/cards" \
       -u "username:password"
```

`GET https://api.switchpayments.com/v1/customers/{id}/cards`

# Tokens

### The token object

> Example Object

```json
{
    "type": "card",
    "value": "4360eeff94bda5bf0040cd9b907cbadc1008e36154ad319f"
}
```

Attribute        | Type     | Description
---------------- | -------- | -----------
type             | string   | The type of token.
value            | string   | The token itself, that should be provided in any request that requires a token.

## Create a new token

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v1/tokens" \
       -d '{"type": "card"}'
```

`POST https://api.switchpayments.com/v1/tokens`

### Request Parameters

Parameter   | Required | Description
----------- | -------- | -----------
type        | yes      | The type of token (possible values: "card")

### Credit Card Details

If the token is of the **card** type, additional parameters must be provided with the request.

Parameter        | Required | Description
---------------- | -------- | -----------
number           | yes      | The card's number.
expiration_month | yes      | The card's expiration month.
expiration_year  | yes      | The card's expiration year.
cvc              | yes      | The card's security code.
name             | yes      | The cardholder name.

### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the card was created, with the respective [object](#tokens). If something goes wrong, an [error](#errors) is returned.