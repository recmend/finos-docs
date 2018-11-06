---
title: Finos API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: curl
  - go: Go
  - javascript: Node
  - python: Python
  - ruby: Ruby

toc_footers:

---

# Introduction

```shell
https://api.finos.com
```

```go
import (
  "github.com/finos/go"
)
```

```javascript
npm install finos
```

```python
pip install finos
```

```ruby
# use rubygems
gem install finos

# or use bundler
gem 'finos'
```

Welcome to Finos! This API reference provides information on available endpoints and how to interact with it.

The Finos API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). Our API has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. All requests should be over SSL. All request and response bodies, including errors are encoded in JSON.

We also have some specific language bindings to make integration easier. You can switch the programming language of the examples with the tabs in the top right.

Currently we support the following official client bindings:

* **Go**
* **Node**
* **Python**
* **Ruby**

# Authentication

> Example Request

```ruby
require 'finos'
Finos.api_key = "your_api_key"

Finos::Account.fetch("account_id")
```

```python
import finos
finos.api_key = "your_api_key"

finos.Account.fetch("account_id")
```

```go
import (
    finos "github.com/finos/go"
)

finos.Key = "your_api_key"
finos.Account.fetch("account_id")
```

```shell
$ curl https://api.finos.com/accounts \
  -H "Authorization: your_api_key"
```

```javascript
const finos = require('finos')
("your_api_key");

finos.accounts.fetch('account_id');
```

Authentication is done via the API key which you can find in your account settings. Your API keys carry many privileges, so be sure to keep them secure! Do not share your secret API keys in publicly accessible areas such as GitHub, client-side code, and so forth.

Authentication to the API is performed via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication). Provide your API key as the basic auth username value. You do not need to provide a password.

Alternatively you can also pass your API key as a bearer token in an Authorization header.

`-H "Authorization: Bearer *your_api_key*"`

All API requests must be made over HTTPS. Calls made over plain HTTP will fail. API requests without authentication will also fail.

# Errors

Our API returns standard HTTP success or error status codes. For errors, we will also include extra information about what went wrong encoded in the response as JSON. The various HTTP status codes we might return are listed below.

In general: Codes in the 2xx range indicate success. Codes in the 4xx range indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.). Codes in the 5xx range indicate an error with Finos's servers (these are rare).

## HTTP Status codes

Code | Title | Description
---- | ----- | -----------
200 | OK | Everything worked as expected.
400 | Bad Request | The request was unacceptable, often due to missing a required parameter.
401 | Unauthorized | No valid API key provided.
402 | Request Failed | The parameters were valid but the request failed.
404 | Not Found | The requested resource doesn't exist.
409 | Conflict | The request conflicts with another request (perhaps due to using the same idempotent key).
429 | Too Many Request | Too many requests hit the API too quickly. We recommend an exponential backoff of your requests.
500,502,503,504 | Server Errors | Something went wrong on Finos's end. (These are rare.)

## Error types

> Example error response.

```json
{
  "error": {
    "type": "invalid_request",
    "message": "name is required"
  }
}
```

Type | Description
---- | -----------
api_connection | Failure to connect to Finos's API.
api_error | API errors cover any other type of problem (e.g., a temporary problem with our servers), and are extremely uncommon.
authentication_error | Failure to properly authenticate yourself in the request.
invalid_request | Invalid request errors arise when your request has invalid parameters.
rate_limit_exceeded | Too many requests hit the API too quickly.
validation_error | Unable to validate POST/PUT
not_found | Resource was not found.

# Rate limiting

> Example rate limit error response.

```json
{
  "error": [
    {
      "type": "rate_limit_exceeded",
      "message": "Too many requests"
    }
  ]
}
```

The Finos API is rate limited to prevent abuse that would degrade our ability to maintain consistent API performance for all users. By default, each API key or app is rate limited at 10,000 requests per hour. If your requests are being rate limited, HTTP response code 429 will be returned with an rate_limit_exceeded error.

# Webhooks
A webhook is a messaging mechanism that enables you to receive notification of events as they occur. The Finos platform sends such event notifications to a webhook endpoint, an endpoint hosted in your environment that you have configured to receive and process event notifications.

