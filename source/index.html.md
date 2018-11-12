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

# Users

The users resource represents a person who uses a Finos account. This endpoint enables you to create and manage users on the Finos platform.

This resource stores user attributes such as name, address, and date of birth, as well as financial information such as social security.

## Create user

> Example Request

```shell
curl -X POST https://api.finos.com/v1/users \
  -H "Bearer: sk_yourapikey" \
  -H "Content-Type: application/json" \
  -d $'{
    "phone_number": "18042562188",
    "first_name": "John",
    "last_name": "Doe",
    "email_address": "johndoe@finos.com",
    "birth_date": "1985-01-23",
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
    "id": "f6e54303-10ce-42b5-a4b0-b28c19cf5025",
    "status": "action_required",
    "kyc_token": "S-BNTwVDLrlRvHbKVhRa8X"
}
```

### HTTP Request

`POST https://api.finos.com/v1/users`

This endpoint enables you to create a user. It will automatically perform the Know Your Customer (KYC) checks if the complete address and ssn_number is passed in the user create request.

**ARGUMENTS**

Parameter | Type | Required
--------- | ---- | -----------
`phone_number` | string | optional
`first_name`   | string | required
`last_name` | string | required
`email_address` | string | required
`birth_date` | string  "YYYY-MM-DD"| required
`ssn_number` | string | required
`address_line_1` | string | required
`address_city` | string | required
`address_state` | string *2 letter code* | required
`address_postal_code` | string | required,
`address_country_code` | string *2 letter code* | required

**RESPONSE EXPLAINED**

Parameter | Description
--------- | -----------
`user_id` | Unique user id on the finos platform
`kyc_token` | To perform follow up KYC checks if neccessary
`status` | Status of the user account as per KYC initial verification.

Unless the user status becomes `approved`, the user cannot perform any subsequent steps like funding their account, make trades etc.

<aside class="information">
For <code>action_required</code> status use KYC Patch request for automated verification. For <code>documentation_required</code> status use KYC document endpoint.
</aside>

## Retrieve user

> Example Request

```shell
curl -X GET https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025 /
  -H "Bearer: sk_yourapikey" \
```

> Example Response

```json
{
    "id": "f6e54303-10ce-42b5-a4b0-b28c19cf5025",
    "status": "active",
    "kyc_token": "S-BNTwVDLrlRvHbKVhRa8X",
    "first_name": "John",
    "last_name": "Doe",
    "phone_number": "9502562188",
    "email_address": "johndoe@example.com",
    "address_line_1": "123 Test St",
    "address_line_2": "unit 4",
    "address_city": "Redmond",
    "address_state": "VO",
    "address_postal_code": "34512",
    "address_country_code": "US"
}
```

### HTTP Request

`GET https://api.finos.com/v1/users/{id}`

This endpoint enables you to retrieve a specific user.

## Update user
> Example Request

```shell
curl -X PUT https://api.finos.com/v1/users/{id} /
  -H "Bearer: sk_yourapikey" \
  -H "Content-Type: application/json" \
  -d $'{
    "phone_number": "6742562188"
  }'
```

> Example Response

```json
{
    "id": "f6e54303-10ce-42b5-a4b0-b28c19cf5025",
    "status": "active",
    "kyc_token": "S-BNTwVDLrlRvHbKVhRa8X",
    "first_name": "John",
    "last_name": "Doe",
    "phone_number": "6742562188",
    "email_address": "johndoe@example.com",
    "address_line_1": "123 Test St",
    "address_line_2": "unit 4",
    "address_city": "Redmond",
    "address_state": "VO",
    "address_postal_code": "34512",
    "address_country_code": "US"
}
```

### HTTP Request

`PUT https://api.finos.com/v1/users/{used_id}`

**ARGUMENTS**

Parameter | Type | Required
--------- | ---- | -----------
`phone_number` | string | optional
`first_name`   | string | optional
`last_name` | string | optional
`email_address` | string | optional
`birth_date` | string  "YYYY-MM-DD"| optional
`ssn_number` | string | optional
`address_line_1` | string | optional
`address_city` | string | optional
`address_state` | string *2 letter code* | optional
`address_postal_code` | string | optional
`address_country_code` | string *2 letter code* | optional

