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
https://api.switchpayments.com/v2

```

All requests must be made over HTTPS.

## Resources

> Resources List

```json
{
    "url": "https://api.switchpayments.com/v2",
    "resources": {
        "charges": {
            "url": "https://api.switchpayments.com/v2/charges"
        },
        "instruments": {
            "url": "https://api.switchpayments.com/v2/instruments"
        },
        "customers": {
            "url": "https://api.switchpayments.com/v2/customers"
        },
        "events": {
            "url": "https://api.switchpayments.com/v2/events"
        },
        "payments": {
            "url": "https://api.switchpayments.com/v2/payments"
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
$ curl -vX GET "https://api.switchpayments.com/v2/customers" -u "username:password"
```

All the API endpoints require authentication and the respective credentials should be provided via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication):

`Authorization: Basic {credentials}`

You must replace {credentials} with the base64 encoded string in the form of "username:password"

### Public Credentials

Resources that require the Merchant's public credentials for authentication, username is _Account ID_ and password is blank.

### Private Credentials

Resources that required the Merchant's private credentials for authentication, username is your _Account ID_ and password your _Private Key_.

<aside class="notice">
Your private credentials should NEVER be shared as they unlock sensitive resources, such as payments.
</aside>

# Charges

### The charge object

> Example Object

```json
{
  "id": "06d7c2e4145f3be209e9ab5c6ed24da8b786f65256d462a5",
  "charge_type": "multibanco",
  "confirmed": true,
  "amount": 23,
  "currency": "EUR",
  "events_url": "https://localhost:8005/my-events-url",
  "metadata": {
    "user": 42
  },
  "instruments": [{
    "id": "8fa25ce7f0d4f74cab86d23bfe1350ce2bac2bc556d462aa",
    "success": true,
      "last_payment": {
        "id": "69c2a925a7a7d5d4d10e333b4d34442053b5d7c756d462bf",
        "success": true
      }
  }],
  "created_at": "2016-02-29T15:24:21.052021+00:00",
  "updated_at": "2016-02-29T15:24:25.976851+00:00",
  "expires_at": "2016-02-29T15:29:21+00:00",
  "used_at": null
}
```

Attribute        | Type     | Description
---------------- | -------- | -----------
id               | string   | Unique identifier of the charge.
charge_type      | string   | The payment [type](#charge-types)
confirmed        | boolean  | Is the charge confirmed and can be used to create an instrument
amount           | float    | The amount that will be charged
currency         | string   | The currency of the charge
events_url       | string   | The webhook URL to which all related events should be sent
metadata         | url      | Metadata that the Merchant wishes to attach to the charge (e.g. order ID)
instruments      | array    | Array of [Instruments](#instruments) (only one successful instrument per charge)
created_at       | string   | Date and time of the creation of the charge.
updated_at       | string   | Date and time of the last time the charge was updated
expires_at       | string   | Date and time until when the charge is valid
used_at          | string   | Date and time when the charge was used

## Create a new charge

A Charge is created by the Client using the Merchant's [public credentials](#public-credentials).

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v2/charges" \
       -u "username:password" \
       -d '{"charge_type": "sofort", "amount": 42, "currency": "EUR", "metadata": {"orderId": "1337"}}'
```

`POST https://api.switchpayments.com/v2/charges`

### Request Parameters

Parameter   | Required | Description
----------- | -------- | -----------
charge_type | yes      | The type of payment
amount      | yes      | The amount that will be charged when creating the payment
currency    | yes      | The currency of the amount being charged
metadata    | yes      | A JSON object containing metadata that will allow the merchant to link the charge with, for example, an order


### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the charge was created, with the respective [object](#charges). If something goes wrong, an [error](#errors) is returned.

## Confirm a charge

In order for a Charge to be used to create an [Instrument](#instruments), it has to be confirmed by the Merchant using its [private credentials](#private-credentials).

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v2/charges/8135032a4dc488116a25dcca63b251af727dcca6549ee2a2/confirm" \
       -u "username:password" \
       -d '{}'
```

`POST https://api.switchpayments.com/v2/charges/{id}/confirm`

### Returns

If the request succeeded, then `HTTP 201` is returned. If something goes wrong, an [error](#errors) is returned.

## List all charges

List the Merchant's charges using it's [private credentials](#private-credentials).

### HTTP Request

> Example Request

```shell
$ curl -vX GET "https://api.switchpayments.com/v2/charges" \
       -u "username:password"
```

`GET https://api.switchpayments.com/v2/charges`

## Charge Types

List the charge types available for the Merchant using it's [public credentials](#public-credentials).

For each of the available charge types, additional information is provided such as if it is possible to make recurring payments
and the schema of the required and optional parameters when creating a new instrument.

### HTTP Request

> Example Object

```json
{
  "collection": [
    {
      "label": "Referência Multibanco",
      "recurring": false,
      "id": "multibanco",
      "schema": {
        "required:": [],
        "type": "object",
        "properties": {
          "start_date": {
            "type": "string"
          },
          "end_date": {
            "type": "string"
          }
        }
      }
    },
    {
      "label": "MBWay",
      "recurring": false,
      "id": "mbway",
      "schema": {
        "required": [
          "phone"
        ],
        "type": "object",
        "properties": {
          "phone": {
            "type": "string"
          }
        }
      }
    },
    {
      "label": "PayPal",
      "recurring": false,
      "id": "paypal",
      "schema": null
    }
  ]
}
```

`GET https://api.switchpayments.com/v2/charges/types`

# Instruments

### The instrument object

> Example Object

```json
{
  "id": "5bdd065237b6b17162d1eb49331b4567825da91156d4ee16",
  "success": true,
  "redirect": {
    "url": null,
    "parameters": {
      "reference": "000",
      "value": "00"
    }
  },
  "params": {
    "phone": "910000000"
  },
  "used": false,
  "last_payment": null,
  "customer": null,
  "charge": {
    "charge_type": "mbway",
    "id": "77f372c7e3e04c4cd4a2b4c036c7e05a4b17662956d4ed0d"
  },
  "created_at": "2016-03-01T01:19:18.822445+00:00",
  "updated_at": "2016-03-01T01:19:18.822494+00:00"
}
```

Attribute        | Type     | Description
---------------- | -------- | -----------
id               | string   | The unique ID of the instrument
success          | string   | If the instrument was created successfully and can be used to create payments

## Create a new instrument

After a [Charge](#charges) created by the Client is confirmed by the Merchant, it is possible for the Client to create a
Payment Instrument that will be used to create single or recurring payments.

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v2/instruments" \
       -d '{"charge": "06d7c2e4145f3be209e9ab5c6ed24da8b786f65256d462a5"}'
```

`POST https://api.switchpayments.com/v2/instruments`

### Request Parameters

Parameter   | Required | Description
----------- | -------- | -----------
charge      | yes      | The ID of the charge

### Instrument Details

Additional parameters may be required, according to the [charge type](#charge-types).

### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the instrument was created, with the respective [object](#instruments). If something goes wrong, an [error](#errors) is returned.

## Archive an instrument

When an instrument is no longer required, it should be archived. For example, if the charge type is a Card Authorization,
archiving the instrument will also make sure the authorization is voided and the respective funds released.

### HTTP Request

> Example Request

```shell
$ curl -vX DELETE "https://api.switchpayments.com/v2/instruments/06d7c2e4145f3be209e9ab5c6ed24da8b786f65256d462a5"
```

`DELETE https://api.switchpayments.com/v2/instruments/{id}`


### Returns

If the request succeeded, then `HTTP 204` is returned, meaning that the instrument was archived. If something goes wrong, an [error](#errors) is returned.

# Payments

### The payment object

> Example Object

```json
{
  "id": "323c4771dc518d79cae787bb2c5963474bb441ab56d4ddc8",
  "success": true,
  "amount": 1,
  "currency": "EUR",
  "description": "",
  "charge": {
    "charge_type": "card_recurring",
    "charge_type_label": "Card Recurring",
    "id": "7569ac2f4e4ab8e3bdpb96196b3798d60e75e47b56d4610a"
  },
  "instrument": {
    "id": "d8a2d0abb9fx972aea201fd805366dda6b83671b56d46112"
  },
  "refunds": [
    {
      "created_at": "2016-03-01T00:26:36.059632+00:00",
      "amount": 1,
      "description": "",
      "success": true
    }
  ],
  "created_at": "2016-03-01T00:09:46.729628+00:00",
  "updated_at": "2016-03-01T00:09:46.729672+00:00"
}
```

Attribute           | Type     | Description
------------------- | -------- | -----------
id                  | string   | Unique identifier of the payment.
success             | boolean  | Whether or not the payment was successful.
amount              | float    | Amount captured.
currency            | string   | Currency in which the payment was made.
description         | string   | Description of the payment.
charge              | object   | Details of the charge associated with the payment.
instrument          | object   | Details of the instrument used to create the payment.
refunds             | array    | Refunds made for this payment.
created_at          | string   | Date and time of the creation of the payment.
updated_at          | string   | Date and time of the last time an update was made to the payment.

## Create a new payment

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v2/payments" \
       -u "username:password" \
       -d '{"instrument": "d8a2d0abb9fx972aea201fd805366dda6b83671b56d46112", "amount": 42, "currency": "EUR"}'
```

`POST https://api.switchpayments.com/v2/payments`

### Request Parameters

Parameter           | Type     | Required | Description
------------------- | -------- | -------- | -----------
instrument          | string   | yes      | The ID of the payment instrument.
amount              | float    | yes      | A positive integer in the smallest currency unit.
currency            | string   | yes      | Three-letter ISO currency code representing the currency in which the payment should be made.
description         | string   | no       | A description of the payment.

### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the payment was created, with the respective [object](#payments). If something goes wrong, an [error](#errors) is returned.

## Refund a payment

You can only make refunds of a payment that has been _captured_ and until the total refund value amounts to the payment's total value.

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v2/payments/323c4771dc518d79cae787bb2c5963474bb441ab56d4ddc8/refund" \
       -u "username:password" \
       -d '{"amount": 40, "description": "Client did not like the product"}'
```

`POST https://api.switchpayments.com/v2/payments/{id}/refund`

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
$ curl -vX GET "https://api.switchpayments.com/v2/payments" \
       -u "username:password"
```

`GET https://api.switchpayments.com/v2/payments`

# Customers

### The customer object

> Example Object

```json
{
  "id": "ef40a7a20144b23b3b82a82f86e3005e0fb6e07654ad265e",
  "name": "John Doe",
  "email": "john@example.com",
  "description": "The best client in the world!",
  "created_at": "2015-01-07 12:28:14",
  "updated_at": "2015-01-07 12:28:14"
}
```

Attribute        | Type     | Description
---------------- | -------- | -----------
id               | string   | Unique identifier of the customer.
name             | string   | The customer's full name.
email            | string   | The customer's email address.
description      | string   | A description about the customer.
created_at       | string   | Date and time of the creation of the customer.
updated_at       | string   | Date and time of the last update to the customer's details.

## Create a new customer

### HTTP Request

> Example Request

```shell
$ curl -vX POST "https://api.switchpayments.com/v2/customers" \
       -u "username:password" \
       -d '{"email": "john@example.com", "name": "John Doe", "description": "The best client in the world!"}'
```

`POST https://api.switchpayments.com/v2/customers`

### Request Parameters

Parameter   | Required | Description
----------- | -------- | -----------
email       | yes      | The customer's email address.
name        | no       | The customer's full name.
description | no       | A description of the customer.

### Returns

If the request succeeded, then `HTTP 201` is returned, meaning that the customer was created, with the respective [object](#customers). If something goes wrong, an [error](#errors) is returned.

## Update a customer

### HTTP Request

> Example Request

```shell
$ curl -vX PUT "https://api.switchpayments.com/v2/customers/ef40a7a20144b23b3b82a82f86e3005e0fb6e07654ad265e" \
       -u "username:password" \
       -d '{"name": "John"}'
```

`PUT https://api.switchpayments.com/v2/customers/{id}`

### Request Parameters

Parameter    | Required | Description
------------ | -------- | -----------
email        | yes      | The customer's email address.
name         | no       | The customer's full name.
description  | no       | A description of the customer.

### Returns

If the request succeeded, then `HTTP 200` is returned. If something goes wrong, an [error](#errors) is returned.

### Returns

If the request succeeded, then `HTTP 204` is returned, meaning that the instrument was archived. If something goes wrong, an [error](#errors) is returned.

# Events

### The event object

> Example Object

```json
{
  "id": "5d59645e6f40da00230be601300920628d918a0856d4ee16",
  "type": "instrument.success",
  "result_status": -1,
  "created_at": "2016-03-01T01:19:18.829914+00:00",
  "instrument": {
    "customer": null,
    "used": false,
    "updated_at": "2016-03-01T01:19:18.822494+00:00",
    "id": "5bdd065237b6b17162d1eb49331b4567825da91156d4ee16",
    "last_payment": null,
    "redirect": {
      "url": null,
      "parameters": {
        "reference": "000",
        "value": "000"
      }
    },
    "success": true,
    "created_at": "2016-03-01T01:19:18.822445+00:00",
    "charge": {
      "charge_type": "mbway",
      "charge_type_label": "MBWay",
      "id": "77f372c7e3e04c4cd4a2b4c036c7e05a4b17662956d4ed0d"
    },
    "params": {
      "phone": "910000000"
    }
  },
  "charge": {
    "events_url": "https://localhost:8001/default_awesome_webhook_url",
    "updated_at": "2016-03-01T01:16:26.615584+00:00",
    "expires_at": "2016-03-01T01:19:53+00:00",
    "charge_type_label": "MBWay",
    "used_at": null,
    "id": "77f372c7e3e04c4cd4a2b4c036c7e05a4b17662956d4ed0d",
    "charge_type": "mbway",
    "confirmed": true,
    "currency": "EUR",
    "created_at": "2016-03-01T01:14:53.320743+00:00",
    "amount": 42,
    "metadata": {
      "orderId": 1337
    }
  }
}
```

Attribute           | Type     | Description
------------------- | -------- | -----------
id                  | string   | Unique identifier of the event.
type                | string   | Code that specifies the type of event.
result_status       | string   | The status of the call to the specified events webhook.
charge              | object   | [Charge](#charges) object (for events related to Charges, Instruments and Payments).
instrument          | object   | [Instrument](#instruments) object (for events related to Instruments and Payments).
payment             | object   | [Payment](#payments) object (for events related to Payments).

## List events

### HTTP Request

> Example Request

```shell
$ curl -vX GET "https://api.switchpayments.com/v2/events"
```

`GET https://api.switchpayments.com/v2/events`