You configure the webhook object with the URL of your webhook endpoint and with a set credentials for accessing that endpoint. You also configure it to send notifications for specific types of events (you can configure it to send all types or any subset of types). If you would like to set up multiple webhook endpoints and route different types of event notifications to each, you can do so by creating multiple webhook objects and configuring each to send a specific type of event notification to a specific endpoint.

## Securing webhooks
To validate a webhook came from Finos we suggest verifying the webhook payloads with the X-Request-Signature header (which we pass with every webhook). If you’re using one of our client bindings to parse webhooks, say the Ruby library, we’ll do this for you.

Header | Description
------ | -----------
X-Request-Signature | A SHA1 HMAC hexdigest computed with your API key and the raw body of the request. Make sure to strip the prefix (sk_, pk_) from the start of the key before generating the HMAC.

## Create webhook

## Retrieve webhook

## Update webhook

## List webhooks

# Pagination

```json
{
  "pagination": {
    "ending_before": null,
    "starting_after": null,
    "limit": 25,
    "order": "desc",
    "previous_page": null,
    "next_page": "/accounts?&limit=25&starting_after=5d5aed5f-b7c0-5585-a3dd-a7ed9ef0e414"
  },
  "data": [
    ...
  ]
}
```

All `GET` endpoints which return an object list support cursor based pagination with pagination information inside a `pagination` object. This means that to get all objects, you need to paginate through the results by always using the `id` of the last resource in the list as a `starting_after` parameter for the next call. To make it easier, the API will construct the next call into `next_page` together with all the currently used pagination parameters. You know that you have paginated all the results when the response’s `next_page` is empty. While using cursor based pagination might seem weird compared to many APIs it protects from the situation when the resulting object list changes during pagination (new resource gets added or removed).

Default limit is set to 25 but values up to 100 are permitted.

The result list is in descending order by default (newest item first) but it can be reversed by supplying order=asc instead.

**ARGUMENTS**

Parameter | Description
--------- | -----------
`limit`  optional | Number of results per call. Accepted values: 0 - 100. Default 25
`order`  optional | Result order. Accepted values: `desc` (default), `asc`
`starting_after` optional | A cursor for use in pagination. `starting_after` is an resource ID that defines your place in the list.
`ending_before` optional | A cursor for use in pagination. `ending_before` is an resource ID that defines your place in the list.

# KYC Verification
Finos requires account holders to pass an identity verification process before accounts are active. This identity verification process is known as KYC (know your customer).


## Perform KYC
> Example Request

```shell
curl -X POST https://api.finos.com/v1/kyc /
  -H "Content-Type: application/json" \
  -u sk_yourapikey \
  -d $'{
    "phone_number": "18042562188",
    "first_name": "John",
    "last_name": "Doe",
    "email_address": "johndoe@finos.com",
    "date_of_birth": "1985-01-23",
    "ssn_number": "123456789",
    "address_line_1": "123 Test St",
    "address_city": "Richmond",
    "address_state": "VA",
    "address_postal_code": "23220",
    "address_country_code": "US",
  }'
```

> Example Response

```json
{
  "status_code": 201,
  "token": "S-BNTwVDLrlRvHbKVhRa8X",
  "data": {
    "result": "success",
    "score": 0.99,
    "tags": ["Address Match", "Phone Match"],
    "outcome": "Approved",
  }
}
```

> If more information is needed, the response is going to look something like this

```json
{
  "token": "S-eJ38yUNDqcMxKwwdxbya",
  "required": [
    {
      "key": "answers",
      "type": "object",
      "description": "Object containing answers to out of wallet question prompts.",
      "template": {
        "answers": [
          {
            "question_id": 1,
            "answer_id": 0
          },
          {
            "question_id": 2,
            "answer_id": 0
          },
          {
            "question_id": 3,
            "answer_id": 0
          },
          {
            "question_id": 4,
            "answer_id": 0
          },
          {
            "question_id": 5,
            "answer_id": 0
          }
        ]
      }
    }
  ],
  "optional": [],
  "prompts": {
    "answers": {
      "questions": [
        {
          "id": 1,
          "question": "What state was your SSN issued in?",
          "answers": [
            {
              "id": 1,
              "answer": "Arkansas"
            },
            {
              "id": 2,
              "answer": "Alabama"
            },
            {
              "id": 3,
              "answer": "West Virginia"
            },
            {
              "id": 4,
              "answer": "Virginia"
            },
            {
              "id": 5,
              "answer": "None Of The Above"
            }
          ]
        },
        <...>
      ]
    }
  }
}
```