## List users

> Example Request

```shell
curl -X GET https://api.finos.com/v1/users?limit=3 \
  -H "Bearer: sk_yourapikey"
```

> Example Response

```json
{
  "pagination": {
    "ending_before": null,
    "starting_after": null,
    "limit": 3,
    "order": "desc",
    "previous_uri": null,
    "next_uri": "https://api.finos.com/v1/users?limit=3&starting_after=58542935-67b5-56e1-a3f9-42686e07fa40"
  },
  "data": [
    { ... },
    { ... },
    {
      "id": "58542935-67b5-56e1-a3f9-42686e07fa40",
      ...
    },
  ]
}
```

### HTTP Request

`GET https://api.finos.com/v1/users`

# KYC Verification
Finos requires account holders to pass an identity verification process before accounts are active. This identity verification process is known as KYC (know your customer).


## Perform KYC
> Example Request

```shell
curl -X POST https://api.finos.com/v1/kyc \
  -H "Bearer: sk_yourapikey" \
  -H "Content-Type: application/json" \
  -d $'{
    "phone_number": "18042562188",
    "first_name": "John",
    "last_name": "Doe",
    "email_address": "johndoe@finos.com",
    "birth_date": "1985-01-23",
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
`birth_date` | string  "YYYY-MM-DD"| required
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

# Accounts

The accounts resource represents the brokerage accounts with which users conduct financial transactions.


## Create account

> Example Request

```shell
curl -X POST https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/accounts \
  -H "Bearer: sk_yourapikey" \
  -H "Content-Type: application/json" \
  -d $'{
    "ownership": "individual",
    "account_type": "cash",
    "martial_status": "single",
    "country_of_citizenship": "USA",
    "employment_status": "Employed / Self-Employed",
    "employment_position": "Software Engineer",
    "employer_name": "Finos Inc",
    "annual_income": "$100,000 - $199,999",
    "investment_objective": "growth",
    "investment_experience": "limited",
    "net_liquid_worth": "$25,000 - $99,999",
    "net_total_worth": "$100,000 - $199,999",
    "is_director": false,
    "risk_tolerance": "moderate"
  }'
```

> For Joint accounts, change ownership to "joint"

```shell
curl -X POST https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/accounts \
  -H "Bearer: sk_yourapikey" \
  -H "Content-Type: application/json" \
  -d $'{
    "ownership": "joint",
    "account_type": "cash",
    ...,
    "joint_member": {
      "phone_number": "18042562188",
      "first_name": "Marry",
      "last_name": "Doe",
      "email_address": "marryjoe@finos.com",
      "birth_date": "1985-04-21",
      "ssn_number": "666456789",
      "address_line_1": "123 Test St",
      "address_city": "Richmond",
      "address_state": "VA",
      "address_postal_code": "23220",
      "address_country_code": "US",
    }
  }'
