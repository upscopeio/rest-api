# [Upscope.io](https://upscope.io/)'s REST API

With Upscope's REST API you can retrieve information about your account and your
visitors, as well as generate authorization tokens that allow unauthenticated visitors
to screen share with Upscope. This enables you to integrate Upscope within your
application and handle authentication however fits your needs.

Please note that our REST API is only available to our Enterprise customers.

## Authentication
To use our REST API, you will need a secret API key. If the API is enabled for
your account, you will find this in the
[general settings](https://app.upscope.io/settings/teams/_). You can only copy
the API key once before it is hidden. You will need to update the API key if you
lose access to it.

To authenticate your requests you will need to set a `X-Api-Key` header with
your API key as value.

If authorization is unsuccessful your request will fail.

### Authentication errors

```json
{
  "status": "error",
  "error": "invalid_api_key"
}
```

| HTTP Code | Error | Meaning |
| --------- | ----- | ------- |
| 401 | `invalid_api_key` | The API key provided was not found / no API key was provided |
| 401 | `subscription_expired` | The team's subscription has now expired |
| 401 | `api_not_enabled` | The team's subscription does not include API access |

## Retrieving usage statistics
To get your account's current usage statistics, make a `GET` request to:
```
https://api.upscope.io/v1.1/usage.json
```

A response will look like this:

```json
{
  "usage": {
    "visitors": {
      "online_now": 100,
      "last_24_hrs": 1000,
      "last_30_days": 40000
    },
    "agents": {
      "today": 5,
      "last_30_days": 20
    },
    "session_counts": {
      "now": 3,
      "today": 30,
      "last_30_days": 100
    },
    "session_seconds": {
      "today": 400,
      "last_30_days": 10000
    }
  }
}
```

In this context, "session" means one screen sharing session.

## Searching for a visitor or retrieving a list of visitors
To get a list of visitors or search for a specific visitor, you can make a `GET` call to the following
endpoint:
```
https://api.upscope.io/v1.1/list.json?search=QUERY
```

An optional query parameter `search` will let you search just like you can do on
`https://app.upscope.io/s`.

A successful response will look like this:

```json
{
  "status": "ok",
  "visitors": [
    {
      "short_id": "1234",
      ...
    }
  ]
}
```

## Retrieving a visitor's information
To get information about a visitor, you can make a `GET` call to the following
endpoint:
```
https://api.upscope.io/v1.1/visitors/:visitor_id.json
```
With `:visitor_id` being the unique Upscope ID for the visitor. This is also called
`short_id`.

A successful response will look like this:
```json
{
  "status": "ok",
  "visitor": {
    "api_key": "xyzxyzxyz",
    "browser_name": "Chrome 65",
    "device_name": null,
    "device_type": "desktop",
    "first_seen_at": "2018-03-16T15:25:59.000Z",
    "identities": ["Joe Smith"],
    "integration": "intercom",
    "ip_address": "8.8.8.8",
    "is_in_session": false,
    "is_online": true,
    "js_configuration": false,
    "last_seen_at": "2018-04-17T14:43:44.000Z",
    "last_url": "https://your-website.com/abc",
    "location_city": "New York, NY",
    "location_country": "US",
    "location_country_name": "united states",
    "lookup_code": null,
    "short_id": "ABCD1234",
    "tags": ["#customer"],
    "unique_id": "123",
    "sessions": [
    
    ],
    "_type", "visitor"
  }
}
```

### Attributes
The visitor is described by the following attributes:

| Attribute | Description |
| --------- | ----------- |
| visitor.short_id | The Upscope ID for the specific visitor you looked up. |
| visitor.api_key | Your public API key. |
| visitor.ip_addres | The IP address the visitor last connected from. |
| visitor.location_city | The city the visitor is in (according to their IP address). If the city is not known, this will be null. |
| visitor.location_country | The country (code) the visitor is in (according to their IP address). If the country is not known, this will be null. |
| visitor.last_url | The last url the visitor has visited. |
| visitor.identities | A list of strings used to identify the visitor. If the visitor was not identified, this will be null. |
| visitor.unique_id | The unique ID used to identify the visitor. If this was not set, it will be null. |
| visitor.lookup_code | The code your agent can look for to quickly find the visitor in the interface. If not set, it will be null. |
| visitor.device_type | The category of device the visitor is connecting from. It will be one of desktop , console , mobile , tablet , smarttv , wearable , embedded. |
| visitor.device_name | The name or brand of the device (e.g. iPhone ) |
| visitor.browser_name | The name of the browser used by the visitor (e.g. Mobile Safari ) |
| visitor.integration | The auto integration we have performed on this visitor. This will be set to the live chat system you have installed on the page last visited by the visitor. |
| visitor.last_seen_at | A timestamp of the last time the visitor was seen on the website |
| visitor.first_seen_at | A timestamp of the first time the visitor was seen on the website |
| visitor.is_online | A boolean describing whether the visitor is currently connected to Upscope (and available for screensharing). |
| visitor.is_in_session | A boolean describing whether the visitor is currently screen sharing with someone. |
| visitor.js_configuration | A boolean describing whether the visitor has has configured Upscope through the Javascript API in a way that could conflict with the account's general settings. |
| visitor.sessions | A list of recent screen sharing sessions with this visitor. |

### Errors
- If the visitor is not found, a `404 - NOT FOUND` error will be returned.

## Deleting a visitor from Upscope
To completely delete a visitor from our system, you can make a `DELETE` call to the
following endpoint:
```
https://api.upscope.io/v1.1/visitors/:visitor_id.json
```
With `:visitor_id` being the unique Upscope ID for the visitor. This is also called
`short_id`.

### Errors
- If the visitor is not found, a `404 - NOT FOUND` error will be returned.

## Generating a link to start screen sharing with a visitor
To start screen sharing with a visitor, you can make a `POST` request to the
following endpoint:
```
https://api.upscope.io/v1.1/visitors/:visitor_id/watch_url.json
```
With `:visitor_id` being the unique Upscope ID for the visitor. This is also called
`short_id`.

The content of the request should be a JSON-encoded document with this format:

```json
{
  "branding": {
    "naked": true,
    "retry_url": "https://your-website.com/admin/screenshare/XYZ",
    "no_bottom_bar": false
  },
  "agent": {
    "id": "123",
    "name": "Joe Smith"
  }
}
```

A successful response will look like this:
```json
{
  "watch_url": "https://o.upscope.io?xyz"
}
```

By visiting the link contained in `watch_url`, screen sharing will begin without
further authorization required.

### Options
| Parameter | Description |
| --------- | ----------- |
| `agent.id` | The ID of the agent (required). |
| `agent.name` | The name of the agent (required). |
| `branding.naked` | If true , the Upscope logo will not be shown on the page. This is ideal if you want to display the page inside an iframe. Defaults to false. |
| `branding.retry_url` | If set, the visitor will be able to click a "Retry now" link that redirects to this url if there are problems. When null , no button is displayed. Defaults to null. |
| `branding.no_bottom_bar` | If true the bar with connection information at the bottom of the screen will not be shown. Defaults to null. |

### Errors
- If the visitor is not found, a `404 - NOT FOUND` error will be returned.
- If the screen share would put you over your subscription limits, a
  `402 - PAYMENT REQUIRED` error will be returned.
