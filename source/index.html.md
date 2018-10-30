---
title: FINOS API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: curl
  - ruby: Ruby
  - go: Go
  - javascript: Node

toc_footers:

includes:
  - errors

---

# Introduction

Welcome to the FINOS API! You can use our API to access FINOS API endpoints.

We have language bindings in Ruby, Go, and JavaScript! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.


# Authentication

> To authorize, use this code:

```ruby
require 'finos'

api = Finos::APIClient.authorize!('meowmeowmeow')
```


```go
package main

import (
    "log"

    "github.com/finos/go-sdk"
)
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: meowmeowmeow"
```

```javascript
const finos = require('finos');

let api = finos.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Finos uses API keys to allow access to the API. You can register a new FINOS API key at our [developer portal](http://example.com/developers).

FINOS expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'finos'

api = FINOS::APIClient.authorize!('meowmeowmeow')
```

```shell
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const FINOS = require('FINOS');

let api = FINOS.authorize('meowmeowmeow');
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'FINOS'

api = FINOS::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import FINOS

api = FINOS.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const FINOS = require('FINOS');

let api = FINOS.authorize('meowmeowmeow');
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
require 'FINOS'

api = FINOS::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import FINOS

api = FINOS.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const FINOS = require('FINOS');

let api = FINOS.authorize('meowmeowmeow');
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