```

> Example Response

```json
{
    "user_id": "f6e54303-10ce-42b5-a4b0-b28c19cf5025",
    "id": "2d931510-d99f-494a-8c67-87feb05e1594",
    "account_number": "FOBN50250",
    "ownership": "joint",
    "account_type": "cash",
}
```

### HTTP Request

`POST https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/accounts`

**ARGUMENTS**

Parameter | Description
--------- | -----------
`ownership` *required* |  Individual or Joint Account
`account_type` *required* | Type of account. Options `cash` or `investment`
`martial_status` *required* | Martial Status. Options `single`, `divorced`, `married`, `widowed`
`country_of_citizenship` *required* | Country Citizenship, uses 3 character country code.
`employment_status` *required* | Employment Status. Options `employed / self-employed`, `retired`, `student`, `not employed`
`employment_position` *required* | Occupation e.g. "Software Engineer",
`employer_name` *required* | Employer's name
`annual_income` *optional for `cash` accounts* | Annual income. Options `$0 - $24,999`, `$25,000 - $99,999`, `$100,000 - $199,999`, `$200,000+`
`investment_objective`*optional for `cash` accounts* | Investment objectives. Options `capital preservation`, `growth`, `income`, `speculation`
`investment_experience` *optional for `cash` accounts* | Investment experience. Options `none`, `limited`, `good`, `extensive`
`net_liquid_worth` *optional for `cash` accounts* | Liquid net worth. Options `$0 - $4,999`, `$5,000 - $99,999`, `$100,000 - $199,999`, `$200,000+`
`net_total_worth` options for `cash` accounts | Total net worth. Options `$0 - $24,999`, `$25,000 - $99,999`, `$100,000 - $199,999`, `$200,000+`
`is_director` *optional for `cash` accounts* | Is the account holder(s) a control person of a publicly traded company? A Director, Officer or 10% stock owner?
`director_of` *optional for `cash` accounts* | If director, please list the company name and its ticker symbol.
`risk_tolerance` *optional for `cash` accounts* | Risk tolerance. Options `conservative`, `moderate`, `aggressive`

<aside class="information">
For <code>joint</code> accounts optionally include the <code>joint_member</code> object in the request body as shown in the sample code.
</aside>

## Retrieve account

> Example Request

```shell
curl -X GET https://api.finos.com/v1/accounts/2d931510-d99f-494a-8c67-87feb05e1594 /
  -H "Bearer: sk_yourapikey" \
```

> Example Response

```json
{
  "id": "2d931510-d99f-494a-8c67-87feb05e1594",
  "account_number": "FOBN50250",
  "ownership": "joint",
  "account_type": "cash",
  "martial_status": "married",
  "country_of_citizenship": "USA",
  "employment_status": "Employed / Self-Employed",
  "employment_position": "Software Engineer",
  "employer_name": "Finos Inc"
  "owner": {
    ...
  },
  "joint_member": {

  }
}
```

### HTTP Request

`GET https://api.finos.com/v1/accounts/{id}`

This endpoint enables you to retrieve a specific account.

## Update account
> Example Request

```shell
curl -X PUT https://api.finos.com/v1/accounts/{id} /
  -H "Bearer: sk_yourapikey" \
  -H "Content-Type: application/json" \
  -d $'{
    "account_type": "investment",
    "annual_income": "$100,000 - $199,999",
    "investment_objective": "growth",
    "investment_experience": "limited",
    "net_liquid_worth": "$25,000 - $99,999",
    "net_total_worth": "$100,000 - $199,999",
    "is_director": false,
    "risk_tolerance": "moderate"
  }'
```

> Example Response

```json
{
    "id": "2d931510-d99f-494a-8c67-87feb05e1594",
    "user_id": "f6e54303-10ce-42b5-a4b0-b28c19cf5025",
    "account_number": "FOBN50250",
    "ownership": "single",
    "account_type": "investment",
}
```

### HTTP Request

`PUT https://api.finos.com/v1/accounts/{id}`

This endpoint enables you to update an account. For any updates to the user profile information, use the users resource update method. You can change an active account from single to joint by passing required `joint_member` object in the request. You can also convert a `cash` account to an `investment` by passing the required fields for converting it to an investment account or vice versa.

## List accounts

> Example Request

```shell
curl -X GET https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/accounts?limit=3 \
  -H "Bearer: sk_yourapikey"
```

> Example Response

```json
{
  "pagination": {
    "ending_before": null,
    "starting_after": null,
    "limit": 3,
    "order": "desc",
    "previous_uri": null,
    "next_uri": "https://api.finos.com/v1/acccounts?limit=3&starting_after=58542935-67b5-56e1-a3f9-42686e07fa40"
  },
  "data": [
    { ... },
    { ... },
    {
      "id": "58542935-67b5-56e1-a3f9-42686e07fa40",
      ...
    },
  ]
}
```
You can see a list of the brokerage accounts belonging to a user.

### HTTP Request

`GET https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/accounts`

# Bank Accounts

The bank account resource is used to connect user's external bank accounts with Finos brokerage accounts. The funds are transferred to and from user's bank accounts and Finos brokerage accounts.

## Create bank account

