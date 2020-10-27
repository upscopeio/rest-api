# [Upscope](https://upscope.com/)'s REST API

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
      {
        "id": 1,
        "length_seconds": 91,
        "formatted_length": "1 minute",
        "call_length_seconds": 0,
        "formatted_call_length": "-",
        "started_at": "2018-07-05T11:53:38.000+01:00",
        "ended_at": "2018-07-05T11:55:09.000+01:00",
        "went_live": true,
        "features_used": [
          "click"
        ],
        "visitor_id": "ABCD1234",
        "team": {
          "id": 1,
          "domain": "upscope.io",
          "_type": "team"
        },
        "agents": [
          {
            "id": 1,
            "name": "Joe Smith",
            "agent_identifier": "ext-123",
            "_type": "external_agent"
          }
        ],
        "_type": "session"
      }
    ],
    "_type": "visitor"
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

### Getting screenshots and pageviews
If you'd like to get the pageviews and screenshots captured for the visitor, add `?with_history=true` in the query string. You will then have access to the `visitor.history` attribute containing pageviews and screenshots.

Screenshots and history need to be saved on the javascript SDK with the `Upscope('saveHistory');` method.

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


## Deleting all visitor from Upscope
To delete all offline visitors from our system, make a `DELETE` call to the following endpoint:
```
https://api.upscope.io/v1.1/visitors.json?all=true
```
Only visitors that are not currently online will be removed, and the process might take a few minutes to complete.


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
    "no_bottom_bar": false,
    "on_end_url": "https://your-website.com/admin/screenshare/XYZ/ended"
  },
  "permissions": {
    "allow_click": false
  },
  "agent": {
    "id": "123",
    "name": "Joe Smith"
  },
  "metadata": {
    "key": "value"
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
| `branding.on_end_url` | If set, the agent will be redirected to this url after the session ends. If not set, the window will be attempted to be closed. |
| `branding.no_bottom_bar` | If true the bar with connection information at the bottom of the screen will not be shown. Defaults to null. |
| `branding.metadata` | A JSON-encodeable object of metadata to add to the session object. This can be used when retrieving a list of sessions to run your own analytics. The object should be flat, meaning values can only be strings, numbers, or booleans |
| `permissions.allow_console` | Whether the agent should have access to remote console capabilities (defaults to `false`) |
| `permissions.allow_click` | Whether the agent should have access to remote click (defaults to `true`, unless disabled through the Javascript SDK) |
| `permissions.allow_draw` | Whether the agent should have access to drawing (defaults to `true`, unless disabled through the Javascript SDK) |
| `permissions.allow_scroll` | Whether the agent should have access to remote scroll (defaults to `true`, unless disabled through the Javascript SDK) |
| `permissions.allow_audio` | Whether the agent should have access to audio (defaults to `true`, unless disabled through the Javascript SDK) |
| `permissions.allow_type` | Whether the agent should have access to remote type (defaults to `true`, unless disabled through the Javascript SDK) |
| `permissions.allow_agent_redirect` | Whether the agent should have access to remote agent redirect & reload features (defaults to `true`, unless disabled through the Javascript SDK) |

### Errors
- If the visitor is not found, a `404 - NOT FOUND` error will be returned.
- If the screen share would put you over your subscription limits, a
  `402 - PAYMENT REQUIRED` error will be returned.
