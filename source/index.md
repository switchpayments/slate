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

All the API endpoints require merchant authentication, even though some may require public credentials (i.e. available to the world).

# Payments

## Create a new payment


```shell
curl -vX GET https://api.switchpayments.com
```

## Authorize a payment

## Capture a payment

## Void a payment

## Refund a payment

## List all payments

# Cards

## Create a new card

## Update a card

# Customers

## Create a new customer

## Update a customer

## List a customer's cards

## Update a customer's card

# Tokens

## Create a new token