> Example Request

```shell
curl -X POST https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/bank_accounts \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "bank_account_number": "4327843043",
   "bank_routing_number": "121042882",
   "bank_account_nickname": "Joe checking"
 }'
```

> Example Response

```json
{
  "id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
  "user_id": "f6e54303-10ce-42b5-a4b0-b28c19cf5025",
  "bank_account_number": "4327843043",
  "bank_routing_number": "121042882",
  "default": false,
  "bank_account_nickname": "Joe checking"
}
```

### HTTP Request

`POST https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/bank_accounts`

**ARGUMENTS**

Parameter | Description
--------- | -----------
`bank_account_number` *required* | The account number for the bank account. This should be a checking account.
`bank_routing_number` *required* | The routing transit number for the bank account.
`bank_account_nickname` *required* | Any user friendly name for the bank account

## Retrieve bank account

> Example Request

```shell
curl -X GET https://api.finos.com/v1/bank_accounts/bad85eb9-0713-4da7-8d36-07a8e4b00eab /
  -H "Bearer: sk_yourapikey" \
```

> Example Response

```json
{
  "id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
  "user_id": "f6e54303-10ce-42b5-a4b0-b28c19cf5025",
  "bank_account_number": "4327843043",
  "bank_routing_number": "121042882",
  "default": false,
  "bank_account_nickname": "Joe checking"
}
```

### HTTP Request

`GET https://api.finos.com/v1/bank_accounts/{id}`

This endpoint enables you to retrieve a specific user bank account.

## Update bank account

> Example Request

```shell
curl -X PUT https://api.finos.com/v1/bank_accounts/{id} /
  -H "Bearer: sk_yourapikey" \
  -H "Content-Type: application/json" \
  -d $'{
    "default": true,
    "bank_account_nickname": "Joe WF Checking"
  }'
```

> Example Response

```json
{
  "id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
  "user_id": "f6e54303-10ce-42b5-a4b0-b28c19cf5025",
  "default": true,
  "bank_account_nickname": "Joe checking"
}
```

### HTTP Request

`PUT https://api.finos.com/v1/bank_accounts/{id}`

Parameter | Description
--------- | -----------
`default` *optional* | Default bank account for credit and debit of funds.
`bank_account_nickname` *optional* | Any user friendly name for the bank account

## Delete bank account

> Example Request

```shell
curl -X DELETE https://api.finos.com/v1/bank_accounts/{id} /
  -H "Bearer: sk_yourapikey"
```

> Example Response

```json
{
  "id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
  "object": "bank_account",
  "deleted": true
}
```

### HTTP Request

`DELETE https://api.finos.com/v1/bank_accounts/{id}`

## List bank accounts

> Example Request

```shell
curl -X GET https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/bank_accounts?limit=3 \
  -H "Bearer: sk_yourapikey"
```

> Example Response

```json
{
  "pagination": {
    "ending_before": null,
    "starting_after": null,
    "limit": 3,
    "order": "desc",
    "previous_uri": null,
    "next_uri": "https://api.finos.com/v1/users/f6e54303-10ce-42b5-a4b0-b28c19cf5025/bank_accounts?limit=3&starting_after=bad85eb9-0713-4da7-8d36-07a8e4b00eab"
  },
  "data": [
    { ... },
    { ... },
    {
      "id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
      ...
    },
  ]
}
```

### HTTP Request

`GET https://api.finos.com/v1/users/6e54303-10ce-42b5-a4b0-b28c19cf5025/bank_accounts/`

You can see a list of the bank accounts belonging to a user.

# Deposits

## Create deposit

Create one-time ACH deposit with a linked bank account. Funds can be made instantly available depending upon your business account balance and your account configs. You can choose to defer the investments of the newly funds unless the funds have settled in T+1 days.

> Example Request

```shell
curl -X POST https://api.finos.com/v1/deposits \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "account_id": "2d931510-d99f-494a-8c67-87feb05e1594",
   "bank_account_id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
   "amount": "202.00"
 }'
```

> Example Response

