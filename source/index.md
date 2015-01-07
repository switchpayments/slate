---
title: API Reference

language_tabs:
  - shell
  - javascript
  - python

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
    "currency": "EUR",
    "amount": 100,
    "authorizations": [
        {
            "status": "active",
            "authorized_at": "2014-12-27 16:50:23"
        }
    ],
    "description": "",
    "captured": false,
    "refunded": false,
    "created_at": "2014-12-27 16:49:51",
    "card": {
        "id": "8135032a4dc488116a25dcca63b251af727dcca6549ee2a2",
        "expiration_month": 12,
        "created_at": "2014-12-27 16:47:30",
        "funding": "credit",
        "name": "John Doe",
        "expiration_year": 2042,
        "brand": "visa",
        "status": "ok",
        "last_4_digits": "0022"
    },
    "updated_at": "2014-12-27 16:49:51"
}
```

Attribute      | Type     | Description
-------------- | -------- | -----------
id             | string   | Unique identifier of the payment.
card           | object   | Details of the card used to make the payment.
amount         | int      | Payment amount.
currency       | string   | Currency in which the payment was made.
description    | string   | Description of the payment.
created_at     | string   | Date and time of the creation of the payment.
updated_at     | string   | Date and time of the last time an update was made to the payment.
captured       | boolean  | Whether or not the payment was captured.
authorizations | array    | Authorizations requested for this payment.
refunded       | boolean  | Whether any refunds were made for this payment.
refunds        | array    | Refunds made for this payment.

## Create a new payment

### HTTP Request

`POST https://api.switchpayments.com/v1/payments`

### Request Parameters

Parameter | Required | Description
--------- | -------- | -----------
card      | no       | The ID of an existing card or a [dictionary](#create-a-new-card) containing a credit card's details.
customer  | no       | The ID of an existing customer whose default card will be used in this request.

<aside class="notice">
Either a customer or a card must be provided.
</aside>

### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the payment was created, with the respective [object](#payments). If something goes wrong, an [error](#errors) is returned.

## Authorize a payment

This is the step that indeed requests that the given payment is authorized to be captured.

### HTTP Request

`POST https://api.switchpayments.com/v1/payments/{id}/authorize`

### Returns

If the request succeeded, then `HTTP 201` is returned. If something goes wrong, an [error](#errors) is returned.

<aside class="warning">
Don't forget to either capture or void an authorization.
</aside>

## Capture a payment

You can only capture a payment that has an _active_ authorization.

### HTTP Request

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

`POST https://api.switchpayments.com/v1/payments/{id}/void`

### Returns

If the request succeeded, then `HTTP 201` is returned. If something goes wrong, an [error](#errors) is returned.

## Refund a payment

You can only make refunds of a payment that has been _captured_ and until the total refund value amounts to the payment's total value.

### HTTP Request

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

`PUT https://api.switchpayments.com/v1/cards/{id}`

### Request Parameters

Parameter   | Required | Description
----------- | -------- | -----------
customer_id | yes      | The ID of an existing [customer](#customers). If this value is provided, the card's owner will be the given customer. If the value is empty, then the card will have no customer associated with it.

### Returns

If the request succeeded, then `HTTP 200` is returned. If something goes wrong, an [error](#errors) is returned.

## List all cards

### HTTP Request

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

## Update a customer

## List a customer's cards

## Update a customer's card

# Tokens

## Create a new token