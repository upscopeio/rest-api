# [Upscope.io](https://upscope.io/)'s REST API

With Upscope's REST API you can retrieve information about your account and your
users, as well as generate authorization tokens that allow unauthenticated users
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
https://api.upscope.io/v1.0/usage.json
```

A response will look like this:

```json
{
  "usage": {
    "users": {
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

## Searching for a user or retrieving a list of users
To get information about a user, you can make a `GET` call to the following
endpoint:
```
https://api.upscope.io/v1.0/list.json
```

An optional query parameter `search` will let you search just like you can do on
`https://app.upscope.io/s`.

A successful response will look like this:

```json
{
  "status": "ok",
  "users": [
    {
      "short_id": "1234",
      ...
    }
  ]
}
``` 

## Retrieving a user's information
To get information about a user, you can make a `GET` call to the following
endpoint:
```
https://api.upscope.io/v1.0/users/:user_id.json
```
With `:user_id` being the unique Upscope ID for the user. This is also called
`short_id`.

A successful response will look like this:
```json
{
  "status": "ok",
  "user": {
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
  }
}
```

### Attributes
The user is described by the following attributes:

| Attribute | Description |
| --------- | ----------- |
| user.short_id | The Upscope ID for the specific user you looked up. |
| user.api_key | Your public API key. |
| user.ip_addres | The IP address the user last connected from. |
| user.location_city | The city the user is in (according to their IP address). If the city is not known, this will be null. |
| user.location_country | The country (code) the user is in (according to their IP address). If the country is not known, this will be null. |
| user.last_url | The last url the user has visited. |
| user.identities | A list of strings used to identify the user. If the user was not identified, this will be null. |
| user.unique_id | The unique ID used to identify the user. If this was not set, it will be null. |
| user.lookup_code | The code your agent can look for to quickly find the user in the interface. If not set, it will be null. |
| user.device_type | The category of device the user is connecting from. It will be one of desktop , console , mobile , tablet , smarttv , wearable , embedded. |
| user.device_name | The name or brand of the device (e.g. iPhone ) |
| user.browser_name | The name of the browser used by the user (e.g. Mobile Safari ) |
| user.integration | The auto integration we have performed on this user. This will be set to the live chat system you have installed on the page last visited by the user. |
| user.last_seen_at | A timestamp of the last time the user was seen on the website |
| user.first_seen_at | A timestamp of the first time the user was seen on the website |
| user.is_online | A boolean describing whether the user is currently connected to Upscope (and available for screensharing). |
| user.is_in_session | A boolean describing whether the user is currently screen sharing with someone. |
| user.js_configuration | A boolean describing whether the user has has configured Upscope through the Javascript API in a way that could conflict with the account's general settings. |

### Errors
- If the user is not found, a `404 - NOT FOUND` error will be returned.

## Deleting a user from Upscope
To completely delete a user from our system, you can make a `DELETE` call to the
following endpoint:
```
https://api.upscope.io/v1.0/users/:user_id.json
```
With `:user_id` being the unique Upscope ID for the user. This is also called
`short_id`.

### Errors
- If the user is not found, a `404 - NOT FOUND` error will be returned.

## Generating a link to start screen sharing with a user
To start screen sharing with a user, you can make a `POST` request to the
following endpoint:
```
https://api.upscope.io/v1.0/users/:user_id/watch_link.json
```
With `:user_id` being the unique Upscope ID for the user. This is also called
`short_id`.

The content of the request should be a JSON-encoded document with this format:

```json
{
  "observe_view": true,
  "branding": {
    "naked": true,
    "retry_url": "https://your-website.com/admin/screenshare/XYZ",
    "no_bottom_bar": false,
    "no_events": false
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

### Options
| Parameter | Description |
| --------- | ----------- |
| `agent.id` | The ID of the agent (required). |
| `agent.name` | The name of the agent (required). |
| `observe_view` | If true, the agent will be shown the screen share screen instead of the events list if available (i.e. the user is online). |
| `branding.naked` | If true , the Upscope logo will not be shown on the page. This is ideal if you want to display the page inside an iframe. Defaults to false. |
| `branding.retry_url` | If set, the user will be able to click a "Retry now" link that redirects to this url if there are problems. When null , no button is displayed. Defaults to null. |
| `branding.no_bottom_bar` | If true the bar with connection information at the bottom of the screen will not be shown. Defaults to null. |
| `branding.no_events` | If true  the user will not have access to the events list, but only to the user's screen. This is particularly useful used with branding.naked  when embedding the page in an iframe. |

### Errors
- If the user is not found, a `404 - NOT FOUND` error will be returned.
- If the screen share would put you over your subscription limits, a
  `402 - PAYMENT REQUIRED` error will be returned.