```json
{
  "id": "62936e70-1815-439b-bf89-8492855a7e6b",
  "account_id": "2d931510-d99f-494a-8c67-87feb05e1594",
  "status": "pending"
}
```

### HTTP Request

`POST https://api.finos.com/v1/deposits/`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
account_id | string | Brokerage account to which to fund the money
bank_account_id | string | Bank account from which to debit the money
amount | double | The deposit amount

# Withdrawals

## Create deposit

Create one-time ACH withdrawal from brokerage account to a linked bank account.

> Example Request

```shell
curl -X POST https://api.finos.com/v1/withdrawal \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "account_id": "2d931510-d99f-494a-8c67-87feb05e1594",
   "bank_account_id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
   "amount": "202.00"
 }'
```

> Example Response

```json
{
  "id": "62936e70-1815-439b-bf89-8492855a7e6b",
  "account_id": "2d931510-d99f-494a-8c67-87feb05e1594",
  "status": "pending"
}
```

### HTTP Request

`POST https://api.finos.com/v1/withdrawal/`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
account_id | string | Brokerage account to which to withdraw the money from
bank_account_id | string | Bank account from which to credit the money
amount | double | The withdrawal amount.

<aside class="information">
For an invested account, this will trigger sales of some equity to fulfill the requested withdrawal amount. In a highly volatile market, if the requested amount is more than 90% of the total account balance, the withdrawal amount could be lower than the requested amount. It can take **3-5 business days** for the funds to be available in the bank account.
</aside>

# Orders

The orders API allows a user to monitor, place and cancel orders. Each order has a unique identifier automatically generated by the system, and will be returned as part of the order object along with the rest of the fields described below. Once an order is placed, it can be queried using the order ID to check the status.

## Order Types

When you submit an order, you can choose one of supported order types. Currently, we support four different types of orders.

### Market Order

A market order is a request to buy or sell a security at the current-available price in the current market. It provides the most likely method of filling.

If there is enough liquidity, market orders fill nearly instantaneously. However, with less liquid securities, it may still take some time.

As a trade-off, your filling price may slip depending on the market activities. There is also the risk with market orders that they may get filled at unexpected prices due to the short period price spikes.

### Limit Order

A limit order is an order to buy or sell at a specified price or better. A buy limit order (a limit order to buy) is executed at the specified limit price or lower (i.e., better). Conversely, a sell limit order (a limit order to sell) is executed at the specified limit price or higher (better). Unlike a market order, you have to specify the limit price parameter when submitting your order.

While a limit order can prevent negative slippage, it may not get a fill for a quite a bit of time –- it will only be filled if price reaches the specified limit price level. You could miss a trading opportunity if price moves away from the limit price before your order can be filled. Note that even if the price moves to the limit price level, the order still may not get filled there are not enough buyers or sellers at that particular price level.

### Stop Order

A stop order is an order to buy or sell a security when its price moves past a particular point, ensuring a higher probability of achieving a predetermined entry or exit price. Once the price crosses the predefined entry/exit point, the stop order becomes a market order.

A stop order does not guarantee the order will be filled at certain price after converted to a market order.

In order to submit a stop order, you will need to specify the stop price parameter in the API.

### Stop Limit Order

A stop-limit order is a conditional trade over a set time frame that combines the features of stop with those of a limit order and is used to mitigate risk. The stop-limit order will be executed at a specified price, or better, after a given stop price has been reached. Once the stop price is reached, the stop-limit order becomes a limit order to buy or sell at the limit price or better.

In order to submit a stop limit order, you will need to specify both the limit and stop price parameters in the API.

## Order Lifecycle

Order Status | Description
------------ | -----------
`new` | The order has been received, and routed to exchanges for execution. This is the usual initial state of an order.
`partially_filled` | The order has been partially filled.
`filled` | The order has been filled, and no further updates will occur for the order.
`canceled` | The order has been canceled by the user, and no further updates will occur for the order.
`accepted` | The order has been received, but hasn’t yet been routed to exchanges. This state only occurs on rare occasions.
`rejected` | The order has been rejected, and no further updates will occur for the order. This state occurs on rare occasions, and may occur based on various conditions decided by the exchanges.