>

```shell
curl -X PATCH https://api.finos.com/v1/kyc/S-eJ38yUNDqcMxKwwdxbya \
    -H "Bearer: sk_yourapikey" \
    -H "Content-Type: application/json" \
    -d $'{"answers":[
        {"question_id": 1, "answer_id": 1},
        {"question_id": 2, "answer_id": 5},
        {"question_id": 3, "answer_id": 2},
        {"question_id": 4, "answer_id": 1},
        {"question_id": 5, "answer_id": 5}
      ],
      "name_first": "Charles",
      "name_last": "Hearn"
    }'
```

Use this endpoint to verify the identity of an account holder. If we are able to resolve the person and there is no need for further information. If more information is needed, the response will indicate what is needed and what the format for that information should be.

### HTTP Request

`POST https://api.finos.com/v1/kyc`

**ARGUMENTS**

Parameter | Type | Required
--------- | ---- | -----------
`phone_number` | string | optional
`first_name`   | string | required
`last_name` | string | required
`email_address` | string | required
`date_of_birth` | string  "YYYY-MM-DD"| required
`ssn_number` | string | required
`address_line_1` | string | required
`address_city` | string | required
`address_state` | string *2 letter code* | required
`address_postal_code` | string | required,
`address_country_code` | string *2 letter code* | required

<aside class="information">
<code>token</code> can be used retrieve KYC results. Must be passed back to PATCH <code>/kyc/{token}</code> if necessary.
</aside>

## Retrieve KYC result

Use this endpoint to retrieve a specific KYC result.

> Example Response

```json
{
  "status_code": 200,
  "token": "S-BNTwVDLrlRvHbKVhRa8X",
  "data": {
    "result": "success",
    "score": 0.99,
    "tags": ["Address Match", "Phone Match"],
    "outcome": "Approved",
  }
}
```

### HTTP Request

`GET https://api.finos.com/v1/kyc/{token}`

## KYC Documents
```shell
curl -X PUT GET https://api.finos.com/v1/kyc/{token}/documents \
  -H "Bearer: sk_yourapikey" \
  -H "Content-Type: application/json" \
  -d $'{
    "name": "john_doe_driver_license",
    "extension": "pdf",
    "type": "license",
    "notes": "Uploaded from email customer sent",
    "data": <file stream>
  }'
```

> A successful request will return a JSON structured like this:

```json
{
  "document_token": "D-fu1WeoS2ywAROk1srtP9",
  "name": "john_doe_driver_license",
  "extension": "pdf",
  "uploaded": true,
  "notes": [
    {
      "notes": "Uploaded from email customer sent",
      "created_at": 1497305164671,
      "updated_at": 1497305164671
    }
  ]
}
```

When an automated KYC cannot be performed for the customer, it has to go through manual review process and additional documentation is required. Once you have submitted the additional documentation, we will send a webhook notifying the status
of the KYC review.

<aside class="information">
Uploaded documents are not reviewed or approved in UAT. In Production, documents are reviewed by Finos's licensed customer service department to verify the user's identity.
</aside>

### Upload a Document

`POST https://api.finos.com/v1/kyc/{token}/documents`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`name` | string |	Name of the document to be uploaded
`type` | string	| Type of document requested by the kyc endpoint (ssn card, ‘license’, ‘passport’, or ‘utility’)
`notes` | string | Document notes
`data` | string | base64 encoded file content

# Users

The users resource represents a person who uses a Finos account. This endpoint enables you to create and manage users on the Finos platform.

This resource stores user attributes such as name, address, and date of birth, as well as financial information such as social security.


## Create user




## Retrieve user

## Update user

## List users

## Search users

# Accounts

# Bank Accounts

# Deposits

# Withdrawals

# Trading

# Portfolio

# Statements