An order may be canceled through the API up until the point it reaches a state of  `filled`

<aside class="information">
Order API currently accepts orders only while the market is open. Out of the regular market hours, order submission will be rejected.
</aside>

## Create order

> Example Request

```shell
curl -X POST https://api.finos.com/v1/orders/ \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "symbol": "GOOG",
   "quantity": "0.2",
   "side": "buy",
   "type": "market"
 }'
```

> Example Response

```json
{
  "id": "904837e3-3b76-47ec-b432-046db621571b",
  "submitted_at": "2018-10-05T05:48:59Z",
  "asset_id": "904837e3-3b76-47ec-b432-046db621571b",
  "symbol": "GOOG",
  "exchange": "NASDAQ",
  "asset_class": "us_equity",
  "quantity": "0.2",
  "filled_quantity": "0",
  "type": "market",
  "side": "buy",
  "status": "accepted"
}
```

### HTTP Request

`POST https://api.finos.com/v1/orders`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`symbol` *required* | string | symbol or asset ID to identify the asset to trade
`quantity` *required* |  double| number of shares up to four decimal places. Fractional shares are supported.
`side` *required* | string | `buy` or `sell`
`type` *required* | string | `market`, `limit`, `stop`, or `stop_limit`
`limit_price` | double | required if type is `limit` or `stop_limit`
`stop_price` | double | required if type is `limit` or `stop_limit`

## Retrieve an order

> Example Request

```shell
curl -X GET https://api.finos.com/v1/orders/904837e3-3b76-47ec-b432-046db621571b /
  -H "Bearer: sk_yourapikey" \
```

> Example Response

```json
{
  "id": "904837e3-3b76-47ec-b432-046db621571b",
  "submitted_at": "2018-11-05T05:48:59Z",
  "filled_at": "2018-11-05T05:48:59Z",
  "expired_at": "2018-11-05T05:48:59Z",
  "canceled_at": "2018-11-05T05:48:59Z",
  "failed_at": "2018-11-05T05:48:59Z",
  "asset_id": "904837e3-3b76-47ec-b432-046db621571b",
  "symbol": "GOOG",
  "exchange": "NASDAQ",
  "asset_class": "us_equity",
  "quantity": "0.2",
  "filled_quantity": "0",
  "type": "market",
  "side": "buy",
  "limit_price": "107.00",
  "stop_price": "106.00",
  "filled_average_price": "106.00",
  "status": "accepted"
}
```

### HTTP Request

`GET https://api.finos.com/v1/orders/{id}`

This endpoint enables you to retrieve a specific order.

## Cancel order

> Example Request

```shell
curl -X DELETE https://api.finos.com/v1/orders/{id} /
  -H "Bearer: sk_yourapikey"
```

> Example Response

```json
{
  "id": "904837e3-3b76-47ec-b432-046db621571b",
  "object": "order",
  "canceled": true
}
```

### HTTP Request

`DELETE https://api.finos.com/v1/orders/{id}`

## List orders

> Example Request

```shell
curl -X GET https://api.finos.com/v1/orders?limit=10 \
  -H "Bearer: sk_yourapikey"
```

> Example Response

```json
{
  "pagination": {
    "ending_before": null,
    "starting_after": null,
    "limit": 10,
    "order": "desc",
    "status": "open",
    "previous_uri": null,
    "next_uri": "https://api.finos.com/v1/orders?limit=3&starting_after=bad85eb9-0713-4da7-8d36-07a8e4b00eab"
  },
  "data": [
    { ... },
    { ... },
    {
      "id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
      ...
    },
  ]
}
```

### HTTP Request

`GET https://api.finos.com/v1/orders`

Retrieves a list of orders for your account.

<aside class="information">
<code>status</code> is order status query parameter. `open`, `closed` or `all`. Defaults to `open`.
</aside>

# Managed Accounts

Finos offers complete suite of fully automated investment management APIs that maximizes the long-term, net-of-fee, after-tax, real investment return for each user’s particular tolerance for risk.  You can upload your own custom portfolios or use the one offered by us. Our portfolios are identified using Modern Portfolio Theory (MPT) and combine a broad set of asset classes, each usually represented by a low-cost, passive ETF.

## Account Settings

> Example Request

```shell
curl -X PUT https://api.finos.com/v1/accounts/2d931510-d99f-494a-8c67-87feb05e1594/settings \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "managed": true,
   "min_cash_percent": "1",
   "max_cash_percent": "5",
   "portfolio_id": "62936e70-1815-439b-bf89-8492855a7e6b"
 }'
```

> Example Response

```json
{
  "id": "2d931510-d99f-494a-8c67-87feb05e1594",
  "portfolio_id": "62936e70-1815-439b-bf89-8492855a7e6b",
  "managed": true
}
```

Configure account settings for investment management.

### HTTP Request

`PUT https://api.finos.com/v1/accounts/2d931510-d99f-494a-8c67-87feb05e1594/settings`

**AGRUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`portfolio_id` | string | Portfolio Id created by the RIA developer based on user's risk profile. If not passed, will be automatically set by our default portfolio id based on user's risk profile.
`min_cash_percent` | Double | Minimum cashing holdings in the account. Defaults to `1.0` or minimum of monthly management fees of the total AUM.
`max_cash_percent` | Double | Maximum cashing holdings in the account. Defaults to `5.0` or minimum of monthly management fees of the total AUM.

## Rebalancing

We continuously monitor and periodically rebalance portfolios to ensure they remain optimally diversified. We also attempt to minimize your taxes by analyzing the taxes likely to be generated by each asset class, and creating allocations that are specifically customized for taxable and non-taxable (retirement) portfolios.

## Tax-Loss Harvesting

For taxable accounts, we monitor your portfolio daily to look for opportunities to harvest losses on the ETFs that represent each asset class in your portfolio. Under the right circumstances we will sell one of your ETFs that is trading at a loss and replace it with an alternative ETF that tracks a different (as supplied in the asset class as secondary), but highly correlated index to maintain the risk and return characteristics of your portfolio. We will then hold that alternative ETF in your portfolio for a minimum of 30 days to avoid wash sales. We will not sell the alternative ETF until it can be sold for a loss (which will generate additional tax-loss harvesting benefit).

# Portfolio Management

Finos allows you to upload your own investment models or use the ones provided by us.
Our default investment methodology employs five steps:

* Identify a diverse set of asset classes
* Select the most appropriate ETFs to represent each asset class
* Apply Modern Portfolio Theory to construct asset allocations that maximize the expected net-of-fee, after tax real return for each level of portfolio risk  
* Determine your risk tolerance to select the allocation that is most appropriate for you
* Monitor and periodically rebalance your portfolio taking advantage of dividend reinvestment
s

## Create asset class

> Example Request

```shell
curl -X POST https://api.finos.com/v1/asset_classes/ \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "name": "U.S. Stocks",
   "primary": "VTI",
   "secondary": ["SCHB", "ITOT"]
   ]
 }'
```

> Example Response

```json
{
  "id": "us-stocks",
  "name": "U.S. Stocks",
  "primary": "VTI",
  "secondary": ["SCHB", "ITOT"]
}
```

Create an asset class type

### HTTP Request

`PUT https://api.finos.com/v1/asset_classes/`

<aside class="information">
Secondary ETFs are primarily used for tax-loss harvesting benefits for taxable managed investment accounts.
</aside>

The following table list our default asset class compositions.

Asset Class | ID | Primary | Secondary
----------- | -- | ------- | ---------
U.S. Stocks | us-stocks | VTI | SCHB, ITOT
Foreign Stocks | foreign-stocks | SCHF | VEA, IXUS
Emerging Markets | emerging-market | VWO | IEMG, SCHE
Dividend Stocks | dividend-stocks | VIG | SCHD, DVY
Natural Resources | natural-resources | VDE | XLE
Municipal Bonds | municipal-bonds | VTEB | TFI, MUB
Emerging Market Bonds | emerging-market-bonds | EMB | PCY, EMLC
Corporate Bonds | corporate-bonds | LQD | VCIT, SPIB

## Update asset class

> Example Request

```shell
curl -X PUT https://api.finos.com/v1/asset_classes/us-equity \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "primary": "ITOT",
   "secondary": ["SCHB", "VTI"]
   ]
 }'
```

> Example Response

```json
{
  "id": "us-stocks",
  "name": "U.S. Stocks",
  "primary": "ITOT",
  "secondary": ["SCHB", "VTI"]
}
```

Update an asset class compositions.

### HTTP Request

`PUT https://api.finos.com/v1/asset_classes/us-equity`

## Create portfolio

> Example Request

```shell
curl -X POST https://api.finos.com/v1/portfolios/ \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "name": "moderate",
   "allocations": [
    {
      "asset_class_id": "us-stocks",
      "weight": "30.0"
    },
    {
      "asset_class_id": "emerging-market-stocks",
      "weight": "30.0"
    },
    ...
   ]
 }'
```

> Example Response

```json
{
  "id": "62936e70-1815-439b-bf89-8492855a7e6b",
  "name": "moderate",
  "allocations": [
   {
     "asset_class_id": "us-stocks",
     "weight": "30.0"
   },
   {
     "asset_class_id": "emerging-market-stocks",
     "weight": "30.0"
   },
   ...
  ]
}
```

Create a portfolio.

### HTTP Request

`POST https://api.finos.com/v1/portfolios/`

<aside class="information">
weights must sum to 100
</aside>

## Update portfolio

> Example Request

```shell
curl -X PUT https://api.finos.com/v1/portfolios/ \
 -H "Bearer: sk_yourapikey" \
 -H "Content-Type: application/json" \
 -d $'{
   "name": "conservative",
   "allocations": [
    {
      "asset_class_id": "us-stocks",
      "weight": "80.0"
    },
    {
      "asset_class_id": "emerging-market-bonds",
      "weight": "20.0"
    }
   ]
 }'
```

> Example Response

```json
{
  "id": "62936e70-1815-439b-bf89-8492855a7e6b",
  "name": "conservative",
  "allocations": [
   {
     "asset_class_id": "us-stocks",
     "weight": "80.0"
   },
   {
     "asset_class_id": "emerging-market-bonds",
     "weight": "20.0"
   }
  ]
}
```

Update a portfolio composition.

### HTTP Request

`PUT https://api.finos.com/v1/portfolios/`

# Statements

## Retrieve a statement

> Example Request

```shell
curl -X GET https://api.finos.com/v1/statements/904837e3-3b76-47ec-b432-046db621571b /
  -H "Bearer: sk_yourapikey"
```

> Example Response

```json
{
  "id": "904837e3-3b76-47ec-b432-046db621571b",
  "url": "signed-url"
}
```

### HTTP Request

`GET https://api.finos.com/v1/statements/{id}`

## List statements

> Example Request

```shell
curl -X GET https://api.finos.com/v1/statements?account_id=904837e3-3b76-47ec-b432-046db621571b \
  -H "Bearer: sk_yourapikey"
```

> Example Response

```json
{
  "pagination": {
    "ending_before": null,
    "starting_after": null,
    "limit": 10,
    "order": "desc",
    "start_date": "2018-06-01T00:00:00Z",
    "end_date": "2018-06-01T00:00:00Z",
    "previous_uri": null,
    "next_uri": "https://api.finos.com/v1/statements?account_id=&904837e3-3b76-47ec-b432-046db621571b"
  },
  "data": [
    { ... },
    { ... },
    {
      "id": "bad85eb9-0713-4da7-8d36-07a8e4b00eab",
      "url": "signed-url"
    },
  ]
}
```
Retrieves a list of statements for an account.

### HTTP Request

`GET https://api.finos.com/v1/statements?account_id={id}&start_date={datetime}`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`account_id` | string | Brokerage Account ID
`start_date` | string | Start date in ISO format. Defaults to 90 days ago.
`end_date` | string | End date in ISO format. Defaults to the end of the month.

<aside class="information">
<code>url</code> access will expire in 30 minutes.
</aside>
