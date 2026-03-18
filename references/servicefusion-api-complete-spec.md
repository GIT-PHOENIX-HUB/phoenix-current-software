# Service Fusion API Reference

**Complete API Specification** -- Generated from RAML spec

| Property | Value |
|----------|-------|
| **Title** | Service Fusion API |
| **Version** | v1 |
| **Base URI** | `https://api.servicefusion.com/{version}` |
| **Protocols** | https |
| **Media Type** | `application/json` |
| **Namespace** | Api |
| **Security** | oauth_2_0 |

### Base URI Parameters

| Parameter | Type | Required | Description | Allowed Values |
|-----------|------|----------|-------------|----------------|
| `version` | string | Yes | Used to send a version of the API to be used. | `v1` |

---

## Documentation

### Getting Started

# Getting Started
The Service Fusion API allows you to programmatically access data stored in your Service Fusion account with ease.

1. You need a valid access token to send requests to the API endpoints. To get your access token see
the [authentication documentation](/v1/#/docs/documentation-1).
2. The API has an access [rate limit](/v1/#/docs/documentation-2) applied to it.
3. The Service Fusion API will only respond to secure communications done over HTTPS. HTTP requests will be sent
a `301` redirect to corresponding HTTPS resources.
4. The request format is controlled by the header `Content-Type` (if not specified, then `application/json` will be used).
The following headers are currently supported:
  - `application/json` - [JSON format](https://en.wikipedia.org/wiki/JSON)
5. The response format is controlled by the header `Accept` (if not specified, then `application/json` will be used).
The following headers are currently supported:
  - `application/json` - [JSON format](https://en.wikipedia.org/wiki/JSON)
  - `application/xml` - [XML format](https://en.wikipedia.org/wiki/XML)

All API requests use the following format: `https://api.servicefusion.com/{version}/{resource}`, where:
- `version` is the version of the API. The current supported version is `v1`.
- `resource` is an API resource. A complete list of all supported resources can be found in the `Resources` tab.

The API has 3 basic operations: 
- Get a list of records.
- Get a single record by ID.
- Create a new record.
  
With each response, the [HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) corresponding to this response is returned:
- `2xx` successful
- `3xx` redirection
- `4xx` client error
- `5xx` server error

## The answers are `2xx` and `3xx`.
### Getting a list of records (`GET /`)
To get the list of records of the selected resource, you must make a `GET` request to this resource. If successful
the response will return with the [HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) `200` with approximately the following contents:
```
{
    "items": [
        {
            "id": "1",
            "first_name": "Max",
            "last_name": "Paltsev"
        },
        {
            "id": "2",
            "first_name": "Jerry",
            "last_name": "Wheeler"
        },
        ...
    ],
    "_meta": {
        "totalCount": 200,
        "pageCount": 20,
        "currentPage": 1,
        "perPage": 10
    }
}
```
This answer contains two root elements - `items` contains an array of records of the current resource and `_meta` contains
service information. Also, access to the data of the service information can be obtained through the headers that are returned with each answer.

| Meta | Header | Description |
| ---- | ------ | ----------- |
| `totalCount` | `X-Pagination-Total-Count` | The total number of resources. |
| `pageCount` | `X-Pagination-Page-Count` | The number of pages. |
| `currentPage` | `X-Pagination-Current-Page` | The current page (1-based). |
| `perPage` | `X-Pagination-Per-Page` | The number of resources per page. |

Additionally the GET operation accepts the following parameters:
- `page` : returns the current page of results. If the specified page number is less than the first or last,
the first or last page will be displayed. Example: `?page=2`. By default, this parameter is set to `1`.
- `per-page` : the number of records displayed per page, from `1` to `50`. Example: `?per-page=20`. Default
this parameter is equal to `10`.
- `sort` : sort the displayed records by the specified fields. Example: `?sort=-name,description` sort all records
in the descending order of the field `name` and in the ascending order of the `description` field.
- `filters` : filtering the displayed records according to the specified criteria.
Example: `?filters[name]=John&filters[description]=Walter`.
- `fields` : a list of the displayed fields in the response, separated by a comma. Example: `?fields=name,description`.
Default displays all fields.

### Getting a record by ID (`GET /{id}`)
To obtain a single record from the selected resoure, you must make a `GET` request to this resource with the ID
of the record being requested. If successful, the response returns with the [HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) `200` with approximately the following
contents:
```
{
    "id": "1",
    "first_name": "Max",
    "last_name": "Paltsev"
}
```
Additionally, the (`GET/{id}`) operation accepts the following paramter:
- `fields` : a list of the displayed fields in the response, separated by a comma.  Example: `?fields=name,description`.

### Creating a new record (`POST /`)
To create a new record for the selected resource, you need to `POST` a request to the resource. If successful
the response will return with the [HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) `201` with approximately the following contents:
```
{
    "id": "1",
    "first_name": "Max",
    "last_name": "Paltsev"
}
```
Additionally, the (`POST /`) operation accepts the following paramter:
- `fields` : a list of the displayed fields in the response, separated by a comma.  Example: `?fields=name,description`.

## The answers are `4xx` and `5xx`.
### Validation error
If there is an error in the create/update validation, a response will be returned with the [HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) `422`
with the following content represented by [Error Validation](/v1/#/docs/types-7):
```
[
    {
        "field": "name",
        "message": "Name is too long (maximum is 45 characters)."
    },
    ...
]
```

### Exception
If other errors occur, the response will be returned with the [HTTP status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) `4xx` or` 5xx` 
with the following content represented by [Error Type](/v1/#/docs/types-2):
```
{
    "code": 500,
    "name": "Internal server error.",
    "message": "Failed to create the object for unknown reason."
}
```


### Authentication

# Authentication
## Overview
An Access Token is required to be sent as part of every request to the Service Fusion API, in the
form of an `Authorization: Bearer {{access_token}}` request header or as query parameter
`?access_token={{access_token}}` in the url. Do not use them together.
An Access Token uniquely identifies you for each API request.

## Get an Access Token
Our API uses the [OAuth 2](https://oauth.net/2/) specification and supports 2
of [RFC-6749's](https://tools.ietf.org/html/rfc6749) grant flows.
### Authorization Code Grant ([4.1](https://tools.ietf.org/html/rfc6749#section-4.1))

> This authentication method allows you to get an access token in exchange for the user's usual credentials to log into
the ServiceFusion account, which he will enter in a pop-up window on your site or any other third-party application. This
method consists of 3 steps and is rather complicated to implement, if you need something simpler please look at the
Client Credentials Grant authentication method below.

1. Before you can implement OAuth 2.0 for your app, you need to register your app in
[OAuth Apps](https://admin.servicefusion.com/developerSettings/oauthApps):
  - In [OAuth Apps](https://admin.servicefusion.com/developerSettings/oauthApps), create new app (if you don't already have
  one) by clicking to `Add New OAuth App`.
  - Enter the Name and Redirect URL. When you implement OAuth 2.0 in your app (see next section), the `redirect_uri` must
  match this URL.
  - Click `Add OAuth App`, you will be redirected to the page with the generated Client ID and Client Secret for your app.
  - Save the generated Client ID and Client Secret of your app, you will need them in the next steps.

2. Once you have registered your app, you can implement OAuth 2.0 in your app's code. Your app should start the authorization
flow by directing the user to the Authorization URL:
  ```
    https://api.servicefusion.com/oauth/authorize
      ?response_type=code
      &client_id=YOUR_APP_CLIENT_ID
      &redirect_uri=YOUR_APP_REDIRECT_URL
      &state=YOUR_USER_BOUND_VALUE
  ```
  Where:
  - `response_type`: (required) Set this to `code`.
  - `client_id`: (required) Set this to the app's Client ID generated for your app in
  [OAuth Apps](https://admin.servicefusion.com/developerSettings/oauthApps).
  - `redirect_uri`: (optional) Set this to the Redirect URL configured for your app in
  [OAuth Apps](https://admin.servicefusion.com/developerSettings/oauthApps) (must be identical).
  - `state`: (optional, but recommended for security) Set this to a value that is associated with the user you are directing
  to the authorization URL, for example, a hash of the user's session ID. Make sure that this is a value that cannot be guessed.

3. After the user directed to the Authorization URL successfully passes authentication, he will be redirected back to the
Redirect URL (which you set for your app in [OAuth Apps](https://admin.servicefusion.com/developerSettings/oauthApps)) with
the `code` and `state` (if indicated in the previous step) query parameters. First check that the `state` value matches what
you set it to originally - this serves as a CSRF protection mechanism and you will ensure an attacker can't intercept the
authorization flow. Then exchange the received query parameter `code` for an access token:

> Note: the `code` query parameter lifetime is 60sec and it can be exchanged only once within this 60sec (it is for security
reasons), otherwise an error message that code invalid or expired will be occured.

```
  curl --request POST \
    --url 'https://api.servicefusion.com/oauth/access_token' \
    --header 'content-type: application/json' \
    --data '{"grant_type": "authorization_code", "client_id": "YOUR_APP_CLIENT_ID", "client_secret": "YOUR_APP_CLIENT_SECRET", "code": "QUERY_PARAMETER_CODE", "redirect_uri": "YOUR_APP_REDIRECT_URL"}'
```
Where:
  - `grant_type`: (required) Set this to `authorization_code`.
  - `client_id`: (required) Set this to the app's Client ID generated for your app in
  [OAuth Apps](https://admin.servicefusion.com/developerSettings/oauthApps).
  - `client_secret`: (required) Set this to the app's Client Secret generated for your app in
  [OAuth Apps](https://admin.servicefusion.com/developerSettings/oauthApps).
  - `code`: (required) Set this to the `code` query parameter that the user received when redirecting to the Redirect URL.
  - `redirect_uri`: (optional) Set this to the `redirect_uri` query parameter which was included in the initial authorization
  request (must be identical).

The [response](/v1/#/docs/types-0) contains an Access Token, the token's type (which is `Bearer`), the time (in seconds,
3600 = 1 hour) when the token expires, and a Refresh Token to refresh your Access Token when it expires. If the request
results in an error, it is represented by an [OAuthTokenError](/v1/#/docs/types-1) in the response.
```
  {
    "access_token": "eyJz93a...k4laUWw",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "afGb76r...t8erDVe"
  }
```
### Client Credentials Grant ([4.4](https://tools.ietf.org/html/rfc6749#section-4.4))

> This authentication method allows you to get an access token in exchange for the client's ID and Secret, which he can find
on the page [OAuth Consumer](https://admin.servicefusion.com/developerSettings/oauthConsumer) of his ServiceFusion account.
If you want a more convenient authorization for a user with his usual credentials to enter the ServiceFusion account, please
look at the Authorization Code Grant authentication method above.

To ask for an Access Token for any of your authorized consumers, perform a `POST` operation to
the `https://api.servicefusion.com/oauth/access_token` endpoint with a payload in the following format:
```
  curl --request POST \
    --url 'https://api.servicefusion.com/oauth/access_token' \
    --header 'content-type: application/json' \
    --data '{"grant_type": "client_credentials", "client_id": "YOUR_USER_CLIENT_ID", "client_secret": "YOUR_USER_CLIENT_SECRET"}'
```
Where:
- `grant_type`: (required) Set this to `client_credentials`.
- `client_id`: (required) Set this to the consumer's Client ID generated for your user in
[OAuth Consumer](https://admin.servicefusion.com/developerSettings/oauthConsumer).
- `client_secret`: (required) Set this to the consumer's Client Secret generated for your user in
[OAuth Consumer](https://admin.servicefusion.com/developerSettings/oauthConsumer).

The [response](/v1/#/docs/types-0) contains an Access Token, the token's type (which is `Bearer`), the time (in seconds,
3600 = 1 hour) when the token expires, and a Refresh Token to refresh your Access Token when it expires. If the request
results in an error, it is represented by an [OAuthTokenError](/v1/#/docs/types-1) in the response.
```
  {
    "access_token": "eyJz93a...k4laUWw",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "afGb76r...t8erDVe"
  }
```
## Refresh an Access Token
When the Access Token expires, you can use the Refresh Token to get a new Access Token by using the
token endpoint as shown below:
```
  curl --request POST \
    --url 'https://api.servicefusion.com/oauth/access_token' \
    --header 'content-type: application/json' \
    --data '{"grant_type": "refresh_token", "refresh_token": "afGb76r...t8erDVe"}'
```
Where:
- `grant_type`: (required) Set this to `refresh_token`.
- `refresh_token`: (required) Set this to `refresh_token` value from the Access Token response.


### Rate Limits

# Rate Limits
API access rate limits are applied to each access token at a rate of 60 requests per minute. In addition, every API response is accompanied
by the following set of headers to identify the status of your consumption. 

| Header | Description |
| ------ | ----------- |
| `X-Rate-Limit-Limit` | The maximum number of requests that the consumer is permitted to make per minute. |
| `X-Rate-Limit-Remaining` | The number of requests remaining in the current rate limit window. |
| `X-Rate-Limit-Reset` | The time at which the current rate limit window resets in UTC epoch seconds. |

If too many requests are received from a user within the stated period of the time, a response with status code
`429` (meaning `Too Many Requests`) will be returned.


---

## Security Schemes

### oauth_2_0

**Type:** OAuth 2.0

This API supports OAuth 2.0 for authenticating all API requests.


#### Query Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. | `eyJz93a...k4laUWw` |

#### Headers

| Header | Type | Required | Description | Example |
|--------|------|----------|-------------|---------|
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. | `Bearer eyJz93a...k4laUWw` |

#### Error Responses

**401 Response**

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**403 Response**

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

#### OAuth Settings

- **accessTokenUri:** https://api.servicefusion.com/oauth/access_token
- **authorizationGrants:** authorization_code, client_credentials

---

## Traits

Traits define shared query parameters applied across resources.

### `tra.formatable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |

### `tra.me-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `first_name`, `last_name`, `email` | `id,email` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |

### `tra.calendarTask-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `type`, `description`, `start_time`, `end_time`, `start_date`, `end_date`, `created_at`, `updated_at`, `is_public`, `is_completed`, `repeat_id`, `users_id`, `customers_id`, `jobs_id`, `estimates_id` | `id,description,is_completed` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `repeat` | `repeat` |

### `tra.calendarTask-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `type`, `description`, `start_time`, `end_time`, `start_date`, `end_date`, `created_at`, `updated_at`, `is_public`, `is_completed`, `repeat_id` | `type,-end_time` |

### `tra.calendarTask-filtrable`

### `tra.customer-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `customer_name`, `fully_qualified_name`, `account_number`, `account_balance`, `private_notes`, `public_notes`, `payment_terms`, `discount`, `discount_type`, `credit_rating`, `labor_charge_type`, `labor_charge_default_rate`, `qbo_sync_token`, `qbo_currency`, `qbo_id`, `qbd_id`, `created_at`, `updated_at`, `last_serviced_date`, `is_bill_for_drive_time`, `is_vip`, `is_taxable`, `parent_customer`, `referral_source`, `agent`, `assigned_contract`, `payment_type`, `tax_item_name`, `industry` | `id,customer_name,discount` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `contacts`, `contacts.phones`, `contacts.emails`, `locations`, `custom_fields` | `contacts.phones,locations` |

### `tra.customer-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `customer_name`, `fully_qualified_name`, `account_number`, `private_notes`, `public_notes`, `payment_terms`, `discount`, `discount_type`, `credit_rating`, `labor_charge_type`, `labor_charge_default_rate`, `qbo_sync_token`, `qbo_currency`, `qbo_id`, `qbd_id`, `created_at`, `updated_at`, `last_serviced_date`, `is_bill_for_drive_time`, `is_vip`, `is_taxable`, `parent_customer`, `referral_source`, `agent`, `assigned_contract`, `payment_type`, `tax_item_name`, `industry` | `-customer_name,created_at` |

### `tra.customer-filtrable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `filters[name]` | string | No | Used to filter results by given name (partial match). |  |  | `John` |
| `filters[contact_first_name]` | string | No | Used to filter results by given contact's first name (partial match). |  |  | `John` |
| `filters[contact_last_name]` | string | No | Used to filter results by given contact's last name (partial match). |  |  | `Walter` |
| `filters[address]` | string | No | Used to filter results by given address (partial match). |  |  | `3210 Midway Ave` |
| `filters[city]` | string | No | Used to filter results by given city (full match). |  |  | `Dallas` |
| `filters[postal_code]` | integer | No | Used to filter results by given postal code (full match). |  |  | `75242` |
| `filters[phone]` | string | No | Used to filter results by given phone (partial match). |  |  | `214-555-1212` |
| `filters[email]` | string | No | Used to filter results by given email (full match). |  |  | `john.walter@gmail.com` |
| `filters[tags]` | string | No | Used to filter results by given tags (full match). Accepted value is comma-separated string. |  |  | `Problem Customer, User` |
| `filters[last_serviced_date][lte]` | string | No | Used to filter results by given `less than or equal` of last serviced date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[last_serviced_date][gte]` | string | No | Used to filter results by given `greater than or equal` of last serviced date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[agreement_date_effective][lte]` | string | No | Used to filter results by given `less than or equal` of agreement date effective (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[agreement_date_effective][gte]` | string | No | Used to filter results by given `greater than or equal` of agreement date effective (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[agreement_date_expires][lte]` | string | No | Used to filter results by given `less than or equal` of agreement date expires (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[agreement_date_expires][gte]` | string | No | Used to filter results by given `greater than or equal` of agreement date expires (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |

### `tra.job-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `check_number`, `priority`, `description`, `tech_notes`, `completion_notes`, `payment_status`, `taxes_fees_total`, `drive_labor_total`, `billable_expenses_total`, `total`, `payments_deposits_total`, `due_total`, `cost_total`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `end_date`, `created_at`, `updated_at`, `closed_at`, `customer_id`, `customer_name`, `parent_customer`, `status`, `sub_status`, `contact_first_name`, `contact_last_name`, `street_1`, `street_2`, `city`, `state_prov`, `postal_code`, `location_name`, `is_gated`, `gate_instructions`, `category`, `source`, `payment_type`, `customer_payment_terms`, `project`, `phase`, `po_number`, `contract`, `note_to_customer`, `called_in_by`, `is_requires_follow_up` | `id,number,description` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `agents`, `custom_fields`, `pictures`, `documents`, `equipment`, `equipment.custom_fields`, `techs_assigned`, `tasks`, `notes`, `products`, `services`, `other_charges`, `labor_charges`, `expenses`, `payments`, `invoices`, `signatures`, `printable_work_order`, `visits`, `visits.techs_assigned` | `agents,equipment.custom_fields,visits.techs_assigned` |

### `tra.job-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `number`, `po_number`, `check_number`, `description`, `tech_notes`, `completion_notes`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `end_date`, `created_at`, `updated_at`, `closed_at`, `customer_id`, `customer_name`, `status`, `sub_status`, `category`, `source`, `payment_type`, `customer_payment_terms`, `contract`, `called_in_by` | `number,-start_date` |

### `tra.job-filtrable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `filters[status]` | string | No | Used to filter results by given statuses (full match). Accepted value is comma-separated string. |  |  | `Job Closed, Cancelled` |
| `filters[number]` | string | No | Used to filter results by given number (partial match). |  |  | `101` |
| `filters[po_number]` | string | No | Used to filter results by given po number (partial match). |  |  | `101` |
| `filters[invoice_number]` | string | No | Used to filter results by given invoice number (partial match). |  |  | `101` |
| `filters[customer_name]` | string | No | Used to filter results by given customer's name (partial match). |  |  | `John Walter` |
| `filters[parent_customer_name]` | string | No | Used to filter results by given parent customer's name (partial match). |  |  | `John Walter` |
| `filters[contact_first_name]` | string | No | Used to filter results by given contact's first name (partial match). |  |  | `John` |
| `filters[contact_last_name]` | string | No | Used to filter results by given contact's last name (partial match). |  |  | `Walter` |
| `filters[address]` | string | No | Used to filter results by given address (partial match). |  |  | `3210 Midway Ave` |
| `filters[city]` | string | No | Used to filter results by given city (full match). |  |  | `Dallas` |
| `filters[zip_code]` | integer | No | Used to filter results by given zip code (full match). |  |  | `75242` |
| `filters[phone]` | string | No | Used to filter results by given phone (partial match). |  |  | `214-555-1212` |
| `filters[email]` | string | No | Used to filter results by given email (full match). |  |  | `john.walter@gmail.com` |
| `filters[category]` | string | No | Used to filter results by given categories (full match). Accepted value is comma-separated string. |  |  | `Install, Service Call` |
| `filters[source]` | string | No | Used to filter results by given sources (full match). Accepted value is comma-separated string. |  |  | `Google, Yelp` |
| `filters[start_date][lte]` | string | No | Used to filter results by given `less than or equal` of start date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[start_date][gte]` | string | No | Used to filter results by given `greater than or equal` of start date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[end_date][lte]` | string | No | Used to filter results by given `less than or equal` of end date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[end_date][gte]` | string | No | Used to filter results by given `greater than or equal` of end date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[updated_date][lte]` | string | No | Used to filter results by given `less than or equal` of updated date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[updated_date][gte]` | string | No | Used to filter results by given `greater than or equal` of updated date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[closed_date][lte]` | string | No | Used to filter results by given `less than or equal` of closed date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[closed_date][gte]` | string | No | Used to filter results by given `greater than or equal` of closed date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |

### `tra.jobCategory-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `name` | `id` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |

### `tra.jobCategory-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `name` | `-id,name` |

### `tra.jobCategory-filtrable`

### `tra.jobStatus-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `code`, `name`, `is_custom`, `category` | `id,code,is_custom` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |

### `tra.jobStatus-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `code`, `name`, `is_custom`, `category` | `-id,code` |

### `tra.jobStatus-filtrable`

### `tra.estimate-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `description`, `tech_notes`, `payment_status`, `taxes_fees_total`, `total`, `due_total`, `cost_total`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `created_at`, `updated_at`, `customer_id`, `customer_name`, `parent_customer`, `status`, `sub_status`, `contact_first_name`, `contact_last_name`, `street_1`, `street_2`, `city`, `state_prov`, `postal_code`, `location_name`, `is_gated`, `gate_instructions`, `category`, `source`, `payment_type`, `customer_payment_terms`, `project`, `phase`, `po_number`, `contract`, `note_to_customer`, `opportunity_rating`, `opportunity_owner` | `id,tech_notes` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `agents`, `custom_fields`, `pictures`, `documents`, `equipment`, `equipment.custom_fields`, `techs_assigned`, `tasks`, `notes`, `products`, `services`, `other_charges`, `payments`, `signatures`, `printable_work_order`, `tags` | `agents,printable_work_order` |

### `tra.estimate-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `number`, `po_number`, `description`, `tech_notes`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `created_at`, `updated_at`, `customer_id`, `customer_name`, `status`, `sub_status`, `category`, `source`, `payment_type`, `customer_payment_terms`, `contract`, `opportunity_rating` | `number,-start_date` |

### `tra.estimate-filtrable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `filters[status]` | string | No | Used to filter results by given statuses (full match). Accepted value is comma-separated string. |  |  | `Estimate Closed, Cancelled` |
| `filters[number]` | string | No | Used to filter results by given number (partial match). |  |  | `101` |
| `filters[po_number]` | string | No | Used to filter results by given po number (partial match). |  |  | `101` |
| `filters[customer_name]` | string | No | Used to filter results by given customer's name (partial match). |  |  | `John Walter` |
| `filters[parent_customer_name]` | string | No | Used to filter results by given parent customer's name (partial match). |  |  | `John Walter` |
| `filters[contact_first_name]` | string | No | Used to filter results by given contact's first name (partial match). |  |  | `John` |
| `filters[contact_last_name]` | string | No | Used to filter results by given contact's last name (partial match). |  |  | `Walter` |
| `filters[address]` | string | No | Used to filter results by given address (partial match). |  |  | `3210 Midway Ave` |
| `filters[city]` | string | No | Used to filter results by given city (full match). |  |  | `Dallas` |
| `filters[zip_code]` | integer | No | Used to filter results by given zip code (full match). |  |  | `75242` |
| `filters[phone]` | string | No | Used to filter results by given phone (partial match). |  |  | `214-555-1212` |
| `filters[email]` | string | No | Used to filter results by given email (full match). |  |  | `john.walter@gmail.com` |
| `filters[category]` | string | No | Used to filter results by given categories (full match). Accepted value is comma-separated string. |  |  | `Install, Service Call` |
| `filters[source]` | string | No | Used to filter results by given sources (full match). Accepted value is comma-separated string. |  |  | `Google, Yelp` |
| `filters[start_date][lte]` | string | No | Used to filter results by given `less than or equal` of start date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[start_date][gte]` | string | No | Used to filter results by given `greater than or equal` of start date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[end_date][lte]` | string | No | Used to filter results by given `less than or equal` of end date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[end_date][gte]` | string | No | Used to filter results by given `greater than or equal` of end date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[requested_date][lte]` | string | No | Used to filter results by given `less than or equal` of requested date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[requested_date][gte]` | string | No | Used to filter results by given `greater than or equal` of requested date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |

### `tra.invoice-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `currency`, `po_number`, `terms`, `customer_message`, `notes`, `pay_online_url`, `qbo_invoice_no`, `qbo_sync_token`, `qbo_synced_date`, `qbo_id`, `qbd_id`, `total`, `is_paid`, `date`, `mail_send_date`, `created_at`, `updated_at`, `customer`, `customer_contact`, `payment_terms`, `bill_to_customer_id`, `bill_to_customer_location_id`, `bill_to_customer_contact_id`, `bill_to_email_id`, `bill_to_phone_id` | `id,notes` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |

### `tra.invoice-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `number`, `currency`, `po_number`, `terms`, `customer_message`, `notes`, `qbo_invoice_no`, `qbo_sync_token`, `qbo_synced_date`, `qbo_id`, `qbd_id`, `total`, `is_paid`, `date`, `mail_send_date`, `created_at`, `updated_at`, `customer`, `customer_contact`, `payment_terms`, `bill_to_customer_id`, `bill_to_customer_location_id`, `bill_to_customer_contact_id`, `bill_to_email_id`, `bill_to_phone_id` | `created_at,-number` |

### `tra.invoice-filtrable`

### `tra.paymentType-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `code`, `short_name`, `type`, `is_custom` | `id,short_name` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |

### `tra.paymentType-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `code`, `short_name`, `type`, `is_custom` | `type` |

### `tra.paymentType-filtrable`

### `tra.source-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `short_name`, `long_name` | `id,short_name` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |

### `tra.source-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `short_name`, `long_name` | `id,-long_name` |

### `tra.source-filtrable`

### `tra.tech-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `first_name`, `last_name`, `nickname_on_workorder`, `nickname_on_dispatch`, `color_code`, `email`, `phone_1`, `phone_2`, `gender`, `department`, `title`, `bio`, `is_phone_1_mobile`, `is_phone_1_visible_to_client`, `is_phone_2_mobile`, `is_phone_2_visible_to_client`, `is_sales_rep`, `is_field_worker`, `created_at`, `updated_at` | `id,created_at,updated_at` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |

### `tra.tech-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `first_name`, `last_name`, `nickname_on_workorder`, `nickname_on_dispatch`, `color_code`, `email`, `phone_1`, `phone_2`, `gender`, `department`, `title`, `bio`, `is_phone_1_mobile`, `is_phone_1_visible_to_client`, `is_phone_2_mobile`, `is_phone_2_visible_to_client`, `is_sales_rep`, `is_field_worker`, `created_at`, `updated_at` | `created_at,-first_name` |

### `tra.tech-filtrable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `filters[first_name]` | string | No | Used to filter results by given first name (partial match). |  |  | `Justin` |
| `filters[last_name]` | string | No | Used to filter results by given last name (partial match). |  |  | `Wormell` |
| `filters[email]` | string | No | Used to filter results by given email (partial match). |  |  | `@servicefusion.com` |
| `filters[nickname_on_workorder]` | string | No | Used to filter results by given nickname on workorder (partial match). |  |  | `Workorder Heating` |
| `filters[nickname_on_dispatch]` | string | No | Used to filter results by given nickname on dispatch (partial match). |  |  | `Dispatch Heating` |

### `tra.equipment-fieldable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `type`, `make`, `model`, `sku`, `serial_number`, `location`, `notes`, `extended_warranty_provider`, `is_extended_warranty`, `extended_warranty_date`, `warranty_date`, `install_date`, `created_at`, `updated_at`, `customer_id`, `customer`, `customer_location` | `id,location,customer_location` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `custom_fields` | `custom_fields` |

### `tra.equipment-sortable`

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `type`, `make`, `model`, `sku`, `serial_number`, `location`, `notes`, `extended_warranty_provider`, `is_extended_warranty`, `extended_warranty_date`, `warranty_date`, `install_date`, `created_at`, `updated_at`, `customer_id`, `customer`, `customer_location` | `created_at,-type` |

### `tra.equipment-filtrable`

---

## Data Types

All data types used in the API, with full property definitions.

### `OAuthToken`

An authentication schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `access_token` | string | No | The access token string as issued by the authorization server. |  |
| `token_type` | string | No | The type of token this is. |  |
| `expires_in` | integer | No | The duration of time the access token is granted for. |  |
| `refresh_token` | string | No | When an access token expires (exceeds the `expires_in` time), the `refresh_token` is used to obtain a new access token. |  |

### `OAuthTokenError`

An authentication error's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `error` | string | No | The error title. |  |
| `error_description` | string | No | The error description. |  |

### `typ.Error`

An error's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

### `typ.400Error`

Bad request client's error schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

### `typ.404Error`

Not found client's error schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

### `typ.405Error`

Method not allowed client's error schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

### `typ.415Error`

Unsupported media type client's error schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

### `typ.422Error`

Unprocessable entity client's error schema.

**Base type:** `array`

### `typ.429Error`

Too many requests client's error schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

### `typ.500Error`

Internal server's error schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

### `typ.Agent`

An agent's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

### `typ.AgentBody`

An agent's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | Used to send the agent's identifier that will be searched. If this field is set then the entry will be searched by it, otherwise the search will be performed by its full name. | Default: If not passed, it takes the value from full name search entry. |
| `first_name` | string | No | Used to send the agent's first name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |
| `last_name` | string | No | Used to send the agent's last name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |

### `typ.AssignedTech`

An assigned tech's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

### `typ.AssignedTechBody`

An assigned tech's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | Used to send the assigned tech's identifier that will be searched. If this field is set then the entry will be searched by it, otherwise the search will be performed by its full name. | Default: If not passed, it takes the value from full name search entry. |
| `first_name` | string | No | Used to send the assigned tech's first name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |
| `last_name` | string | No | Used to send the assigned tech's last name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |

### `typ.CalendarTask`

A calendar task's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The calendar task's identifier. |  |
| `type` | string | No | The calendar task's type. |  |
| `description` | string | No | The calendar task's description. |  |
| `start_time` | string | No | The calendar task's start time. |  |
| `end_time` | string | No | The calendar task's end time. |  |
| `start_date` | datetime | No | The calendar task's start date. |  |
| `end_date` | datetime | No | The calendar task's end date. |  |
| `created_at` | datetime | No | The calendar task's created date. |  |
| `updated_at` | datetime | No | The calendar task's updated date. |  |
| `is_public` | boolean | No | The calendar task's is public flag. |  |
| `is_completed` | boolean | No | The calendar task's is completed flag. |  |
| `repeat_id` | integer | No | The calendar task's repeat id. |  |
| `users_id` | array[integer] | Yes | The calendar task's users list of identifiers. |  |
| `customers_id` | array[integer] | Yes | The calendar task's customers list of identifiers. |  |
| `jobs_id` | array[integer] | Yes | The calendar task's jobs list of identifiers. |  |
| `estimates_id` | array[integer] | Yes | The calendar task's estimates list of identifiers. |  |
| `repeat` | object | No | The calendar task's repeat. | Example: `{   "id": 92,   "repeat_type": "Daily",   "repeat_frequency": 2,   "repeat_weekly_days": [],   "repeat_monthly_type": null,   "stop_repeat_type": "On Occurrence",   "stop_repeat_on_occurrence": 10,   "stop_repeat_on_date": null,   "start_date": "2021-05-27T00:00:00+00:00",   "end_date": "2021-06-14T00:00:00+00:00" }` |

#### `typ.CalendarTask.repeat` (nested object)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The repeat's identifier. |  |
| `repeat_type` | string | No | The repeat's type. |  |
| `repeat_frequency` | integer | No | The repeat's frequency. |  |
| `repeat_weekly_days` | array[string] | Yes | The repeat's weekly days list. |  |
| `repeat_monthly_type` | string | No | The repeat's monthly type. |  |
| `stop_repeat_type` | string | No | The repeat's stop type. |  |
| `stop_repeat_on_occurrence` | integer | No | The repeat's stop on occurrence. |  |
| `stop_repeat_on_date` | datetime | No | The repeat's stop on date. |  |
| `start_date` | datetime | No | The repeat's start date. |  |
| `end_date` | datetime | No | The repeat's end date. |  |

### `typ.CalendarTaskView`

A calendar task's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The calendar task's identifier. |  |
| `type` | string | No | The calendar task's type. |  |
| `description` | string | No | The calendar task's description. |  |
| `start_time` | string | No | The calendar task's start time. |  |
| `end_time` | string | No | The calendar task's end time. |  |
| `start_date` | datetime | No | The calendar task's start date. |  |
| `end_date` | datetime | No | The calendar task's end date. |  |
| `created_at` | datetime | No | The calendar task's created date. |  |
| `updated_at` | datetime | No | The calendar task's updated date. |  |
| `is_public` | boolean | No | The calendar task's is public flag. |  |
| `is_completed` | boolean | No | The calendar task's is completed flag. |  |
| `repeat_id` | integer | No | The calendar task's repeat id. |  |
| `users_id` | array[integer] | Yes | The calendar task's users list of identifiers. |  |
| `customers_id` | array[integer] | Yes | The calendar task's customers list of identifiers. |  |
| `jobs_id` | array[integer] | Yes | The calendar task's jobs list of identifiers. |  |
| `estimates_id` | array[integer] | Yes | The calendar task's estimates list of identifiers. |  |
| `repeat` | object | No | The calendar task's repeat. | Example: `{   "id": 92,   "repeat_type": "Daily",   "repeat_frequency": 2,   "repeat_weekly_days": [],   "repeat_monthly_type": null,   "stop_repeat_type": "On Occurrence",   "stop_repeat_on_occurrence": 10,   "stop_repeat_on_date": null,   "start_date": "2021-05-27T00:00:00+00:00",   "end_date": "2021-06-14T00:00:00+00:00" }` |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

#### `typ.CalendarTaskView.repeat` (nested object)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The repeat's identifier. |  |
| `repeat_type` | string | No | The repeat's type. |  |
| `repeat_frequency` | integer | No | The repeat's frequency. |  |
| `repeat_weekly_days` | array[string] | Yes | The repeat's weekly days list. |  |
| `repeat_monthly_type` | string | No | The repeat's monthly type. |  |
| `stop_repeat_type` | string | No | The repeat's stop type. |  |
| `stop_repeat_on_occurrence` | integer | No | The repeat's stop on occurrence. |  |
| `stop_repeat_on_date` | datetime | No | The repeat's stop on date. |  |
| `start_date` | datetime | No | The repeat's start date. |  |
| `end_date` | datetime | No | The repeat's end date. |  |

### `typ.CalendarTaskRepeat`

A calendar task repeat's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The repeat's identifier. |  |
| `repeat_type` | string | No | The repeat's type. |  |
| `repeat_frequency` | integer | No | The repeat's frequency. |  |
| `repeat_weekly_days` | array[string] | Yes | The repeat's weekly days list. |  |
| `repeat_monthly_type` | string | No | The repeat's monthly type. |  |
| `stop_repeat_type` | string | No | The repeat's stop type. |  |
| `stop_repeat_on_occurrence` | integer | No | The repeat's stop on occurrence. |  |
| `stop_repeat_on_date` | datetime | No | The repeat's stop on date. |  |
| `start_date` | datetime | No | The repeat's start date. |  |
| `end_date` | datetime | No | The repeat's end date. |  |

### `typ.CustomField`

A custom field's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

### `typ.CustomFieldBody`

A custom field's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

### `typ.Customer`

A customer's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The customer's identifier. |  |
| `customer_name` | string | No | The customer's name. |  |
| `fully_qualified_name` | string | No | The customer's fully qualified name. |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the customer (Note: `header` - [string] the parent customer's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `account_number` | string | No | The customer's account number. |  |
| `account_balance` | number (float) | No | The customer's account balance. |  |
| `private_notes` | string | No | The customer's private notes. |  |
| `public_notes` | string | No | The customer's public notes. |  |
| `credit_rating` | string | No | The customer's credit rating. |  |
| `labor_charge_type` | string | No | The customer's labor charge type. |  |
| `labor_charge_default_rate` | number (float) | No | The customer's labor charge default rate. |  |
| `last_serviced_date` | datetime | No | The customer's last serviced date. |  |
| `is_bill_for_drive_time` | boolean | No | The customer's is bill for drive time flag. |  |
| `is_vip` | boolean | No | The customer's is vip flag. |  |
| `referral_source` | string | No | The `header` of attached referral source to the customer (Note: `header` - [string] the referral source's fields concatenated by pattern `{short_name}`). |  |
| `agent` | string | No | The `header` of attached agent to the customer (Note: `header` - [string] the agent's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `discount` | number (float) | No | The customer's discount. |  |
| `discount_type` | string | No | The customer's discount type. |  |
| `payment_type` | string | No | The `header` of attached payment type to the customer (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `payment_terms` | string | No | The customer's payment terms. |  |
| `assigned_contract` | string | No | The `header` of attached contract to the customer (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `industry` | string | No | The `header` of attached industry to the customer (Note: `header` - [string] the industry's fields concatenated by pattern `{industry}`). |  |
| `is_taxable` | boolean | No | The customer's is taxable flag. |  |
| `tax_item_name` | string | No | The `header` of attached tax item to the customer (Note: `header` - [string] the tax item's fields concatenated by pattern `{short_name}` with space as separator). |  |
| `qbo_sync_token` | integer | No | The customer's qbo sync token. |  |
| `qbo_currency` | string | No | The customer's qbo currency. |  |
| `qbo_id` | integer | No | The customer's qbo id. |  |
| `qbd_id` | string | No | The customer's qbd id. |  |
| `created_at` | datetime | No | The customer's created date. |  |
| `updated_at` | datetime | No | The customer's updated date. |  |
| `contacts` | array[object] | No | The customer's contacts list. |  |
| `locations` | array[object] | No | The customer's locations list. |  |
| `custom_fields` | array[object] | No | The customer's custom fields list. |  |

#### `typ.Customer.contacts[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | The contact's prefix. |  |
| `fname` | string | No | The contact's first name. |  |
| `lname` | string | No | The contact's last name. |  |
| `suffix` | string | No | The contact's suffix. |  |
| `contact_type` | string | No | The contact's type. |  |
| `dob` | string | No | The contact's dob. |  |
| `anniversary` | string | No | The contact's anniversary. |  |
| `job_title` | string | No | The contact's job title. |  |
| `department` | string | No | The contact's department. |  |
| `created_at` | datetime | No | The contact's created date. |  |
| `updated_at` | datetime | No | The contact's updated date. |  |
| `is_primary` | boolean | No | The contact's is primary flag. |  |
| `phones` | array[object] | No | The contact's phones list. |  |
| `emails` | array[object] | No | The contact's emails list. |  |

##### `typ.Customer.contacts[].phones[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | No | The phone's number. |  |
| `ext` | integer | No | The phone's extension. |  |
| `type` | string | No | The phone's type. |  |
| `created_at` | datetime | No | The phone's created date. |  |
| `updated_at` | datetime | No | The phone's updated date. |  |
| `is_mobile` | boolean | No | The phone's is mobile flag. |  |

##### `typ.Customer.contacts[].emails[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | No | The email's address. |  |
| `class` | string | No | The email's class. |  |
| `types_accepted` | string | No | The email's types accepted. |  |
| `created_at` | datetime | No | The email's created date. |  |
| `updated_at` | datetime | No | The email's updated date. |  |

#### `typ.Customer.locations[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | No | The location's street 1. |  |
| `street_2` | string | No | The location's street 2. |  |
| `city` | string | No | The location's city. |  |
| `state_prov` | string | No | The location's state. |  |
| `postal_code` | string | No | The location's postal code. |  |
| `country` | string | No | The location's country. |  |
| `nickname` | string | No | The location's nickname. |  |
| `gate_instructions` | string | No | The location's gate instructions. |  |
| `latitude` | number (float) | No | The location's latitude. |  |
| `longitude` | number (float) | No | The location's longitude. |  |
| `location_type` | string | No | The location's type. |  |
| `created_at` | datetime | No | The location's created date. |  |
| `updated_at` | datetime | No | The location's updated date. |  |
| `is_primary` | boolean | No | The location's is primary flag. |  |
| `is_gated` | boolean | No | The location's is gated flag. |  |
| `is_bill_to` | boolean | No | The location's is bill to flag. |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the location (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

#### `typ.Customer.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

### `typ.CustomerBody`

A customer's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `customer_name` | string | Yes | Used to send the customer's name that will be set. | Max length: 255 |
| `parent_customer` | string | No | Used to send a parent customer's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the parent customer's identifier, `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `account_number` | string | No | Used to send the customer's account number that will be set. | Max length: 255; Default: If not passed, it takes generated new one. |
| `private_notes` | string | No | Used to send the customer's private notes that will be set. |  |
| `public_notes` | string | No | Used to send the customer's public notes that will be set. |  |
| `credit_rating` | string | No | Used to send the customer's credit rating that will be set. | Enum: `A+`, `A`, `B+`, `B`, `C+`, `C`, `U`; Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `labor_charge_type` | string | No | Used to send the customer's labor charge type that will be set. | Enum: `flat`, `hourly`; Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `labor_charge_default_rate` | number (float) | No | Used to send the customer's labor charge default rate that will be set. | Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `last_serviced_date` | datetime | No | Used to send the customer's last serviced date that will be set. |  |
| `is_bill_for_drive_time` | boolean | No | Used to send the customer's is bill for drive time flag that will be set. | Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `is_vip` | boolean | No | Used to send the customer's is vip flag that will be set. | Default: False |
| `referral_source` | string | No | Used to send a referral source's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the referral source's identifier, `header` - [string] the referral source's fields concatenated by pattern `{short_name}`). |  |
| `agent` | string | No | Used to send an agent's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the agent's identifier, `header` - [string] the agent's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `discount` | number (float) | No | Used to send the customer's discount that will be set. | Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `discount_type` | string | No | Used to send the customer's discount type that will be set. | Enum: `$`, `%`; Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `payment_type` | string | No | Used to send a payment type's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the payment type's identifier, `header` - [string] the payment type's fields concatenated by pattern `{name}`). | Default: If not passed, it takes the value from the company preferences or from parent customer (configurable into the company preferences). |
| `payment_terms` | string | No | Used to send the customer's payment terms that will be set. | Default: If not passed, it takes the value from the company preferences or from parent customer (configurable into the company preferences). |
| `assigned_contract` | string | No | Used to send an assigned contract's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the assigned contract's identifier, `header` - [string] the assigned contract's fields concatenated by pattern `{contract_title}`). |  |
| `industry` | string | No | Used to send an industry's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the industry's identifier, `header` - [string] the industry's fields concatenated by pattern `{industry}`). |  |
| `is_taxable` | boolean | No | Used to send the customer's is taxable flag that will be set. | Default: If not passed, it takes the value `true` (configurable into the company preferences). |
| `tax_item_name` | string | No | Used to send a tax item's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the tax item's identifier, `header` - [string] the tax item's fields concatenated by pattern `{short_name}`). | Default: If not passed, it takes the value from the company preferences (configurable into the company preferences). |
| `qbo_sync_token` | integer | No | Used to send the customer's qbo sync token that will be set. |  |
| `qbo_currency` | string | No | Used to send the customer's qbo currency that will be set. | Enum: `USD`, `CAD`, `JMD`, `THB`; Default: If not passed, it takes the value from the company if it was configured, otherwise it takes the value `USD`. |
| `contacts` | array[object] | No | Used to send the customer's contacts list that will be set. | Default: If not passed, it creates the new one. |
| `locations` | array[object] | No | Used to send the customer's locations list that will be set. | Default: array |
| `custom_fields` | array[object] | No | Used to send the customer's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |

#### `typ.CustomerBody.contacts[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | Used to send the contact's prefix that will be set. | Enum: `Mr.`, `Mrs.`, `Ms.`, `Dr.`, `Atty.`, `Prof.`, `Hon.`, `Gov.`, `Ofc.`, `Rep.`, `Sen.`, `Amb.`, `Sec.`, `Pvt.`, `Cpl.`, `Sgt.`, `Adm.`, `Gen.`, `Maj.`, `Capt.`, `Cmdr.`, `Lt.`, `Lt Col.`, `Col.` |
| `fname` | string | Yes | Used to send the contact's first name that will be set. | Max length: 255 |
| `lname` | string | Yes | Used to send the contact's last name that will be set. | Max length: 255 |
| `suffix` | string | No | Used to send the contact's suffix that will be set. | Max length: 255 |
| `contact_type` | string | No | Used to send the contact's type that will be set. | Max length: 255 |
| `dob` | string | No | Used to send the contact's dob that will be set. | Max length: 250 |
| `anniversary` | string | No | Used to send the contact's anniversary that will be set. | Max length: 250 |
| `job_title` | string | No | Used to send the contact's job title that will be set. | Max length: 200 |
| `department` | string | No | Used to send the contact's department that will be set. | Max length: 200 |
| `is_primary` | boolean | No | Used to send the contact's is primary flag that will be set. When it is passed as `true`, then the customer's existing primary contact (if any) will become secondary, and this one will become the primary one. | Default: If not passed and the customer does not have primary contact, it takes the value `true`, else if the customer already have primary contact, it takes the value `false`. |
| `phones` | array[object] | No | Used to send the contact's phones list that will be set. | Default: array |
| `emails` | array[object] | No | Used to send the contact's emails list that will be set. | Default: array |

##### `typ.CustomerBody.contacts[].phones[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | Yes | Used to send the phone's number that will be set. | Pattern: `^[0-9]{3}\-[0-9]{3}\-[0-9]{4}$` |
| `ext` | integer | No | Used to send the phone's extension that will be set. |  |
| `type` | string | No | Used to send the phone's type that will be set. | Enum: `Mobile`, `Home`, `Work`, `Other` |

##### `typ.CustomerBody.contacts[].emails[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | Yes | Used to send the email's address that will be set. | Max length: 255; Pattern: `^[^@]*<[a-zA-Z0-9!#$%&\'*+\\/=?^_`{\|}~-]+(?:\.[a-zA-Z0-9!#$%&\'*+\\/=?^_`{\|}~-]+)*@(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?>$` |
| `class` | string | No | Used to send the email's class that will be set. | Enum: `Personal`, `Business`, `Other` |
| `types_accepted` | string | No | Used to send the email's types accepted that will be set. Accepted value is comma-separated string. | Enum: `CONF`, `STATUS`, `PMT`, `INV` |

#### `typ.CustomerBody.locations[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | Yes | Used to send the location's street 1 that will be set. | Max length: 255 |
| `street_2` | string | No | Used to send the location's street 2 that will be set. | Max length: 255 |
| `city` | string | No | Used to send the location's city that will be set. | Max length: 255 |
| `state_prov` | string | No | Used to send the location's state that will be set. | Max length: 255 |
| `postal_code` | string | No | Used to send the location's postal code that will be set. | Max length: 255 |
| `country` | string | No | Used to send the location's country that will be set. | Max length: 255 |
| `nickname` | string | No | Used to send the location's nickname that will be set. | Max length: 255 |
| `gate_instructions` | string | No | Used to send the location's gate instructions that will be set. |  |
| `latitude` | number (float) | No | Used to send the location's latitude that will be set. | Default: 0 |
| `longitude` | number (float) | No | Used to send the location's longitude that will be set. | Default: 0 |
| `location_type` | string | No | Used to send the location's type that will be set. | Max length: 200 |
| `is_primary` | boolean | No | Used to send the location's is primary flag that will be set. When it is passed as `true`, then the customer's existing primary location (if any) will become secondary, and this one will become the primary one. | Default: If not passed and the customer does not have primary location, it takes the value `true`, else if the customer already have primary location, it takes the value `false`. |
| `is_gated` | boolean | No | Used to send the location's `is gated` flag that will be set. | Default: False |
| `is_bill_to` | boolean | No | Used to send the location's is bill to flag that will be set. | Default: False |
| `customer_contact` | string | No | Used to send a customer contact's `id` or `header` that will be attached to the location (Note: `id` - [integer] the customer contact's identifier, `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

#### `typ.CustomerBody.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

### `typ.CustomerView`

A customer's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The customer's identifier. |  |
| `customer_name` | string | No | The customer's name. |  |
| `fully_qualified_name` | string | No | The customer's fully qualified name. |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the customer (Note: `header` - [string] the parent customer's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `account_number` | string | No | The customer's account number. |  |
| `account_balance` | number (float) | No | The customer's account balance. |  |
| `private_notes` | string | No | The customer's private notes. |  |
| `public_notes` | string | No | The customer's public notes. |  |
| `credit_rating` | string | No | The customer's credit rating. |  |
| `labor_charge_type` | string | No | The customer's labor charge type. |  |
| `labor_charge_default_rate` | number (float) | No | The customer's labor charge default rate. |  |
| `last_serviced_date` | datetime | No | The customer's last serviced date. |  |
| `is_bill_for_drive_time` | boolean | No | The customer's is bill for drive time flag. |  |
| `is_vip` | boolean | No | The customer's is vip flag. |  |
| `referral_source` | string | No | The `header` of attached referral source to the customer (Note: `header` - [string] the referral source's fields concatenated by pattern `{short_name}`). |  |
| `agent` | string | No | The `header` of attached agent to the customer (Note: `header` - [string] the agent's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `discount` | number (float) | No | The customer's discount. |  |
| `discount_type` | string | No | The customer's discount type. |  |
| `payment_type` | string | No | The `header` of attached payment type to the customer (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `payment_terms` | string | No | The customer's payment terms. |  |
| `assigned_contract` | string | No | The `header` of attached contract to the customer (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `industry` | string | No | The `header` of attached industry to the customer (Note: `header` - [string] the industry's fields concatenated by pattern `{industry}`). |  |
| `is_taxable` | boolean | No | The customer's is taxable flag. |  |
| `tax_item_name` | string | No | The `header` of attached tax item to the customer (Note: `header` - [string] the tax item's fields concatenated by pattern `{short_name}` with space as separator). |  |
| `qbo_sync_token` | integer | No | The customer's qbo sync token. |  |
| `qbo_currency` | string | No | The customer's qbo currency. |  |
| `qbo_id` | integer | No | The customer's qbo id. |  |
| `qbd_id` | string | No | The customer's qbd id. |  |
| `created_at` | datetime | No | The customer's created date. |  |
| `updated_at` | datetime | No | The customer's updated date. |  |
| `contacts` | array[object] | No | The customer's contacts list. |  |
| `locations` | array[object] | No | The customer's locations list. |  |
| `custom_fields` | array[object] | No | The customer's custom fields list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

#### `typ.CustomerView.contacts[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | The contact's prefix. |  |
| `fname` | string | No | The contact's first name. |  |
| `lname` | string | No | The contact's last name. |  |
| `suffix` | string | No | The contact's suffix. |  |
| `contact_type` | string | No | The contact's type. |  |
| `dob` | string | No | The contact's dob. |  |
| `anniversary` | string | No | The contact's anniversary. |  |
| `job_title` | string | No | The contact's job title. |  |
| `department` | string | No | The contact's department. |  |
| `created_at` | datetime | No | The contact's created date. |  |
| `updated_at` | datetime | No | The contact's updated date. |  |
| `is_primary` | boolean | No | The contact's is primary flag. |  |
| `phones` | array[object] | No | The contact's phones list. |  |
| `emails` | array[object] | No | The contact's emails list. |  |

##### `typ.CustomerView.contacts[].phones[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | No | The phone's number. |  |
| `ext` | integer | No | The phone's extension. |  |
| `type` | string | No | The phone's type. |  |
| `created_at` | datetime | No | The phone's created date. |  |
| `updated_at` | datetime | No | The phone's updated date. |  |
| `is_mobile` | boolean | No | The phone's is mobile flag. |  |

##### `typ.CustomerView.contacts[].emails[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | No | The email's address. |  |
| `class` | string | No | The email's class. |  |
| `types_accepted` | string | No | The email's types accepted. |  |
| `created_at` | datetime | No | The email's created date. |  |
| `updated_at` | datetime | No | The email's updated date. |  |

#### `typ.CustomerView.locations[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | No | The location's street 1. |  |
| `street_2` | string | No | The location's street 2. |  |
| `city` | string | No | The location's city. |  |
| `state_prov` | string | No | The location's state. |  |
| `postal_code` | string | No | The location's postal code. |  |
| `country` | string | No | The location's country. |  |
| `nickname` | string | No | The location's nickname. |  |
| `gate_instructions` | string | No | The location's gate instructions. |  |
| `latitude` | number (float) | No | The location's latitude. |  |
| `longitude` | number (float) | No | The location's longitude. |  |
| `location_type` | string | No | The location's type. |  |
| `created_at` | datetime | No | The location's created date. |  |
| `updated_at` | datetime | No | The location's updated date. |  |
| `is_primary` | boolean | No | The location's is primary flag. |  |
| `is_gated` | boolean | No | The location's is gated flag. |  |
| `is_bill_to` | boolean | No | The location's is bill to flag. |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the location (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

#### `typ.CustomerView.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

### `typ.CustomerContact`

A customer contact's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | The contact's prefix. |  |
| `fname` | string | No | The contact's first name. |  |
| `lname` | string | No | The contact's last name. |  |
| `suffix` | string | No | The contact's suffix. |  |
| `contact_type` | string | No | The contact's type. |  |
| `dob` | string | No | The contact's dob. |  |
| `anniversary` | string | No | The contact's anniversary. |  |
| `job_title` | string | No | The contact's job title. |  |
| `department` | string | No | The contact's department. |  |
| `created_at` | datetime | No | The contact's created date. |  |
| `updated_at` | datetime | No | The contact's updated date. |  |
| `is_primary` | boolean | No | The contact's is primary flag. |  |
| `phones` | array[object] | No | The contact's phones list. |  |
| `emails` | array[object] | No | The contact's emails list. |  |

#### `typ.CustomerContact.phones[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | No | The phone's number. |  |
| `ext` | integer | No | The phone's extension. |  |
| `type` | string | No | The phone's type. |  |
| `created_at` | datetime | No | The phone's created date. |  |
| `updated_at` | datetime | No | The phone's updated date. |  |
| `is_mobile` | boolean | No | The phone's is mobile flag. |  |

#### `typ.CustomerContact.emails[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | No | The email's address. |  |
| `class` | string | No | The email's class. |  |
| `types_accepted` | string | No | The email's types accepted. |  |
| `created_at` | datetime | No | The email's created date. |  |
| `updated_at` | datetime | No | The email's updated date. |  |

### `typ.CustomerContactBody`

A customer contact's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | Used to send the contact's prefix that will be set. | Enum: `Mr.`, `Mrs.`, `Ms.`, `Dr.`, `Atty.`, `Prof.`, `Hon.`, `Gov.`, `Ofc.`, `Rep.`, `Sen.`, `Amb.`, `Sec.`, `Pvt.`, `Cpl.`, `Sgt.`, `Adm.`, `Gen.`, `Maj.`, `Capt.`, `Cmdr.`, `Lt.`, `Lt Col.`, `Col.` |
| `fname` | string | Yes | Used to send the contact's first name that will be set. | Max length: 255 |
| `lname` | string | Yes | Used to send the contact's last name that will be set. | Max length: 255 |
| `suffix` | string | No | Used to send the contact's suffix that will be set. | Max length: 255 |
| `contact_type` | string | No | Used to send the contact's type that will be set. | Max length: 255 |
| `dob` | string | No | Used to send the contact's dob that will be set. | Max length: 250 |
| `anniversary` | string | No | Used to send the contact's anniversary that will be set. | Max length: 250 |
| `job_title` | string | No | Used to send the contact's job title that will be set. | Max length: 200 |
| `department` | string | No | Used to send the contact's department that will be set. | Max length: 200 |
| `is_primary` | boolean | No | Used to send the contact's is primary flag that will be set. When it is passed as `true`, then the customer's existing primary contact (if any) will become secondary, and this one will become the primary one. | Default: If not passed and the customer does not have primary contact, it takes the value `true`, else if the customer already have primary contact, it takes the value `false`. |
| `phones` | array[object] | No | Used to send the contact's phones list that will be set. | Default: array |
| `emails` | array[object] | No | Used to send the contact's emails list that will be set. | Default: array |

#### `typ.CustomerContactBody.phones[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | Yes | Used to send the phone's number that will be set. | Pattern: `^[0-9]{3}\-[0-9]{3}\-[0-9]{4}$` |
| `ext` | integer | No | Used to send the phone's extension that will be set. |  |
| `type` | string | No | Used to send the phone's type that will be set. | Enum: `Mobile`, `Home`, `Work`, `Other` |

#### `typ.CustomerContactBody.emails[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | Yes | Used to send the email's address that will be set. | Max length: 255; Pattern: `^[^@]*<[a-zA-Z0-9!#$%&\'*+\\/=?^_`{\|}~-]+(?:\.[a-zA-Z0-9!#$%&\'*+\\/=?^_`{\|}~-]+)*@(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?>$` |
| `class` | string | No | Used to send the email's class that will be set. | Enum: `Personal`, `Business`, `Other` |
| `types_accepted` | string | No | Used to send the email's types accepted that will be set. Accepted value is comma-separated string. | Enum: `CONF`, `STATUS`, `PMT`, `INV` |

### `typ.CustomerEmail`

A customer email's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | No | The email's address. |  |
| `class` | string | No | The email's class. |  |
| `types_accepted` | string | No | The email's types accepted. |  |
| `created_at` | datetime | No | The email's created date. |  |
| `updated_at` | datetime | No | The email's updated date. |  |

### `typ.CustomerEmailBody`

A customer email's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | Yes | Used to send the email's address that will be set. | Max length: 255; Pattern: `^[^@]*<[a-zA-Z0-9!#$%&\'*+\\/=?^_`{\|}~-]+(?:\.[a-zA-Z0-9!#$%&\'*+\\/=?^_`{\|}~-]+)*@(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?>$` |
| `class` | string | No | Used to send the email's class that will be set. | Enum: `Personal`, `Business`, `Other` |
| `types_accepted` | string | No | Used to send the email's types accepted that will be set. Accepted value is comma-separated string. | Enum: `CONF`, `STATUS`, `PMT`, `INV` |

### `typ.CustomerLocation`

A customer location's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | No | The location's street 1. |  |
| `street_2` | string | No | The location's street 2. |  |
| `city` | string | No | The location's city. |  |
| `state_prov` | string | No | The location's state. |  |
| `postal_code` | string | No | The location's postal code. |  |
| `country` | string | No | The location's country. |  |
| `nickname` | string | No | The location's nickname. |  |
| `gate_instructions` | string | No | The location's gate instructions. |  |
| `latitude` | number (float) | No | The location's latitude. |  |
| `longitude` | number (float) | No | The location's longitude. |  |
| `location_type` | string | No | The location's type. |  |
| `created_at` | datetime | No | The location's created date. |  |
| `updated_at` | datetime | No | The location's updated date. |  |
| `is_primary` | boolean | No | The location's is primary flag. |  |
| `is_gated` | boolean | No | The location's is gated flag. |  |
| `is_bill_to` | boolean | No | The location's is bill to flag. |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the location (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

### `typ.CustomerLocationBody`

A customer location's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | Yes | Used to send the location's street 1 that will be set. | Max length: 255 |
| `street_2` | string | No | Used to send the location's street 2 that will be set. | Max length: 255 |
| `city` | string | No | Used to send the location's city that will be set. | Max length: 255 |
| `state_prov` | string | No | Used to send the location's state that will be set. | Max length: 255 |
| `postal_code` | string | No | Used to send the location's postal code that will be set. | Max length: 255 |
| `country` | string | No | Used to send the location's country that will be set. | Max length: 255 |
| `nickname` | string | No | Used to send the location's nickname that will be set. | Max length: 255 |
| `gate_instructions` | string | No | Used to send the location's gate instructions that will be set. |  |
| `latitude` | number (float) | No | Used to send the location's latitude that will be set. | Default: 0 |
| `longitude` | number (float) | No | Used to send the location's longitude that will be set. | Default: 0 |
| `location_type` | string | No | Used to send the location's type that will be set. | Max length: 200 |
| `is_primary` | boolean | No | Used to send the location's is primary flag that will be set. When it is passed as `true`, then the customer's existing primary location (if any) will become secondary, and this one will become the primary one. | Default: If not passed and the customer does not have primary location, it takes the value `true`, else if the customer already have primary location, it takes the value `false`. |
| `is_gated` | boolean | No | Used to send the location's `is gated` flag that will be set. | Default: False |
| `is_bill_to` | boolean | No | Used to send the location's is bill to flag that will be set. | Default: False |
| `customer_contact` | string | No | Used to send a customer contact's `id` or `header` that will be attached to the location (Note: `id` - [integer] the customer contact's identifier, `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

### `typ.CustomerPhone`

A customer phone's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | No | The phone's number. |  |
| `ext` | integer | No | The phone's extension. |  |
| `type` | string | No | The phone's type. |  |
| `created_at` | datetime | No | The phone's created date. |  |
| `updated_at` | datetime | No | The phone's updated date. |  |
| `is_mobile` | boolean | No | The phone's is mobile flag. |  |

### `typ.CustomerPhoneBody`

A customer phone's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | Yes | Used to send the phone's number that will be set. | Pattern: `^[0-9]{3}\-[0-9]{3}\-[0-9]{4}$` |
| `ext` | integer | No | Used to send the phone's extension that will be set. |  |
| `type` | string | No | Used to send the phone's type that will be set. | Enum: `Mobile`, `Home`, `Work`, `Other` |

### `typ.Equipment`

An equipment's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

#### `typ.Equipment.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

### `typ.EquipmentBody`

An equipment's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | string | No | Used to send the equipment's identifier that will be searched. You may pass this parameter if you do not want to create new entry but assign existing one. You may assign by `identifier` or `header` (Note: `identifier` - [integer] the equipment's identifier, `header` - [string] the equipment's fields concatenated by pattern `{type}:{make}:{model}:{serial_number}` with colon as separator). | Default: If not passed, it creates new one. |
| `type` | string | No | Used to send the equipment's type that will be set. |  |
| `make` | string | No | Used to send the equipment's make that will be set. |  |
| `model` | string | No | Used to send the equipment's model that will be set. | Max length: 255 |
| `sku` | string | No | Used to send the equipment's sku that will be set. | Max length: 50 |
| `serial_number` | string | No | Used to send the equipment's serial number that will be set. | Max length: 255 |
| `location` | string | No | Used to send the equipment's location that will be set. | Max length: 255 |
| `notes` | string | No | Used to send the equipment's notes that will be set. | Max length: 250 |
| `extended_warranty_provider` | string | No | Used to send the equipment's extended warranty provider that will be set. | Max length: 255 |
| `is_extended_warranty` | boolean | No | Used to send the equipment's is extended warranty flag that will be set. | Default: False |
| `extended_warranty_date` | datetime | No | Used to send the equipment's extended warranty date that will be set. |  |
| `warranty_date` | datetime | No | Used to send the equipment's warranty date that will be set. |  |
| `install_date` | datetime | No | Used to send the equipment's install date that will be set. |  |
| `customer_location` | string | No | Used to send a customer location's `id` or `header` that will be attached to the equipment (Note: `id` - [integer] the customer location's identifier, `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | Used to send the equipment's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |

#### `typ.EquipmentBody.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

### `typ.EquipmentView`

An equipment's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

#### `typ.EquipmentView.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

### `typ.Estimate`

An estimate's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The estimate's identifier. |  |
| `number` | string | No | The estimate's number. |  |
| `description` | string | No | The estimate's description. |  |
| `tech_notes` | string | No | The estimate's tech notes. |  |
| `customer_payment_terms` | string | No | The estimate's customer payment terms. |  |
| `payment_status` | string | No | The estimate's payment status. |  |
| `taxes_fees_total` | number (float) | No | The estimate's taxes and fees total. |  |
| `total` | number (float) | No | The estimate's total. |  |
| `due_total` | number (float) | No | The estimate's due total. |  |
| `cost_total` | number (float) | No | The estimate's cost total. |  |
| `duration` | integer | No | The estimate's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The estimate's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The estimate's time frame promised end. |  |
| `start_date` | datetime | No | The estimate's start date. |  |
| `created_at` | datetime | No | The estimate's created date. |  |
| `updated_at` | datetime | No | The estimate's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the estimate (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the estimate (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the estimate (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the estimate (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the estimate (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The estimate's contact first name. |  |
| `contact_last_name` | string | No | The estimate's contact last name. |  |
| `street_1` | string | No | The estimate's location street 1. |  |
| `street_2` | string | No | The estimate's location street 2. |  |
| `city` | string | No | The estimate's location city. |  |
| `state_prov` | string | No | The estimate's location state prov. |  |
| `postal_code` | string | No | The estimate's location postal code. |  |
| `location_name` | string | No | The estimate's location name. |  |
| `is_gated` | boolean | No | The estimate's location is gated flag. |  |
| `gate_instructions` | string | No | The estimate's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the estimate (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the estimate (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the estimate (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `project` | string | No | The `header` of attached project to the estimate (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the estimate (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The estimate's po number. |  |
| `contract` | string | No | The `header` of attached contract to the estimate (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The estimate's note to customer. |  |
| `opportunity_rating` | integer | No | The estimate's opportunity rating. |  |
| `opportunity_owner` | string | No | The `header` of attached opportunity owner to the estimate (Note: `header` - [string] the opportunity owner's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `agents` | array[object] | No | The estimate's agents list. |  |
| `custom_fields` | array[object] | No | The estimate's custom fields list. |  |
| `pictures` | array[object] | No | The estimate's pictures list. |  |
| `documents` | array[object] | No | The estimate's documents list. |  |
| `equipment` | array[object] | No | The estimate's equipments list. |  |
| `techs_assigned` | array[object] | No | The estimate's techs assigned list. |  |
| `tasks` | array[object] | No | The estimate's tasks list. |  |
| `notes` | array[object] | No | The estimate's notes list. |  |
| `products` | array[object] | No | The estimate's products list. |  |
| `services` | array[object] | No | The estimate's services list. |  |
| `other_charges` | array[object] | No | The estimate's other charges list. |  |
| `payments` | array[object] | No | The estimate's payments list. |  |
| `signatures` | array[object] | No | The estimate's signatures list. |  |
| `printable_work_order` | array[object] | No | The estimate's printable work order list. |  |
| `tags` | array[object] | No | The estimate's tags list. |  |

#### `typ.Estimate.agents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

#### `typ.Estimate.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

#### `typ.Estimate.pictures[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

#### `typ.Estimate.documents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

#### `typ.Estimate.equipment[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

##### `typ.Estimate.equipment[].custom_fields[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

#### `typ.Estimate.techs_assigned[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

#### `typ.Estimate.tasks[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

#### `typ.Estimate.notes[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

#### `typ.Estimate.products[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.Estimate.services[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.Estimate.other_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.Estimate.payments[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

#### `typ.Estimate.signatures[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

#### `typ.Estimate.printable_work_order[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

#### `typ.Estimate.tags[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | No | The tag's unique tag. |  |
| `created_at` | datetime | No | The tag's created date. |  |
| `updated_at` | datetime | No | The tag's updated date. |  |

### `typ.EstimateBody`

An estimate's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `description` | string | No | Used to send the estimate's description that will be set. |  |
| `tech_notes` | string | No | Used to send the estimate's tech notes that will be set. |  |
| `duration` | integer | No | Used to send the estimate's duration (in seconds) that will be set. | Min: 0; Max: 86400; Default: 3600 |
| `time_frame_promised_start` | string | No | Used to send the estimate's time frame promised start that will be set. |  |
| `time_frame_promised_end` | string | No | Used to send the estimate's time frame promised end that will be set. |  |
| `start_date` | datetime | No | Used to send the estimate's start date that will be set. |  |
| `created_at` | datetime | No | Used to send the estimate's created date that will be set. | Default: If not passed, it takes the value as current date and time. |
| `customer_name` | string | Yes | Used to send a customer's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the customer's identifier, `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | Used to send a status'es `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the status'es identifier, `header` - [string] the status'es fields concatenated by pattern `{name}`). | Default: If not passed, it takes the default status for estimates. |
| `contact_first_name` | string | No | Used to send the estimate's contact first name that will be set. If a contact with the passed name and surname already exists, then a new contact will not be created, but the existing one will be attached. | Max length: 255; Default: If not passed, it takes the first name from primary contact of the customer (if exists), otherwise a primary contact will be created for the customer. |
| `contact_last_name` | string | No | Used to send the estimate's contact last name that will be set. If a contact with the passed name and surname already exists, then a new contact will not be created, but the existing one will be attached. | Max length: 255; Default: If not passed, it takes the last name from primary contact of the customer (if exists), otherwise a primary contact will be created for the customer. |
| `street_1` | string | No | Used to send the estimate's location street 1 that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `street_2` | string | No | Used to send the estimate's location street 2 that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `city` | string | No | Used to send the estimate's location city that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `state_prov` | string | No | Used to send the estimate's location state prov that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `postal_code` | string | No | Used to send the estimate's location postal code that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `location_name` | string | No | Used to send the estimate's location name that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `is_gated` | boolean | No | Used to send the estimate's location is gated flag that will be set. | Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `gate_instructions` | string | No | Used to send the estimate's location gate instructions that will be set. | Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `category` | string | No | Used to send a category's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the category's identifier, `header` - [string] the category's fields concatenated by pattern `{category}`). Optionally required (configurable into the company preferences). |  |
| `source` | string | No | Used to send a source's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the source's identifier, `header` - [string] the source's fields concatenated by pattern `{short_name}`). | Default: If not passed, it takes the value from the customer. |
| `project` | string | No | Used to send a project's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the project's identifier, `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | Used to send a phase's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the phase's identifier, `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | Used to send the estimate's po number that will be set. | Max length: 255 |
| `contract` | string | No | Used to send a contract's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the contract's identifier, `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). | Default: If not passed, it takes the value from the customer. |
| `note_to_customer` | string | No | Used to send the estimate's note to customer that will be set. | Default: If not passed, it takes the value from the company preferences. |
| `opportunity_rating` | integer | No | Used to send the estimate's opportunity rating that will be set. | Min: 0; Max: 5 |
| `opportunity_owner` | string | No | Used to send an opportunity owner's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the opportunity owner's identifier, `header` - [string] the opportunity owner's fields concatenated by pattern `{first_name} {last_name}` with space as separator). | Default: If not passed, it takes the value from the authenticated user. |
| `custom_fields` | array[object] | No | Used to send the estimate's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |
| `equipment` | array[object] | No | Used to send the estimate's equipments list that will be set. | Default: array |
| `techs_assigned` | array[object] | No | Used to send the estimate's techs assigned list that will be set. | Default: array |
| `tasks` | array[object] | No | Used to send the estimate's tasks list that will be set. | Default: array |
| `notes` | array[object] | No | Used to send the estimate's notes list that will be set. | Default: array |
| `products` | array[object] | No | Used to send the estimate's products list that will be set. | Default: array |
| `services` | array[object] | No | Used to send the estimate's services list that will be set. | Default: array |
| `other_charges` | array[object] | No | Used to send the estimate's other charges list that will be set. | Default: If not passed, it creates all entries with `auto added` option enabled. Also it creates all not passed other charges declared into `products` and `services`. |
| `tags` | array[object] | No | Used to send the estimate's tags list that will be set. | Default: array |

#### `typ.EstimateBody.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

#### `typ.EstimateBody.equipment[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | string | No | Used to send the equipment's identifier that will be searched. You may pass this parameter if you do not want to create new entry but assign existing one. You may assign by `identifier` or `header` (Note: `identifier` - [integer] the equipment's identifier, `header` - [string] the equipment's fields concatenated by pattern `{type}:{make}:{model}:{serial_number}` with colon as separator). | Default: If not passed, it creates new one. |
| `type` | string | No | Used to send the equipment's type that will be set. |  |
| `make` | string | No | Used to send the equipment's make that will be set. |  |
| `model` | string | No | Used to send the equipment's model that will be set. | Max length: 255 |
| `sku` | string | No | Used to send the equipment's sku that will be set. | Max length: 50 |
| `serial_number` | string | No | Used to send the equipment's serial number that will be set. | Max length: 255 |
| `location` | string | No | Used to send the equipment's location that will be set. | Max length: 255 |
| `notes` | string | No | Used to send the equipment's notes that will be set. | Max length: 250 |
| `extended_warranty_provider` | string | No | Used to send the equipment's extended warranty provider that will be set. | Max length: 255 |
| `is_extended_warranty` | boolean | No | Used to send the equipment's is extended warranty flag that will be set. | Default: False |
| `extended_warranty_date` | datetime | No | Used to send the equipment's extended warranty date that will be set. |  |
| `warranty_date` | datetime | No | Used to send the equipment's warranty date that will be set. |  |
| `install_date` | datetime | No | Used to send the equipment's install date that will be set. |  |
| `customer_location` | string | No | Used to send a customer location's `id` or `header` that will be attached to the equipment (Note: `id` - [integer] the customer location's identifier, `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | Used to send the equipment's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |

##### `typ.EstimateBody.equipment[].custom_fields[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

#### `typ.EstimateBody.techs_assigned[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | Used to send the assigned tech's identifier that will be searched. If this field is set then the entry will be searched by it, otherwise the search will be performed by its full name. | Default: If not passed, it takes the value from full name search entry. |
| `first_name` | string | No | Used to send the assigned tech's first name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |
| `last_name` | string | No | Used to send the assigned tech's last name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |

#### `typ.EstimateBody.tasks[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `description` | string | Yes | Used to send the task's description that will be set. | Max length: 500 |
| `is_completed` | boolean | No | Used to send the task's is completed flag that will be set. | Default: False |

#### `typ.EstimateBody.notes[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | Yes | Used to send the note's text that will be set. |  |

#### `typ.EstimateBody.products[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the product's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed product. |
| `description` | string | No | Used to send the product's description that will be set. | Default: If not passed, it takes the value of passed product. |
| `multiplier` | integer | No | Used to send the product's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed product. |
| `rate` | number (float) | No | Used to send the product's rate that will be set. | Default: If not passed, it takes the value of passed product. |
| `cost` | number (float) | No | Used to send the product's cost that will be set. | Default: If not passed, it takes the value of passed product. |
| `is_show_rate_items` | boolean | No | Used to send the product's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the product (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | Yes | Used to send a product's `id` or `header` that will be attached to the product (Note: `id` - [integer] the product's identifier, `header` - [string] the product's fields concatenated by pattern `{make}`). |  |

#### `typ.EstimateBody.services[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the service's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed service. |
| `description` | string | No | Used to send the service's description that will be set. | Default: If not passed, it takes the value of passed service. |
| `multiplier` | integer | No | Used to send the service's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed service. |
| `rate` | number (float) | No | Used to send the service's rate that will be set. | Default: If not passed, it takes the value of passed service. |
| `cost` | number (float) | No | Used to send the service's cost that will be set. | Default: If not passed, it takes the value of passed service. |
| `is_show_rate_items` | boolean | No | Used to send the service's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the service (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | Yes | Used to send a service's `id` or `header` that will be attached to the service (Note: `id` - [integer] the service's identifier, `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |

#### `typ.EstimateBody.other_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the other charge's name that will be set. | Max length: 255; Default: If not passed, it takes the value of passed other charge. |
| `rate` | number (float) | No | Used to send the other charge's rate that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `is_percentage` | boolean | No | Used to send the other charge's is percentage flag that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `other_charge` | string | Yes | Used to send an other charge's `id` or `header` that will be attached to the other charge (Note: `id` - [integer] the other charge's identifier, `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |

#### `typ.EstimateBody.tags[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | Yes | Used to send the tag's unique tag that will be set. | Max length: 254 |

### `typ.EstimateView`

An estimate's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The estimate's identifier. |  |
| `number` | string | No | The estimate's number. |  |
| `description` | string | No | The estimate's description. |  |
| `tech_notes` | string | No | The estimate's tech notes. |  |
| `customer_payment_terms` | string | No | The estimate's customer payment terms. |  |
| `payment_status` | string | No | The estimate's payment status. |  |
| `taxes_fees_total` | number (float) | No | The estimate's taxes and fees total. |  |
| `total` | number (float) | No | The estimate's total. |  |
| `due_total` | number (float) | No | The estimate's due total. |  |
| `cost_total` | number (float) | No | The estimate's cost total. |  |
| `duration` | integer | No | The estimate's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The estimate's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The estimate's time frame promised end. |  |
| `start_date` | datetime | No | The estimate's start date. |  |
| `created_at` | datetime | No | The estimate's created date. |  |
| `updated_at` | datetime | No | The estimate's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the estimate (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the estimate (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the estimate (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the estimate (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the estimate (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The estimate's contact first name. |  |
| `contact_last_name` | string | No | The estimate's contact last name. |  |
| `street_1` | string | No | The estimate's location street 1. |  |
| `street_2` | string | No | The estimate's location street 2. |  |
| `city` | string | No | The estimate's location city. |  |
| `state_prov` | string | No | The estimate's location state prov. |  |
| `postal_code` | string | No | The estimate's location postal code. |  |
| `location_name` | string | No | The estimate's location name. |  |
| `is_gated` | boolean | No | The estimate's location is gated flag. |  |
| `gate_instructions` | string | No | The estimate's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the estimate (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the estimate (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the estimate (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `project` | string | No | The `header` of attached project to the estimate (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the estimate (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The estimate's po number. |  |
| `contract` | string | No | The `header` of attached contract to the estimate (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The estimate's note to customer. |  |
| `opportunity_rating` | integer | No | The estimate's opportunity rating. |  |
| `opportunity_owner` | string | No | The `header` of attached opportunity owner to the estimate (Note: `header` - [string] the opportunity owner's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `agents` | array[object] | No | The estimate's agents list. |  |
| `custom_fields` | array[object] | No | The estimate's custom fields list. |  |
| `pictures` | array[object] | No | The estimate's pictures list. |  |
| `documents` | array[object] | No | The estimate's documents list. |  |
| `equipment` | array[object] | No | The estimate's equipments list. |  |
| `techs_assigned` | array[object] | No | The estimate's techs assigned list. |  |
| `tasks` | array[object] | No | The estimate's tasks list. |  |
| `notes` | array[object] | No | The estimate's notes list. |  |
| `products` | array[object] | No | The estimate's products list. |  |
| `services` | array[object] | No | The estimate's services list. |  |
| `other_charges` | array[object] | No | The estimate's other charges list. |  |
| `payments` | array[object] | No | The estimate's payments list. |  |
| `signatures` | array[object] | No | The estimate's signatures list. |  |
| `printable_work_order` | array[object] | No | The estimate's printable work order list. |  |
| `tags` | array[object] | No | The estimate's tags list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

#### `typ.EstimateView.agents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

#### `typ.EstimateView.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

#### `typ.EstimateView.pictures[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

#### `typ.EstimateView.documents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

#### `typ.EstimateView.equipment[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

##### `typ.EstimateView.equipment[].custom_fields[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

#### `typ.EstimateView.techs_assigned[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

#### `typ.EstimateView.tasks[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

#### `typ.EstimateView.notes[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

#### `typ.EstimateView.products[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.EstimateView.services[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.EstimateView.other_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.EstimateView.payments[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

#### `typ.EstimateView.signatures[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

#### `typ.EstimateView.printable_work_order[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

#### `typ.EstimateView.tags[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | No | The tag's unique tag. |  |
| `created_at` | datetime | No | The tag's created date. |  |
| `updated_at` | datetime | No | The tag's updated date. |  |

### `typ.Invoice`

An invoice's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |

### `typ.InvoiceView`

An invoice's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

### `typ.Job`

A job's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job's identifier. |  |
| `number` | string | No | The job's number. |  |
| `check_number` | string | No | The job's check number. |  |
| `priority` | string | No | The job's priority. |  |
| `description` | string | No | The job's description. |  |
| `tech_notes` | string | No | The job's tech notes. |  |
| `completion_notes` | string | No | The job's completion notes. |  |
| `payment_status` | string | No | The job's payment status. |  |
| `taxes_fees_total` | number (float) | No | The job's taxes and fees total. |  |
| `drive_labor_total` | number (float) | No | The job's drive and labor total. |  |
| `billable_expenses_total` | number (float) | No | The job's billable expenses total. |  |
| `total` | number (float) | No | The job's total. |  |
| `payments_deposits_total` | number (float) | No | The job's payments and deposits total. |  |
| `due_total` | number (float) | No | The job's due total. |  |
| `cost_total` | number (float) | No | The job's cost total. |  |
| `duration` | integer | No | The job's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The job's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The job's time frame promised end. |  |
| `start_date` | datetime | No | The job's start date. |  |
| `end_date` | datetime | No | The job's end date. |  |
| `created_at` | datetime | No | The job's created date. |  |
| `updated_at` | datetime | No | The job's updated date. |  |
| `closed_at` | datetime | No | The job's closed date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the job (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the job (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the job (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the job (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the job (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The job's contact first name. |  |
| `contact_last_name` | string | No | The job's contact last name. |  |
| `street_1` | string | No | The job's location street 1. |  |
| `street_2` | string | No | The job's location street 2. |  |
| `city` | string | No | The job's location city. |  |
| `state_prov` | string | No | The job's location state prov. |  |
| `postal_code` | string | No | The job's location postal code. |  |
| `location_name` | string | No | The job's location name. |  |
| `is_gated` | boolean | No | The job's location is gated flag. |  |
| `gate_instructions` | string | No | The job's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the job (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the job (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the job (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `customer_payment_terms` | string | No | The `header` of attached customer payment term to the job (Note: `header` - [string] the customer payment term's fields concatenated by pattern `{name}`). |  |
| `project` | string | No | The `header` of attached project to the job (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the job (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The job's po number. |  |
| `contract` | string | No | The `header` of attached contract to the job (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The job's note to customer. |  |
| `called_in_by` | string | No | The job's called in by. |  |
| `is_requires_follow_up` | boolean | No | The job's is requires follow up flag. |  |
| `agents` | array[object] | No | The job's agents list. |  |
| `custom_fields` | array[object] | No | The job's custom fields list. |  |
| `pictures` | array[object] | No | The job's pictures list. |  |
| `documents` | array[object] | No | The job's documents list. |  |
| `equipment` | array[object] | No | The job's equipments list. |  |
| `techs_assigned` | array[object] | No | The job's techs assigned list. |  |
| `tasks` | array[object] | No | The job's tasks list. |  |
| `notes` | array[object] | No | The job's notes list. |  |
| `products` | array[object] | No | The job's products list. |  |
| `services` | array[object] | No | The job's services list. |  |
| `other_charges` | array[object] | No | The job's other charges list. |  |
| `labor_charges` | array[object] | No | The job's labor charges list. |  |
| `expenses` | array[object] | No | The job's expenses list. |  |
| `payments` | array[object] | No | The job's payments list. |  |
| `invoices` | array[object] | No | The job's invoices list. |  |
| `signatures` | array[object] | No | The job's signatures list. |  |
| `printable_work_order` | array[object] | No | The job's printable work order list. |  |
| `visits` | array[object] | No | The job's visits list. |  |

#### `typ.Job.agents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

#### `typ.Job.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

#### `typ.Job.pictures[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

#### `typ.Job.documents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

#### `typ.Job.equipment[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

##### `typ.Job.equipment[].custom_fields[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

#### `typ.Job.techs_assigned[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

#### `typ.Job.tasks[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

#### `typ.Job.notes[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

#### `typ.Job.products[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.Job.services[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.Job.other_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.Job.labor_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | The labor charge's drive time. |  |
| `drive_time_rate` | number (float) | No | The labor charge's drive time rate. |  |
| `drive_time_cost` | number (float) | No | The labor charge's drive time cost. |  |
| `drive_time_start` | string | No | The labor charge's drive time start. |  |
| `drive_time_end` | string | No | The labor charge's drive time end. |  |
| `is_drive_time_billed` | boolean | No | The labor charge's is drive time billed flag. |  |
| `labor_time` | integer | No | The labor charge's labor time. |  |
| `labor_time_rate` | number (float) | No | The labor charge's labor time rate. |  |
| `labor_time_cost` | number (float) | No | The labor charge's labor time cost. |  |
| `labor_time_start` | string | No | The labor charge's labor time start. |  |
| `labor_time_end` | string | No | The labor charge's labor time end. |  |
| `labor_date` | datetime | No | The labor charge's labor date. |  |
| `is_labor_time_billed` | boolean | No | The labor charge's is labor time billed flag. |  |
| `total` | number (float) | No | The labor charge's total. |  |
| `created_at` | datetime | No | The labor charge's created date. |  |
| `updated_at` | datetime | No | The labor charge's updated date. |  |
| `is_status_generated` | boolean | No | The labor charge's is status generated flag. |  |
| `user` | string | No | The `header` of attached user to the labor charge (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `visit_id` | integer | No | The `id` of attached visit to the labor charge (Note: `id` - [integer] the visit's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the labor charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the labor charge (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.Job.expenses[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | The expense's purchased from. |  |
| `notes` | string | No | The expense's notes. |  |
| `amount` | number (float) | No | The expense's amount. |  |
| `is_billable` | boolean | No | The expense's is billable flag. |  |
| `date` | datetime | No | The expense's date. |  |
| `created_at` | datetime | No | The expense's created date. |  |
| `updated_at` | datetime | No | The expense's updated date. |  |
| `user` | string | No | The `header` of attached user to the expense (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | The `header` of attached category to the expense (Note: `header` - [string] the category's fields concatenated by pattern `{category_name}`). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the expense (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the expense (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.Job.payments[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

#### `typ.Job.invoices[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |

#### `typ.Job.signatures[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

#### `typ.Job.printable_work_order[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

#### `typ.Job.visits[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes_for_techs` | string | No | The visit's notes for techs. |  |
| `time_frame_promised_start` | string | No | The visit's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The visit's time frame promised end. |  |
| `duration` | integer | No | The visit's duration (in seconds). |  |
| `is_text_notified` | boolean | No | The visit's is text notified flag. |  |
| `is_voice_notified` | boolean | No | The visit's is voice notified flag. |  |
| `start_date` | datetime | No | The visit's start date. |  |
| `techs_assigned` | array[object] | No | The visit's techs assigned list. |  |

##### `typ.Job.visits[].techs_assigned[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |
| `status` | string | No | The assigned tech's status. |  |

### `typ.JobBody`

A job's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `check_number` | string | No | Used to send the job's check number that will be set. |  |
| `priority` | string | No | Used to send the job's priority that will be set. | Enum: `Low`, `Normal`, `High`; Default: Normal |
| `description` | string | No | Used to send the job's description that will be set. |  |
| `tech_notes` | string | No | Used to send the job's tech notes that will be set. |  |
| `completion_notes` | string | No | Used to send the job's completion notes that will be set. |  |
| `duration` | integer | No | Used to send the job's duration (in seconds) that will be set. | Min: 0; Max: 86400; Default: 3600 |
| `time_frame_promised_start` | string | No | Used to send the job's time frame promised start that will be set. |  |
| `time_frame_promised_end` | string | No | Used to send the job's time frame promised end that will be set. |  |
| `start_date` | datetime | No | Used to send the job's start date that will be set. |  |
| `end_date` | datetime | No | Used to send the job's end date that will be set. |  |
| `customer_name` | string | Yes | Used to send a customer's `id` or `header` that will be attached to the job (Note: `id` - [integer] the customer's identifier, `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | Used to send a status'es `id` or `header` that will be attached to the job (Note: `id` - [integer] the status'es identifier, `header` - [string] the status'es fields concatenated by pattern `{name}`). Optionally required (configurable into the company preferences). | Default: If not passed, it takes the default status for jobs. |
| `contact_first_name` | string | No | Used to send the job's contact first name that will be set. If a contact with the passed name and surname already exists, then a new contact will not be created, but the existing one will be attached. | Max length: 255; Default: If not passed, it takes the first name from primary contact of the customer (if exists), otherwise a primary contact will be created for the customer. |
| `contact_last_name` | string | No | Used to send the job's contact last name that will be set. If a contact with the passed name and surname already exists, then a new contact will not be created, but the existing one will be attached. | Max length: 255; Default: If not passed, it takes the last name from primary contact of the customer (if exists), otherwise a primary contact will be created for the customer. |
| `street_1` | string | No | Used to send the job's location street 1 that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `street_2` | string | No | Used to send the job's location street 2 that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `city` | string | No | Used to send the job's location city that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `state_prov` | string | No | Used to send the job's location state prov that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `postal_code` | string | No | Used to send the job's location postal code that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `location_name` | string | No | Used to send the job's location name that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `is_gated` | boolean | No | Used to send the job's location is gated flag that will be set. | Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `gate_instructions` | string | No | Used to send the job's location gate instructions that will be set. | Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `category` | string | No | Used to send a category's `id` or `header` that will be attached to the job (Note: `id` - [integer] the category's identifier, `header` - [string] the category's fields concatenated by pattern `{category}`). Optionally required (configurable into the company preferences). |  |
| `source` | string | No | Used to send a source's `id` or `header` that will be attached to the job (Note: `id` - [integer] the source's identifier, `header` - [string] the source's fields concatenated by pattern `{short_name}`). | Default: If not passed, it takes the value from the customer. |
| `payment_type` | string | No | Used to send a payment type's `id` or `header` that will be attached to the job (Note: `id` - [integer] the payment type's identifier, `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). Optionally required (configurable into the company preferences). | Default: If not passed, it takes the value from the customer. |
| `customer_payment_terms` | string | No | Used to send a customer payment term's `id` or `header` that will be attached to the job (Note: `id` - [integer] the customer payment term's identifier, `header` - [string] the customer payment term's fields concatenated by pattern `{name}`). | Default: If not passed, it takes the value from the customer. |
| `project` | string | No | Used to send a project's `id` or `header` that will be attached to the job (Note: `id` - [integer] the project's identifier, `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | Used to send a phase's `id` or `header` that will be attached to the job (Note: `id` - [integer] the phase's identifier, `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | Used to send the job's po number that will be set. | Max length: 255 |
| `contract` | string | No | Used to send a contract's `id` or `header` that will be attached to the job (Note: `id` - [integer] the contract's identifier, `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). | Default: If not passed, it takes the value from the customer. |
| `note_to_customer` | string | No | Used to send the job's note to customer that will be set. | Default: If not passed, it takes the value from the company preferences. |
| `called_in_by` | string | No | Used to send the job's called in by that will be set. |  |
| `is_requires_follow_up` | boolean | No | Used to send the job's is requires follow up flag that will be set. | Default: False |
| `agents` | array[object] | No | Used to send the job's agents list that will be set. | Default: array |
| `custom_fields` | array[object] | No | Used to send the job's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |
| `equipment` | array[object] | No | Used to send the job's equipments list that will be set. | Default: array |
| `techs_assigned` | array[object] | No | Used to send the job's techs assigned list that will be set. | Default: array |
| `tasks` | array[object] | No | Used to send the job's tasks list that will be set. | Default: array |
| `notes` | array[object] | No | Used to send the job's notes list that will be set. | Default: array |
| `products` | array[object] | No | Used to send the job's products list that will be set. | Default: array |
| `services` | array[object] | No | Used to send the job's services list that will be set. | Default: array |
| `other_charges` | array[object] | No | Used to send the job's other charges list that will be set. | Default: If not passed, it creates all entries with `auto added` option enabled. Also it creates all not passed other charges declared into `products` and `services`. |
| `labor_charges` | array[object] | No | Used to send the job's labor charges list that will be set. | Default: array |
| `expenses` | array[object] | No | Used to send the job's expenses list that will be set. | Default: array |

#### `typ.JobBody.agents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | Used to send the agent's identifier that will be searched. If this field is set then the entry will be searched by it, otherwise the search will be performed by its full name. | Default: If not passed, it takes the value from full name search entry. |
| `first_name` | string | No | Used to send the agent's first name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |
| `last_name` | string | No | Used to send the agent's last name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |

#### `typ.JobBody.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

#### `typ.JobBody.equipment[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | string | No | Used to send the equipment's identifier that will be searched. You may pass this parameter if you do not want to create new entry but assign existing one. You may assign by `identifier` or `header` (Note: `identifier` - [integer] the equipment's identifier, `header` - [string] the equipment's fields concatenated by pattern `{type}:{make}:{model}:{serial_number}` with colon as separator). | Default: If not passed, it creates new one. |
| `type` | string | No | Used to send the equipment's type that will be set. |  |
| `make` | string | No | Used to send the equipment's make that will be set. |  |
| `model` | string | No | Used to send the equipment's model that will be set. | Max length: 255 |
| `sku` | string | No | Used to send the equipment's sku that will be set. | Max length: 50 |
| `serial_number` | string | No | Used to send the equipment's serial number that will be set. | Max length: 255 |
| `location` | string | No | Used to send the equipment's location that will be set. | Max length: 255 |
| `notes` | string | No | Used to send the equipment's notes that will be set. | Max length: 250 |
| `extended_warranty_provider` | string | No | Used to send the equipment's extended warranty provider that will be set. | Max length: 255 |
| `is_extended_warranty` | boolean | No | Used to send the equipment's is extended warranty flag that will be set. | Default: False |
| `extended_warranty_date` | datetime | No | Used to send the equipment's extended warranty date that will be set. |  |
| `warranty_date` | datetime | No | Used to send the equipment's warranty date that will be set. |  |
| `install_date` | datetime | No | Used to send the equipment's install date that will be set. |  |
| `customer_location` | string | No | Used to send a customer location's `id` or `header` that will be attached to the equipment (Note: `id` - [integer] the customer location's identifier, `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | Used to send the equipment's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |

##### `typ.JobBody.equipment[].custom_fields[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

#### `typ.JobBody.techs_assigned[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | Used to send the assigned tech's identifier that will be searched. If this field is set then the entry will be searched by it, otherwise the search will be performed by its full name. | Default: If not passed, it takes the value from full name search entry. |
| `first_name` | string | No | Used to send the assigned tech's first name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |
| `last_name` | string | No | Used to send the assigned tech's last name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |

#### `typ.JobBody.tasks[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `description` | string | Yes | Used to send the task's description that will be set. | Max length: 500 |
| `is_completed` | boolean | No | Used to send the task's is completed flag that will be set. | Default: False |

#### `typ.JobBody.notes[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | Yes | Used to send the note's text that will be set. |  |

#### `typ.JobBody.products[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the product's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed product. |
| `description` | string | No | Used to send the product's description that will be set. | Default: If not passed, it takes the value of passed product. |
| `multiplier` | integer | No | Used to send the product's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed product. |
| `rate` | number (float) | No | Used to send the product's rate that will be set. | Default: If not passed, it takes the value of passed product. |
| `cost` | number (float) | No | Used to send the product's cost that will be set. | Default: If not passed, it takes the value of passed product. |
| `is_show_rate_items` | boolean | No | Used to send the product's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the product (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | Yes | Used to send a product's `id` or `header` that will be attached to the product (Note: `id` - [integer] the product's identifier, `header` - [string] the product's fields concatenated by pattern `{make}`). |  |

#### `typ.JobBody.services[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the service's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed service. |
| `description` | string | No | Used to send the service's description that will be set. | Default: If not passed, it takes the value of passed service. |
| `multiplier` | integer | No | Used to send the service's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed service. |
| `rate` | number (float) | No | Used to send the service's rate that will be set. | Default: If not passed, it takes the value of passed service. |
| `cost` | number (float) | No | Used to send the service's cost that will be set. | Default: If not passed, it takes the value of passed service. |
| `is_show_rate_items` | boolean | No | Used to send the service's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the service (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | Yes | Used to send a service's `id` or `header` that will be attached to the service (Note: `id` - [integer] the service's identifier, `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |

#### `typ.JobBody.other_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the other charge's name that will be set. | Max length: 255; Default: If not passed, it takes the value of passed other charge. |
| `rate` | number (float) | No | Used to send the other charge's rate that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `is_percentage` | boolean | No | Used to send the other charge's is percentage flag that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `other_charge` | string | Yes | Used to send an other charge's `id` or `header` that will be attached to the other charge (Note: `id` - [integer] the other charge's identifier, `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |

#### `typ.JobBody.labor_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | Used to send the labor charge's drive time that will be set. Forbidden if drive times start/end passed. | Default: If drive times start/end passed, it takes the calculated difference time (in minutes), otherwise it takes the value `0`. |
| `drive_time_rate` | number (float) | No | Used to send the labor charge's drive time rate that will be set. | Default: 0 |
| `drive_time_cost` | number (float) | No | Used to send the labor charge's drive time cost that will be set. | Default: 0 |
| `drive_time_start` | string | No | Used to send the labor charge's drive time start that will be set. Required if drive time end passed. |  |
| `drive_time_end` | string | No | Used to send the labor charge's drive time end that will be set. Required if drive time start passed. Must be greater than drive time start. |  |
| `is_drive_time_billed` | boolean | No | Used to send the labor charge's is drive time billed flag that will be set. | Default: False |
| `labor_time` | integer | No | Used to send the labor charge's labor time that will be set. Forbidden if labor times start/end passed. | Default: If labor times start/end passed, it takes the calculated difference time (in minutes), otherwise it takes the value `0`. |
| `labor_time_rate` | number (float) | No | Used to send the labor charge's labor time rate that will be set. | Default: 0 |
| `labor_time_cost` | number (float) | No | Used to send the labor charge's labor time cost that will be set. | Default: 0 |
| `labor_time_start` | string | No | Used to send the labor charge's labor time start that will be set. Required if labor time end passed. |  |
| `labor_time_end` | string | No | Used to send the labor charge's labor time end that will be set. Required if labor time start passed. Must be greater than labor time start. |  |
| `labor_date` | datetime | No | Used to send the labor charge's labor date that will be set. |  |
| `is_labor_time_billed` | boolean | No | Used to send the labor charge's is labor time billed flag that will be set. | Default: False |
| `user` | string | No | Used to send a user's `id` or `header` that will be attached to the labor charge (Note: `id` - [integer] the user's identifier, `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |

#### `typ.JobBody.expenses[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | Used to send the expense's purchased from that will be set. | Max length: 255 |
| `notes` | string | No | Used to send the expense's notes that will be set. |  |
| `amount` | number (float) | No | Used to send the expense's amount that will be set. | Default: 0 |
| `is_billable` | boolean | No | Used to send the expense's is billable flag that will be set. | Default: False |
| `date` | datetime | No | Used to send the expense's date that will be set. |  |
| `user` | string | No | Used to send a user's `id` or `header` that will be attached to the expense (Note: `id` - [integer] the user's identifier, `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | Used to send a category's `id` or `header` that will be attached to the expense (Note: `id` - [integer] the category's identifier, `header` - [string] the category's fields concatenated by pattern `{category_name}`). | Default: If not passed, it takes the name of first existing category. |

### `typ.JobView`

A job's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job's identifier. |  |
| `number` | string | No | The job's number. |  |
| `check_number` | string | No | The job's check number. |  |
| `priority` | string | No | The job's priority. |  |
| `description` | string | No | The job's description. |  |
| `tech_notes` | string | No | The job's tech notes. |  |
| `completion_notes` | string | No | The job's completion notes. |  |
| `payment_status` | string | No | The job's payment status. |  |
| `taxes_fees_total` | number (float) | No | The job's taxes and fees total. |  |
| `drive_labor_total` | number (float) | No | The job's drive and labor total. |  |
| `billable_expenses_total` | number (float) | No | The job's billable expenses total. |  |
| `total` | number (float) | No | The job's total. |  |
| `payments_deposits_total` | number (float) | No | The job's payments and deposits total. |  |
| `due_total` | number (float) | No | The job's due total. |  |
| `cost_total` | number (float) | No | The job's cost total. |  |
| `duration` | integer | No | The job's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The job's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The job's time frame promised end. |  |
| `start_date` | datetime | No | The job's start date. |  |
| `end_date` | datetime | No | The job's end date. |  |
| `created_at` | datetime | No | The job's created date. |  |
| `updated_at` | datetime | No | The job's updated date. |  |
| `closed_at` | datetime | No | The job's closed date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the job (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the job (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the job (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the job (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the job (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The job's contact first name. |  |
| `contact_last_name` | string | No | The job's contact last name. |  |
| `street_1` | string | No | The job's location street 1. |  |
| `street_2` | string | No | The job's location street 2. |  |
| `city` | string | No | The job's location city. |  |
| `state_prov` | string | No | The job's location state prov. |  |
| `postal_code` | string | No | The job's location postal code. |  |
| `location_name` | string | No | The job's location name. |  |
| `is_gated` | boolean | No | The job's location is gated flag. |  |
| `gate_instructions` | string | No | The job's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the job (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the job (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the job (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `customer_payment_terms` | string | No | The `header` of attached customer payment term to the job (Note: `header` - [string] the customer payment term's fields concatenated by pattern `{name}`). |  |
| `project` | string | No | The `header` of attached project to the job (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the job (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The job's po number. |  |
| `contract` | string | No | The `header` of attached contract to the job (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The job's note to customer. |  |
| `called_in_by` | string | No | The job's called in by. |  |
| `is_requires_follow_up` | boolean | No | The job's is requires follow up flag. |  |
| `agents` | array[object] | No | The job's agents list. |  |
| `custom_fields` | array[object] | No | The job's custom fields list. |  |
| `pictures` | array[object] | No | The job's pictures list. |  |
| `documents` | array[object] | No | The job's documents list. |  |
| `equipment` | array[object] | No | The job's equipments list. |  |
| `techs_assigned` | array[object] | No | The job's techs assigned list. |  |
| `tasks` | array[object] | No | The job's tasks list. |  |
| `notes` | array[object] | No | The job's notes list. |  |
| `products` | array[object] | No | The job's products list. |  |
| `services` | array[object] | No | The job's services list. |  |
| `other_charges` | array[object] | No | The job's other charges list. |  |
| `labor_charges` | array[object] | No | The job's labor charges list. |  |
| `expenses` | array[object] | No | The job's expenses list. |  |
| `payments` | array[object] | No | The job's payments list. |  |
| `invoices` | array[object] | No | The job's invoices list. |  |
| `signatures` | array[object] | No | The job's signatures list. |  |
| `printable_work_order` | array[object] | No | The job's printable work order list. |  |
| `visits` | array[object] | No | The job's visits list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

#### `typ.JobView.agents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

#### `typ.JobView.custom_fields[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

#### `typ.JobView.pictures[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

#### `typ.JobView.documents[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

#### `typ.JobView.equipment[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

##### `typ.JobView.equipment[].custom_fields[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

#### `typ.JobView.techs_assigned[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

#### `typ.JobView.tasks[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

#### `typ.JobView.notes[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

#### `typ.JobView.products[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.JobView.services[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.JobView.other_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.JobView.labor_charges[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | The labor charge's drive time. |  |
| `drive_time_rate` | number (float) | No | The labor charge's drive time rate. |  |
| `drive_time_cost` | number (float) | No | The labor charge's drive time cost. |  |
| `drive_time_start` | string | No | The labor charge's drive time start. |  |
| `drive_time_end` | string | No | The labor charge's drive time end. |  |
| `is_drive_time_billed` | boolean | No | The labor charge's is drive time billed flag. |  |
| `labor_time` | integer | No | The labor charge's labor time. |  |
| `labor_time_rate` | number (float) | No | The labor charge's labor time rate. |  |
| `labor_time_cost` | number (float) | No | The labor charge's labor time cost. |  |
| `labor_time_start` | string | No | The labor charge's labor time start. |  |
| `labor_time_end` | string | No | The labor charge's labor time end. |  |
| `labor_date` | datetime | No | The labor charge's labor date. |  |
| `is_labor_time_billed` | boolean | No | The labor charge's is labor time billed flag. |  |
| `total` | number (float) | No | The labor charge's total. |  |
| `created_at` | datetime | No | The labor charge's created date. |  |
| `updated_at` | datetime | No | The labor charge's updated date. |  |
| `is_status_generated` | boolean | No | The labor charge's is status generated flag. |  |
| `user` | string | No | The `header` of attached user to the labor charge (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `visit_id` | integer | No | The `id` of attached visit to the labor charge (Note: `id` - [integer] the visit's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the labor charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the labor charge (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.JobView.expenses[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | The expense's purchased from. |  |
| `notes` | string | No | The expense's notes. |  |
| `amount` | number (float) | No | The expense's amount. |  |
| `is_billable` | boolean | No | The expense's is billable flag. |  |
| `date` | datetime | No | The expense's date. |  |
| `created_at` | datetime | No | The expense's created date. |  |
| `updated_at` | datetime | No | The expense's updated date. |  |
| `user` | string | No | The `header` of attached user to the expense (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | The `header` of attached category to the expense (Note: `header` - [string] the category's fields concatenated by pattern `{category_name}`). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the expense (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the expense (Note: `id` - [integer] the qbd class'es identifier). |  |

#### `typ.JobView.payments[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

#### `typ.JobView.invoices[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |

#### `typ.JobView.signatures[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

#### `typ.JobView.printable_work_order[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

#### `typ.JobView.visits[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes_for_techs` | string | No | The visit's notes for techs. |  |
| `time_frame_promised_start` | string | No | The visit's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The visit's time frame promised end. |  |
| `duration` | integer | No | The visit's duration (in seconds). |  |
| `is_text_notified` | boolean | No | The visit's is text notified flag. |  |
| `is_voice_notified` | boolean | No | The visit's is voice notified flag. |  |
| `start_date` | datetime | No | The visit's start date. |  |
| `techs_assigned` | array[object] | No | The visit's techs assigned list. |  |

##### `typ.JobView.visits[].techs_assigned[]` (nested array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |
| `status` | string | No | The assigned tech's status. |  |

### `typ.JobCategory`

A job category's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job category's identifier. |  |
| `name` | string | No | The job category's name. |  |

### `typ.JobCategoryView`

A job category's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job category's identifier. |  |
| `name` | string | No | The job category's name. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

### `typ.JobStatus`

A job statuse's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job statuse's identifier. |  |
| `code` | string | No | The job statuse's code. |  |
| `name` | string | No | The job statuse's name. |  |
| `is_custom` | string | No | The job statuse's is custom flag. |  |
| `category` | string | No | The `header` of attached category to the status (Note: `header` - [string] the category's fields concatenated by pattern `{code}`). |  |

### `typ.JobStatusView`

A job statuse's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job statuse's identifier. |  |
| `code` | string | No | The job statuse's code. |  |
| `name` | string | No | The job statuse's name. |  |
| `is_custom` | string | No | The job statuse's is custom flag. |  |
| `category` | string | No | The `header` of attached category to the status (Note: `header` - [string] the category's fields concatenated by pattern `{code}`). |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

### `typ.JobDocument`

A job document's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

### `typ.JobExpense`

A job expense's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | The expense's purchased from. |  |
| `notes` | string | No | The expense's notes. |  |
| `amount` | number (float) | No | The expense's amount. |  |
| `is_billable` | boolean | No | The expense's is billable flag. |  |
| `date` | datetime | No | The expense's date. |  |
| `created_at` | datetime | No | The expense's created date. |  |
| `updated_at` | datetime | No | The expense's updated date. |  |
| `user` | string | No | The `header` of attached user to the expense (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | The `header` of attached category to the expense (Note: `header` - [string] the category's fields concatenated by pattern `{category_name}`). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the expense (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the expense (Note: `id` - [integer] the qbd class'es identifier). |  |

### `typ.JobExpenseBody`

A job expense's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | Used to send the expense's purchased from that will be set. | Max length: 255 |
| `notes` | string | No | Used to send the expense's notes that will be set. |  |
| `amount` | number (float) | No | Used to send the expense's amount that will be set. | Default: 0 |
| `is_billable` | boolean | No | Used to send the expense's is billable flag that will be set. | Default: False |
| `date` | datetime | No | Used to send the expense's date that will be set. |  |
| `user` | string | No | Used to send a user's `id` or `header` that will be attached to the expense (Note: `id` - [integer] the user's identifier, `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | Used to send a category's `id` or `header` that will be attached to the expense (Note: `id` - [integer] the category's identifier, `header` - [string] the category's fields concatenated by pattern `{category_name}`). | Default: If not passed, it takes the name of first existing category. |

### `typ.JobLaborCharge`

A job labor charge's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | The labor charge's drive time. |  |
| `drive_time_rate` | number (float) | No | The labor charge's drive time rate. |  |
| `drive_time_cost` | number (float) | No | The labor charge's drive time cost. |  |
| `drive_time_start` | string | No | The labor charge's drive time start. |  |
| `drive_time_end` | string | No | The labor charge's drive time end. |  |
| `is_drive_time_billed` | boolean | No | The labor charge's is drive time billed flag. |  |
| `labor_time` | integer | No | The labor charge's labor time. |  |
| `labor_time_rate` | number (float) | No | The labor charge's labor time rate. |  |
| `labor_time_cost` | number (float) | No | The labor charge's labor time cost. |  |
| `labor_time_start` | string | No | The labor charge's labor time start. |  |
| `labor_time_end` | string | No | The labor charge's labor time end. |  |
| `labor_date` | datetime | No | The labor charge's labor date. |  |
| `is_labor_time_billed` | boolean | No | The labor charge's is labor time billed flag. |  |
| `total` | number (float) | No | The labor charge's total. |  |
| `created_at` | datetime | No | The labor charge's created date. |  |
| `updated_at` | datetime | No | The labor charge's updated date. |  |
| `is_status_generated` | boolean | No | The labor charge's is status generated flag. |  |
| `user` | string | No | The `header` of attached user to the labor charge (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `visit_id` | integer | No | The `id` of attached visit to the labor charge (Note: `id` - [integer] the visit's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the labor charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the labor charge (Note: `id` - [integer] the qbd class'es identifier). |  |

### `typ.JobLaborChargeBody`

A job labor charge's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | Used to send the labor charge's drive time that will be set. Forbidden if drive times start/end passed. | Default: If drive times start/end passed, it takes the calculated difference time (in minutes), otherwise it takes the value `0`. |
| `drive_time_rate` | number (float) | No | Used to send the labor charge's drive time rate that will be set. | Default: 0 |
| `drive_time_cost` | number (float) | No | Used to send the labor charge's drive time cost that will be set. | Default: 0 |
| `drive_time_start` | string | No | Used to send the labor charge's drive time start that will be set. Required if drive time end passed. |  |
| `drive_time_end` | string | No | Used to send the labor charge's drive time end that will be set. Required if drive time start passed. Must be greater than drive time start. |  |
| `is_drive_time_billed` | boolean | No | Used to send the labor charge's is drive time billed flag that will be set. | Default: False |
| `labor_time` | integer | No | Used to send the labor charge's labor time that will be set. Forbidden if labor times start/end passed. | Default: If labor times start/end passed, it takes the calculated difference time (in minutes), otherwise it takes the value `0`. |
| `labor_time_rate` | number (float) | No | Used to send the labor charge's labor time rate that will be set. | Default: 0 |
| `labor_time_cost` | number (float) | No | Used to send the labor charge's labor time cost that will be set. | Default: 0 |
| `labor_time_start` | string | No | Used to send the labor charge's labor time start that will be set. Required if labor time end passed. |  |
| `labor_time_end` | string | No | Used to send the labor charge's labor time end that will be set. Required if labor time start passed. Must be greater than labor time start. |  |
| `labor_date` | datetime | No | Used to send the labor charge's labor date that will be set. |  |
| `is_labor_time_billed` | boolean | No | Used to send the labor charge's is labor time billed flag that will be set. | Default: False |
| `user` | string | No | Used to send a user's `id` or `header` that will be attached to the labor charge (Note: `id` - [integer] the user's identifier, `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |

### `typ.JobNote`

A job note's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

### `typ.JobNoteBody`

A job note's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | Yes | Used to send the note's text that will be set. |  |

### `typ.JobOtherCharge`

A job other charge's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

### `typ.JobOtherChargeBody`

A job other charge's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the other charge's name that will be set. | Max length: 255; Default: If not passed, it takes the value of passed other charge. |
| `rate` | number (float) | No | Used to send the other charge's rate that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `is_percentage` | boolean | No | Used to send the other charge's is percentage flag that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `other_charge` | string | Yes | Used to send an other charge's `id` or `header` that will be attached to the other charge (Note: `id` - [integer] the other charge's identifier, `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |

### `typ.JobProduct`

A job product's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

### `typ.JobProductBody`

A job product's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the product's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed product. |
| `description` | string | No | Used to send the product's description that will be set. | Default: If not passed, it takes the value of passed product. |
| `multiplier` | integer | No | Used to send the product's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed product. |
| `rate` | number (float) | No | Used to send the product's rate that will be set. | Default: If not passed, it takes the value of passed product. |
| `cost` | number (float) | No | Used to send the product's cost that will be set. | Default: If not passed, it takes the value of passed product. |
| `is_show_rate_items` | boolean | No | Used to send the product's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the product (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | Yes | Used to send a product's `id` or `header` that will be attached to the product (Note: `id` - [integer] the product's identifier, `header` - [string] the product's fields concatenated by pattern `{make}`). |  |

### `typ.JobService`

A job service's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

### `typ.JobServiceBody`

A job service's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the service's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed service. |
| `description` | string | No | Used to send the service's description that will be set. | Default: If not passed, it takes the value of passed service. |
| `multiplier` | integer | No | Used to send the service's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed service. |
| `rate` | number (float) | No | Used to send the service's rate that will be set. | Default: If not passed, it takes the value of passed service. |
| `cost` | number (float) | No | Used to send the service's cost that will be set. | Default: If not passed, it takes the value of passed service. |
| `is_show_rate_items` | boolean | No | Used to send the service's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the service (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | Yes | Used to send a service's `id` or `header` that will be attached to the service (Note: `id` - [integer] the service's identifier, `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |

### `typ.JobSignature`

A job signature's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

### `typ.JobTag`

A job tag's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | No | The tag's unique tag. |  |
| `created_at` | datetime | No | The tag's created date. |  |
| `updated_at` | datetime | No | The tag's updated date. |  |

### `typ.JobTagBody`

A job tag's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | Yes | Used to send the tag's unique tag that will be set. | Max length: 254 |

### `typ.JobTask`

A job task's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

### `typ.JobTaskBody`

A job task's body schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `description` | string | Yes | Used to send the task's description that will be set. | Max length: 500 |
| `is_completed` | boolean | No | Used to send the task's is completed flag that will be set. | Default: False |

### `typ.JobVisit`

A job visit's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes_for_techs` | string | No | The visit's notes for techs. |  |
| `time_frame_promised_start` | string | No | The visit's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The visit's time frame promised end. |  |
| `duration` | integer | No | The visit's duration (in seconds). |  |
| `is_text_notified` | boolean | No | The visit's is text notified flag. |  |
| `is_voice_notified` | boolean | No | The visit's is voice notified flag. |  |
| `start_date` | datetime | No | The visit's start date. |  |
| `techs_assigned` | array[object] | No | The visit's techs assigned list. |  |

#### `typ.JobVisit.techs_assigned[]` (array item)

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |
| `status` | string | No | The assigned tech's status. |  |

### `typ.MeView`

An authenticated user's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The authenticated user's identifier. |  |
| `first_name` | string | No | The authenticated user's first name. |  |
| `last_name` | string | No | The authenticated user's last name. |  |
| `email` | string | No | The authenticated user's email. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

### `typ.Payment`

A payment's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

### `typ.PaymentType`

A payment type's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The type's identifier. |  |
| `code` | string | No | The type's code. |  |
| `short_name` | string | No | The type's short name. |  |
| `type` | string | No | The type's type. |  |
| `is_custom` | boolean | No | The type's is custom flag. |  |

### `typ.PaymentTypeView`

A payment type's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The type's identifier. |  |
| `code` | string | No | The type's code. |  |
| `short_name` | string | No | The type's short name. |  |
| `type` | string | No | The type's type. |  |
| `is_custom` | boolean | No | The type's is custom flag. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

### `typ.PrintableWorkOrder`

A printable work order's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

### `typ.Source`

A source's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The source's identifier. |  |
| `short_name` | string | No | The source's short name. |  |
| `long_name` | string | No | The source's long name. |  |

### `typ.SourceView`

A source's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The source's identifier. |  |
| `short_name` | string | No | The source's short name. |  |
| `long_name` | string | No | The source's long name. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

### `typ.Tech`

A tech's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The tech's identifier. |  |
| `first_name` | string | No | The tech's first name. |  |
| `last_name` | string | No | The tech's last name. |  |
| `nickname_on_workorder` | string | No | The tech's nickname on workorder. |  |
| `nickname_on_dispatch` | string | No | The tech's nickname on dispatch. |  |
| `color_code` | string | No | The tech's color code. |  |
| `email` | string | No | The tech's email. |  |
| `phone_1` | string | No | The tech's phone 1. |  |
| `phone_2` | string | No | The tech's phone 2. |  |
| `gender` | string | No | The tech's gender. |  |
| `department` | string | No | The tech's department. |  |
| `title` | string | No | The tech's title. |  |
| `bio` | string | No | The tech's bio. |  |
| `is_phone_1_mobile` | boolean | No | The tech's is phone 1 mobile flag. |  |
| `is_phone_1_visible_to_client` | boolean | No | The tech's is phone 1 visible to client flag. |  |
| `is_phone_2_mobile` | boolean | No | The tech's is phone 2 mobile flag. |  |
| `is_phone_2_visible_to_client` | boolean | No | The tech's is phone 2 visible to client flag. |  |
| `is_sales_rep` | boolean | No | The tech's is sales rep flag. |  |
| `is_field_worker` | boolean | No | The tech's is field worker flag. |  |
| `created_at` | datetime | No | The tech's created date. |  |
| `updated_at` | datetime | No | The tech's updated date. |  |

### `typ.TechView`

A tech's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The tech's identifier. |  |
| `first_name` | string | No | The tech's first name. |  |
| `last_name` | string | No | The tech's last name. |  |
| `nickname_on_workorder` | string | No | The tech's nickname on workorder. |  |
| `nickname_on_dispatch` | string | No | The tech's nickname on dispatch. |  |
| `color_code` | string | No | The tech's color code. |  |
| `email` | string | No | The tech's email. |  |
| `phone_1` | string | No | The tech's phone 1. |  |
| `phone_2` | string | No | The tech's phone 2. |  |
| `gender` | string | No | The tech's gender. |  |
| `department` | string | No | The tech's department. |  |
| `title` | string | No | The tech's title. |  |
| `bio` | string | No | The tech's bio. |  |
| `is_phone_1_mobile` | boolean | No | The tech's is phone 1 mobile flag. |  |
| `is_phone_1_visible_to_client` | boolean | No | The tech's is phone 1 visible to client flag. |  |
| `is_phone_2_mobile` | boolean | No | The tech's is phone 2 mobile flag. |  |
| `is_phone_2_visible_to_client` | boolean | No | The tech's is phone 2 visible to client flag. |  |
| `is_sales_rep` | boolean | No | The tech's is sales rep flag. |  |
| `is_field_worker` | boolean | No | The tech's is field worker flag. |  |
| `created_at` | datetime | No | The tech's created date. |  |
| `updated_at` | datetime | No | The tech's updated date. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

### `typ.Picture`

A picture's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

### `typ.Document`

A document's schema.

**Base type:** `object`

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

---

## Resource Type Templates

These are RAML resource type templates that define common patterns applied to resources.

### `res.read-only`

**GET method template:**

- Description: List all <<resourcePathName \| !uppercamelcase>> matching query criteria, if provided, otherwise list all <<resourcePathName \| !uppercamelcase>>.
- Response `200`: ### 200 OK (Success) Standard response for successful HTTP requests. (body: `object`)
- Response `400`: ### 400 Bad Request (Client Error) The server cannot or will not process the request due to an appar... (body: `object`)
- Response `405`: ### 405 Method Not Allowed (Client Error) A request method is not supported for the requested resour... (body: `object`)
- Response `415`: ### 415 Unsupported Media Type (Client Error) The request entity has a media type which the server o... (body: `object`)
- Response `429`: ### 429 Too Many Requests (Client Error) The user has sent too many requests in a given amount of ti... (body: `object`)
- Response `500`: ### 500 Internal Server Error (Server Error) A generic error message, given when an unexpected condi... (body: `object`)

**Sub-resource pattern:** `/{<<resourcePathName | !singularize>>-id}`

- Get a <<resourcePathName \| !uppercamelcase \| !singularize>> by identifier.
- URI Parameters:
  - `<<resourcePathName | !singularize>>-id`: integer (required: True) -- Used to send an identifier of the <<resourcePathName \| !uppercamelcase \| !singularize>> to be used.

### `res.create-read-only`

**POST method template:**

- Description: Create a new <<resourcePathName \| !uppercamelcase \| !singularize>>.
- Response `201`: ### 201 Created (Success) The request has been fulfilled, resulting in the creation of a new resourc... (body: `object`)
- Response `400`: ### 400 Bad Request (Client Error) The server cannot or will not process the request due to an appar... (body: `object`)
- Response `405`: ### 405 Method Not Allowed (Client Error) A request method is not supported for the requested resour... (body: `object`)
- Response `415`: ### 415 Unsupported Media Type (Client Error) The request entity has a media type which the server o... (body: `object`)
- Response `422`: ### 422 Unprocessable Entity (Client Error) The request was well-formed but was unable to be followe... (body: `object`)
- Response `429`: ### 429 Too Many Requests (Client Error) The user has sent too many requests in a given amount of ti... (body: `object`)
- Response `500`: ### 500 Internal Server Error (Server Error) A generic error message, given when an unexpected condi... (body: `object`)

---

## Resources

Complete endpoint reference for all API resources.

### `/me`

#### GET `/me`

Authorized user information.

**Traits applied:** `tra.me-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `first_name`, `last_name`, `email` | `id,email` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `MeView`

An authenticated user's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The authenticated user's identifier. |  |
| `first_name` | string | No | The authenticated user's first name. |  |
| `last_name` | string | No | The authenticated user's last name. |  |
| `email` | string | No | The authenticated user's email. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**Example:**
```json
{
  "id": 1472289,
  "first_name": "Justin",
  "last_name": "Wormell",
  "email": "justin@servicefusion.com",
  "_expandable": []
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/calendar-tasks`

**Resource type:** `res.read-only`

#### GET `/calendar-tasks`

List all CalendarTasks matching query criteria, if provided,
otherwise list all CalendarTasks.

**Traits applied:** `tra.calendarTask-fieldable`, `tra.calendarTask-sortable`, `tra.calendarTask-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `type`, `description`, `start_time`, `end_time`, `start_date`, `end_date`, `created_at`, `updated_at`, `is_public`, `is_completed`, `repeat_id`, `users_id`, `customers_id`, `jobs_id`, `estimates_id` | `id,description,is_completed` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `repeat` | `repeat` |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `type`, `description`, `start_time`, `end_time`, `start_date`, `end_date`, `created_at`, `updated_at`, `is_public`, `is_completed`, `repeat_id` | `type,-end_time` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A calendar-task's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The calendar task's identifier. |  |
| `type` | string | No | The calendar task's type. |  |
| `description` | string | No | The calendar task's description. |  |
| `start_time` | string | No | The calendar task's start time. |  |
| `end_time` | string | No | The calendar task's end time. |  |
| `start_date` | datetime | No | The calendar task's start date. |  |
| `end_date` | datetime | No | The calendar task's end date. |  |
| `created_at` | datetime | No | The calendar task's created date. |  |
| `updated_at` | datetime | No | The calendar task's updated date. |  |
| `is_public` | boolean | No | The calendar task's is public flag. |  |
| `is_completed` | boolean | No | The calendar task's is completed flag. |  |
| `repeat_id` | integer | No | The calendar task's repeat id. |  |
| `users_id` | array[integer] | Yes | The calendar task's users list of identifiers. |  |
| `customers_id` | array[integer] | Yes | The calendar task's customers list of identifiers. |  |
| `jobs_id` | array[integer] | Yes | The calendar task's jobs list of identifiers. |  |
| `estimates_id` | array[integer] | Yes | The calendar task's estimates list of identifiers. |  |
| `repeat` | object | No | The calendar task's repeat. | Example: `{   "id": 92,   "repeat_type": "Daily",   "repeat_frequency": 2,   "repeat_weekly_days": [],   "repeat_monthly_type": null,   "stop_repeat_type": "On Occurrence",   "stop_repeat_on_occurrence": 10,   "stop_repeat_on_date": null,   "start_date": "2021-05-27T00:00:00+00:00",   "end_date": "2021-06-14T00:00:00+00:00" }` |

**`items[].repeat` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The repeat's identifier. |  |
| `repeat_type` | string | No | The repeat's type. |  |
| `repeat_frequency` | integer | No | The repeat's frequency. |  |
| `repeat_weekly_days` | array[string] | Yes | The repeat's weekly days list. |  |
| `repeat_monthly_type` | string | No | The repeat's monthly type. |  |
| `stop_repeat_type` | string | No | The repeat's stop type. |  |
| `stop_repeat_on_occurrence` | integer | No | The repeat's stop on occurrence. |  |
| `stop_repeat_on_date` | datetime | No | The repeat's stop on date. |  |
| `start_date` | datetime | No | The repeat's start date. |  |
| `end_date` | datetime | No | The repeat's end date. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 16546,
      "type": "Call",
      "description": "Zapier task note",
      "start_time": "10:00",
      "end_time": "22:00",
      "start_date": "2021-05-01",
      "end_date": null,
      "created_at": "2021-06-22T11:02:32+00:00",
      "updated_at": "2021-06-22T11:02:32+00:00",
      "is_public": false,
      "is_completed": false,
      "repeat_id": 99,
      "users_id": [
        980190972,
        980190979
      ],
      "customers_id": [
        9303,
        842180
      ],
      "jobs_id": [
        1152721,
        1152722
      ],
      "estimates_id": [
        1152212,
        1152932
      ],
      "repeat": {
        "id": 92,
        "repeat_type": "Daily",
        "repeat_frequency": 2,
        "repeat_weekly_days": [],
        "repeat_monthly_type": null,
        "stop_repeat_type": "On Occurrence",
        "stop_repeat_on_occurrence": 10,
        "stop_repeat_on_date": null,
        "start_date": "2021-05-27T00:00:00+00:00",
        "end_date": "2021-06-14T00:00:00+00:00"
      }
    }
  ],
  "_expandable": [
    "repeat"
  ],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/calendar-tasks/{calendar-task-id}`

Get a CalendarTask by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `calendar-task-id` | integer | Yes | Used to send an identifier of the CalendarTask to be used. |

##### GET `/calendar-tasks/{calendar-task-id}`

Get a CalendarTask by identifier.

**Traits applied:** `tra.calendarTask-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `type`, `description`, `start_time`, `end_time`, `start_date`, `end_date`, `created_at`, `updated_at`, `is_public`, `is_completed`, `repeat_id`, `users_id`, `customers_id`, `jobs_id`, `estimates_id` | `id,description,is_completed` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `repeat` | `repeat` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `CalendarTaskView`

A calendar task's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The calendar task's identifier. |  |
| `type` | string | No | The calendar task's type. |  |
| `description` | string | No | The calendar task's description. |  |
| `start_time` | string | No | The calendar task's start time. |  |
| `end_time` | string | No | The calendar task's end time. |  |
| `start_date` | datetime | No | The calendar task's start date. |  |
| `end_date` | datetime | No | The calendar task's end date. |  |
| `created_at` | datetime | No | The calendar task's created date. |  |
| `updated_at` | datetime | No | The calendar task's updated date. |  |
| `is_public` | boolean | No | The calendar task's is public flag. |  |
| `is_completed` | boolean | No | The calendar task's is completed flag. |  |
| `repeat_id` | integer | No | The calendar task's repeat id. |  |
| `users_id` | array[integer] | Yes | The calendar task's users list of identifiers. |  |
| `customers_id` | array[integer] | Yes | The calendar task's customers list of identifiers. |  |
| `jobs_id` | array[integer] | Yes | The calendar task's jobs list of identifiers. |  |
| `estimates_id` | array[integer] | Yes | The calendar task's estimates list of identifiers. |  |
| `repeat` | object | No | The calendar task's repeat. | Example: `{   "id": 92,   "repeat_type": "Daily",   "repeat_frequency": 2,   "repeat_weekly_days": [],   "repeat_monthly_type": null,   "stop_repeat_type": "On Occurrence",   "stop_repeat_on_occurrence": 10,   "stop_repeat_on_date": null,   "start_date": "2021-05-27T00:00:00+00:00",   "end_date": "2021-06-14T00:00:00+00:00" }` |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**`repeat` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The repeat's identifier. |  |
| `repeat_type` | string | No | The repeat's type. |  |
| `repeat_frequency` | integer | No | The repeat's frequency. |  |
| `repeat_weekly_days` | array[string] | Yes | The repeat's weekly days list. |  |
| `repeat_monthly_type` | string | No | The repeat's monthly type. |  |
| `stop_repeat_type` | string | No | The repeat's stop type. |  |
| `stop_repeat_on_occurrence` | integer | No | The repeat's stop on occurrence. |  |
| `stop_repeat_on_date` | datetime | No | The repeat's stop on date. |  |
| `start_date` | datetime | No | The repeat's start date. |  |
| `end_date` | datetime | No | The repeat's end date. |  |

**Example:**
```json
{
  "id": 16546,
  "type": "Call",
  "description": "Zapier task note",
  "start_time": "10:00",
  "end_time": "22:00",
  "start_date": "2021-05-01",
  "end_date": null,
  "created_at": "2021-06-22T11:02:32+00:00",
  "updated_at": "2021-06-22T11:02:32+00:00",
  "is_public": false,
  "is_completed": false,
  "repeat_id": 99,
  "users_id": [
    980190972,
    980190979
  ],
  "customers_id": [
    9303,
    842180
  ],
  "jobs_id": [
    1152721,
    1152722
  ],
  "estimates_id": [
    1152212,
    1152932
  ],
  "repeat": {
    "id": 92,
    "repeat_type": "Daily",
    "repeat_frequency": 2,
    "repeat_weekly_days": [],
    "repeat_monthly_type": null,
    "stop_repeat_type": "On Occurrence",
    "stop_repeat_on_occurrence": 10,
    "stop_repeat_on_date": null,
    "start_date": "2021-05-27T00:00:00+00:00",
    "end_date": "2021-06-14T00:00:00+00:00"
  },
  "_expandable": [
    "repeat"
  ]
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/customers`

**Resource type:** `res.create-read-only`

#### POST `/customers`

Create a new Customer.

**Traits applied:** `tra.customer-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Content-Type` | string | Yes | Used to send a format of data of the request. Enum: `application/json`, `application/x-www-form-urlencoded` |  |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `customer_name`, `fully_qualified_name`, `account_number`, `account_balance`, `private_notes`, `public_notes`, `payment_terms`, `discount`, `discount_type`, `credit_rating`, `labor_charge_type`, `labor_charge_default_rate`, `qbo_sync_token`, `qbo_currency`, `qbo_id`, `qbd_id`, `created_at`, `updated_at`, `last_serviced_date`, `is_bill_for_drive_time`, `is_vip`, `is_taxable`, `parent_customer`, `referral_source`, `agent`, `assigned_contract`, `payment_type`, `tax_item_name`, `industry` | `id,customer_name,discount` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `contacts`, `contacts.phones`, `contacts.emails`, `locations`, `custom_fields` | `contacts.phones,locations` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Request Body:**

Content-Type: `application/json` (Schema: `CustomerBody`)

A customer's body schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `customer_name` | string | Yes | Used to send the customer's name that will be set. | Max length: 255 |
| `parent_customer` | string | No | Used to send a parent customer's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the parent customer's identifier, `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `account_number` | string | No | Used to send the customer's account number that will be set. | Max length: 255; Default: If not passed, it takes generated new one. |
| `private_notes` | string | No | Used to send the customer's private notes that will be set. |  |
| `public_notes` | string | No | Used to send the customer's public notes that will be set. |  |
| `credit_rating` | string | No | Used to send the customer's credit rating that will be set. | Enum: `A+`, `A`, `B+`, `B`, `C+`, `C`, `U`; Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `labor_charge_type` | string | No | Used to send the customer's labor charge type that will be set. | Enum: `flat`, `hourly`; Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `labor_charge_default_rate` | number (float) | No | Used to send the customer's labor charge default rate that will be set. | Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `last_serviced_date` | datetime | No | Used to send the customer's last serviced date that will be set. |  |
| `is_bill_for_drive_time` | boolean | No | Used to send the customer's is bill for drive time flag that will be set. | Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `is_vip` | boolean | No | Used to send the customer's is vip flag that will be set. | Default: False |
| `referral_source` | string | No | Used to send a referral source's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the referral source's identifier, `header` - [string] the referral source's fields concatenated by pattern `{short_name}`). |  |
| `agent` | string | No | Used to send an agent's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the agent's identifier, `header` - [string] the agent's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `discount` | number (float) | No | Used to send the customer's discount that will be set. | Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `discount_type` | string | No | Used to send the customer's discount type that will be set. | Enum: `$`, `%`; Default: If not passed, it takes the value from parent customer (configurable into the company preferences). |
| `payment_type` | string | No | Used to send a payment type's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the payment type's identifier, `header` - [string] the payment type's fields concatenated by pattern `{name}`). | Default: If not passed, it takes the value from the company preferences or from parent customer (configurable into the company preferences). |
| `payment_terms` | string | No | Used to send the customer's payment terms that will be set. | Default: If not passed, it takes the value from the company preferences or from parent customer (configurable into the company preferences). |
| `assigned_contract` | string | No | Used to send an assigned contract's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the assigned contract's identifier, `header` - [string] the assigned contract's fields concatenated by pattern `{contract_title}`). |  |
| `industry` | string | No | Used to send an industry's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the industry's identifier, `header` - [string] the industry's fields concatenated by pattern `{industry}`). |  |
| `is_taxable` | boolean | No | Used to send the customer's is taxable flag that will be set. | Default: If not passed, it takes the value `true` (configurable into the company preferences). |
| `tax_item_name` | string | No | Used to send a tax item's `id` or `header` that will be attached to the customer (Note: `id` - [integer] the tax item's identifier, `header` - [string] the tax item's fields concatenated by pattern `{short_name}`). | Default: If not passed, it takes the value from the company preferences (configurable into the company preferences). |
| `qbo_sync_token` | integer | No | Used to send the customer's qbo sync token that will be set. |  |
| `qbo_currency` | string | No | Used to send the customer's qbo currency that will be set. | Enum: `USD`, `CAD`, `JMD`, `THB`; Default: If not passed, it takes the value from the company if it was configured, otherwise it takes the value `USD`. |
| `contacts` | array[object] | No | Used to send the customer's contacts list that will be set. | Default: If not passed, it creates the new one. |
| `locations` | array[object] | No | Used to send the customer's locations list that will be set. | Default: array |
| `custom_fields` | array[object] | No | Used to send the customer's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |

**`contacts[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | Used to send the contact's prefix that will be set. | Enum: `Mr.`, `Mrs.`, `Ms.`, `Dr.`, `Atty.`, `Prof.`, `Hon.`, `Gov.`, `Ofc.`, `Rep.`, `Sen.`, `Amb.`, `Sec.`, `Pvt.`, `Cpl.`, `Sgt.`, `Adm.`, `Gen.`, `Maj.`, `Capt.`, `Cmdr.`, `Lt.`, `Lt Col.`, `Col.` |
| `fname` | string | Yes | Used to send the contact's first name that will be set. | Max length: 255 |
| `lname` | string | Yes | Used to send the contact's last name that will be set. | Max length: 255 |
| `suffix` | string | No | Used to send the contact's suffix that will be set. | Max length: 255 |
| `contact_type` | string | No | Used to send the contact's type that will be set. | Max length: 255 |
| `dob` | string | No | Used to send the contact's dob that will be set. | Max length: 250 |
| `anniversary` | string | No | Used to send the contact's anniversary that will be set. | Max length: 250 |
| `job_title` | string | No | Used to send the contact's job title that will be set. | Max length: 200 |
| `department` | string | No | Used to send the contact's department that will be set. | Max length: 200 |
| `is_primary` | boolean | No | Used to send the contact's is primary flag that will be set. When it is passed as `true`, then the customer's existing primary contact (if any) will become secondary, and this one will become the primary one. | Default: If not passed and the customer does not have primary contact, it takes the value `true`, else if the customer already have primary contact, it takes the value `false`. |
| `phones` | array[object] | No | Used to send the contact's phones list that will be set. | Default: array |
| `emails` | array[object] | No | Used to send the contact's emails list that will be set. | Default: array |

**`contacts[].phones[]` (nested array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | Yes | Used to send the phone's number that will be set. | Pattern: `^[0-9]{3}\-[0-9]{3}\-[0-9]{4}$` |
| `ext` | integer | No | Used to send the phone's extension that will be set. |  |
| `type` | string | No | Used to send the phone's type that will be set. | Enum: `Mobile`, `Home`, `Work`, `Other` |

**`contacts[].emails[]` (nested array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | Yes | Used to send the email's address that will be set. | Max length: 255; Pattern: `^[^@]*<[a-zA-Z0-9!#$%&\'*+\\/=?^_`{\|}~-]+(?:\.[a-zA-Z0-9!#$%&\'*+\\/=?^_`{\|}~-]+)*@(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?>$` |
| `class` | string | No | Used to send the email's class that will be set. | Enum: `Personal`, `Business`, `Other` |
| `types_accepted` | string | No | Used to send the email's types accepted that will be set. Accepted value is comma-separated string. | Enum: `CONF`, `STATUS`, `PMT`, `INV` |

**`locations[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | Yes | Used to send the location's street 1 that will be set. | Max length: 255 |
| `street_2` | string | No | Used to send the location's street 2 that will be set. | Max length: 255 |
| `city` | string | No | Used to send the location's city that will be set. | Max length: 255 |
| `state_prov` | string | No | Used to send the location's state that will be set. | Max length: 255 |
| `postal_code` | string | No | Used to send the location's postal code that will be set. | Max length: 255 |
| `country` | string | No | Used to send the location's country that will be set. | Max length: 255 |
| `nickname` | string | No | Used to send the location's nickname that will be set. | Max length: 255 |
| `gate_instructions` | string | No | Used to send the location's gate instructions that will be set. |  |
| `latitude` | number (float) | No | Used to send the location's latitude that will be set. | Default: 0 |
| `longitude` | number (float) | No | Used to send the location's longitude that will be set. | Default: 0 |
| `location_type` | string | No | Used to send the location's type that will be set. | Max length: 200 |
| `is_primary` | boolean | No | Used to send the location's is primary flag that will be set. When it is passed as `true`, then the customer's existing primary location (if any) will become secondary, and this one will become the primary one. | Default: If not passed and the customer does not have primary location, it takes the value `true`, else if the customer already have primary location, it takes the value `false`. |
| `is_gated` | boolean | No | Used to send the location's `is gated` flag that will be set. | Default: False |
| `is_bill_to` | boolean | No | Used to send the location's is bill to flag that will be set. | Default: False |
| `customer_contact` | string | No | Used to send a customer contact's `id` or `header` that will be attached to the location (Note: `id` - [integer] the customer contact's identifier, `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

**`custom_fields[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

**Example (0):**
```json
{
  "customer_name": "Bob Marley",
  "parent_customer": "Jerry Wheeler",
  "account_number": "30000",
  "private_notes": "None",
  "public_notes": "None",
  "credit_rating": "A+",
  "labor_charge_type": "flat",
  "labor_charge_default_rate": 50.45,
  "last_serviced_date": "2018-08-07",
  "is_bill_for_drive_time": true,
  "is_vip": true,
  "referral_source": "Google AdWords",
  "agent": "John Theowner",
  "discount": 10.23,
  "discount_type": "%",
  "payment_type": "Check",
  "payment_terms": "DUR",
  "assigned_contract": "Retail Service Contract",
  "industry": "Advertising Agencies",
  "is_taxable": false,
  "tax_item_name": "Sanity Tax",
  "qbo_sync_token": 385,
  "qbo_currency": "USD",
  "contacts": [
    {
      "prefix": "Mr.",
      "fname": "Jerry",
      "lname": "Wheeler",
      "suffix": "suf",
      "contact_type": "Billing",
      "dob": "April 19",
      "anniversary": "October 4",
      "job_title": "Manager",
      "department": "executive",
      "is_primary": true,
      "phones": [
        {
          "phone": "066-361-8172",
          "ext": 38,
          "type": "Mobile"
        }
      ],
      "emails": [
        {
          "email": "anton.lyubch1@gmail.com",
          "class": "Personal",
          "types_accepted": "CONF,PMT"
        }
      ]
    }
  ],
  "locations": [
    {
      "street_1": "1904 Industrial Blvd",
      "street_2": "103",
      "city": "Colleyville",
      "state_prov": "Texas",
      "postal_code": "76034",
      "country": "USA",
      "nickname": "Office",
      "gate_instructions": "Gate instructions",
      "latitude": "123.45",
      "longitude": "67.89",
      "location_type": "home",
      "is_primary": false,
      "is_gated": false,
      "is_bill_to": false,
      "customer_contact": "Sam Smith"
    }
  ],
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value"
    },
    {
      "name": "Textarea",
      "value": "Example text area value"
    },
    {
      "name": "Date",
      "value": "2018-10-05"
    },
    {
      "name": "Numeric",
      "value": "157.25"
    },
    {
      "name": "Select",
      "value": "1 one"
    },
    {
      "name": "Checkbox",
      "value": true
    }
  ]
}
```

**Responses:**

**`201`** ### 201 Created (Success)
The request has been fulfilled, resulting in the creation of a new resource.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Location` | string | Uri of new resource. | `https://api.servicefusion.com/v1/customers/1472281` |

Body schema: `CustomerView`

A customer's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The customer's identifier. |  |
| `customer_name` | string | No | The customer's name. |  |
| `fully_qualified_name` | string | No | The customer's fully qualified name. |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the customer (Note: `header` - [string] the parent customer's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `account_number` | string | No | The customer's account number. |  |
| `account_balance` | number (float) | No | The customer's account balance. |  |
| `private_notes` | string | No | The customer's private notes. |  |
| `public_notes` | string | No | The customer's public notes. |  |
| `credit_rating` | string | No | The customer's credit rating. |  |
| `labor_charge_type` | string | No | The customer's labor charge type. |  |
| `labor_charge_default_rate` | number (float) | No | The customer's labor charge default rate. |  |
| `last_serviced_date` | datetime | No | The customer's last serviced date. |  |
| `is_bill_for_drive_time` | boolean | No | The customer's is bill for drive time flag. |  |
| `is_vip` | boolean | No | The customer's is vip flag. |  |
| `referral_source` | string | No | The `header` of attached referral source to the customer (Note: `header` - [string] the referral source's fields concatenated by pattern `{short_name}`). |  |
| `agent` | string | No | The `header` of attached agent to the customer (Note: `header` - [string] the agent's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `discount` | number (float) | No | The customer's discount. |  |
| `discount_type` | string | No | The customer's discount type. |  |
| `payment_type` | string | No | The `header` of attached payment type to the customer (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `payment_terms` | string | No | The customer's payment terms. |  |
| `assigned_contract` | string | No | The `header` of attached contract to the customer (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `industry` | string | No | The `header` of attached industry to the customer (Note: `header` - [string] the industry's fields concatenated by pattern `{industry}`). |  |
| `is_taxable` | boolean | No | The customer's is taxable flag. |  |
| `tax_item_name` | string | No | The `header` of attached tax item to the customer (Note: `header` - [string] the tax item's fields concatenated by pattern `{short_name}` with space as separator). |  |
| `qbo_sync_token` | integer | No | The customer's qbo sync token. |  |
| `qbo_currency` | string | No | The customer's qbo currency. |  |
| `qbo_id` | integer | No | The customer's qbo id. |  |
| `qbd_id` | string | No | The customer's qbd id. |  |
| `created_at` | datetime | No | The customer's created date. |  |
| `updated_at` | datetime | No | The customer's updated date. |  |
| `contacts` | array[object] | No | The customer's contacts list. |  |
| `locations` | array[object] | No | The customer's locations list. |  |
| `custom_fields` | array[object] | No | The customer's custom fields list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**`contacts[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | The contact's prefix. |  |
| `fname` | string | No | The contact's first name. |  |
| `lname` | string | No | The contact's last name. |  |
| `suffix` | string | No | The contact's suffix. |  |
| `contact_type` | string | No | The contact's type. |  |
| `dob` | string | No | The contact's dob. |  |
| `anniversary` | string | No | The contact's anniversary. |  |
| `job_title` | string | No | The contact's job title. |  |
| `department` | string | No | The contact's department. |  |
| `created_at` | datetime | No | The contact's created date. |  |
| `updated_at` | datetime | No | The contact's updated date. |  |
| `is_primary` | boolean | No | The contact's is primary flag. |  |
| `phones` | array[object] | No | The contact's phones list. |  |
| `emails` | array[object] | No | The contact's emails list. |  |

**`contacts[].phones[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | No | The phone's number. |  |
| `ext` | integer | No | The phone's extension. |  |
| `type` | string | No | The phone's type. |  |
| `created_at` | datetime | No | The phone's created date. |  |
| `updated_at` | datetime | No | The phone's updated date. |  |
| `is_mobile` | boolean | No | The phone's is mobile flag. |  |

**`contacts[].emails[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | No | The email's address. |  |
| `class` | string | No | The email's class. |  |
| `types_accepted` | string | No | The email's types accepted. |  |
| `created_at` | datetime | No | The email's created date. |  |
| `updated_at` | datetime | No | The email's updated date. |  |

**`locations[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | No | The location's street 1. |  |
| `street_2` | string | No | The location's street 2. |  |
| `city` | string | No | The location's city. |  |
| `state_prov` | string | No | The location's state. |  |
| `postal_code` | string | No | The location's postal code. |  |
| `country` | string | No | The location's country. |  |
| `nickname` | string | No | The location's nickname. |  |
| `gate_instructions` | string | No | The location's gate instructions. |  |
| `latitude` | number (float) | No | The location's latitude. |  |
| `longitude` | number (float) | No | The location's longitude. |  |
| `location_type` | string | No | The location's type. |  |
| `created_at` | datetime | No | The location's created date. |  |
| `updated_at` | datetime | No | The location's updated date. |  |
| `is_primary` | boolean | No | The location's is primary flag. |  |
| `is_gated` | boolean | No | The location's is gated flag. |  |
| `is_bill_to` | boolean | No | The location's is bill to flag. |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the location (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

**`custom_fields[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**Example:**
```json
{
  "id": 1472289,
  "customer_name": "Bob Marley",
  "fully_qualified_name": "Bob Marley",
  "parent_customer": "Jerry Wheeler",
  "account_number": "30000",
  "account_balance": 10.34,
  "private_notes": "None",
  "public_notes": "None",
  "credit_rating": "A+",
  "labor_charge_type": "flat",
  "labor_charge_default_rate": 50.45,
  "last_serviced_date": "2018-08-07",
  "is_bill_for_drive_time": true,
  "is_vip": true,
  "referral_source": "Google AdWords",
  "agent": "John Theowner",
  "discount": 10.23,
  "discount_type": "%",
  "payment_type": "Check",
  "payment_terms": "DUR",
  "assigned_contract": "Retail Service Contract",
  "industry": "Advertising Agencies",
  "is_taxable": false,
  "tax_item_name": "Sanity Tax",
  "qbo_sync_token": 385,
  "qbo_currency": "USD",
  "qbo_id": null,
  "qbd_id": null,
  "created_at": "2018-08-07T18:31:28+00:00",
  "updated_at": "2018-08-07T18:31:28+00:00",
  "contacts": [
    {
      "prefix": "Mr.",
      "fname": "Jerry",
      "lname": "Wheeler",
      "suffix": "suf",
      "contact_type": "Billing",
      "dob": "April 19",
      "anniversary": "October 4",
      "job_title": "Manager",
      "department": "executive",
      "created_at": "2016-12-21T14:12:08+00:00",
      "updated_at": "2016-12-21T14:12:08+00:00",
      "is_primary": true,
      "phones": [
        {
          "phone": "066-361-8172",
          "ext": 38,
          "type": "Mobile",
          "created_at": "2018-10-05T11:51:48+00:00",
          "updated_at": "2018-10-05T11:54:09+00:00",
          "is_mobile": true
        }
      ],
      "emails": [
        {
          "email": "anton.lyubch1@gmail.com",
          "class": "Personal",
          "types_accepted": "CONF,PMT",
          "created_at": "2018-10-05T11:51:48+00:00",
          "updated_at": "2018-10-05T11:54:09+00:00"
        }
      ]
    }
  ],
  "locations": [
    {
      "street_1": "1904 Industrial Blvd",
      "street_2": "103",
      "city": "Colleyville",
      "state_prov": "Texas",
      "postal_code": "76034",
      "country": "USA",
      "nickname": "Office",
      "gate_instructions": "Gate instructions",
      "latitude": 123.45,
      "longitude": 67.89,
      "location_type": "home",
      "created_at": "2018-08-07T18:31:28+00:00",
      "updated_at": "2018-08-07T18:31:28+00:00",
      "is_primary": false,
      "is_gated": false,
      "is_bill_to": false,
      "customer_contact": "Sam Smith"
    }
  ],
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value",
      "type": "text",
      "group": "Default",
      "created_at": "2018-10-11T11:52:33+00:00",
      "updated_at": "2018-10-11T11:52:33+00:00",
      "is_required": true
    }
  ],
  "_expandable": [
    "contacts",
    "contacts.phones",
    "contacts.emails",
    "locations",
    "custom_fields"
  ]
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`422`** ### 422 Unprocessable Entity (Client Error)
The request was well-formed but was unable to be followed due to semantic errors.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `422Error`

Unprocessable entity client's error schema.

**Example:**
```json
[
  {
    "field": "name",
    "message": "Name is too long (maximum is 45 characters)."
  }
]
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### GET `/customers`

List all Customers matching query criteria, if provided,
otherwise list all Customers.

**Traits applied:** `tra.customer-fieldable`, `tra.customer-sortable`, `tra.customer-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `customer_name`, `fully_qualified_name`, `account_number`, `account_balance`, `private_notes`, `public_notes`, `payment_terms`, `discount`, `discount_type`, `credit_rating`, `labor_charge_type`, `labor_charge_default_rate`, `qbo_sync_token`, `qbo_currency`, `qbo_id`, `qbd_id`, `created_at`, `updated_at`, `last_serviced_date`, `is_bill_for_drive_time`, `is_vip`, `is_taxable`, `parent_customer`, `referral_source`, `agent`, `assigned_contract`, `payment_type`, `tax_item_name`, `industry` | `id,customer_name,discount` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `contacts`, `contacts.phones`, `contacts.emails`, `locations`, `custom_fields` | `contacts.phones,locations` |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `customer_name`, `fully_qualified_name`, `account_number`, `private_notes`, `public_notes`, `payment_terms`, `discount`, `discount_type`, `credit_rating`, `labor_charge_type`, `labor_charge_default_rate`, `qbo_sync_token`, `qbo_currency`, `qbo_id`, `qbd_id`, `created_at`, `updated_at`, `last_serviced_date`, `is_bill_for_drive_time`, `is_vip`, `is_taxable`, `parent_customer`, `referral_source`, `agent`, `assigned_contract`, `payment_type`, `tax_item_name`, `industry` | `-customer_name,created_at` |
| `filters[name]` | string | No | Used to filter results by given name (partial match). |  |  | `John` |
| `filters[contact_first_name]` | string | No | Used to filter results by given contact's first name (partial match). |  |  | `John` |
| `filters[contact_last_name]` | string | No | Used to filter results by given contact's last name (partial match). |  |  | `Walter` |
| `filters[address]` | string | No | Used to filter results by given address (partial match). |  |  | `3210 Midway Ave` |
| `filters[city]` | string | No | Used to filter results by given city (full match). |  |  | `Dallas` |
| `filters[postal_code]` | integer | No | Used to filter results by given postal code (full match). |  |  | `75242` |
| `filters[phone]` | string | No | Used to filter results by given phone (partial match). |  |  | `214-555-1212` |
| `filters[email]` | string | No | Used to filter results by given email (full match). |  |  | `john.walter@gmail.com` |
| `filters[tags]` | string | No | Used to filter results by given tags (full match). Accepted value is comma-separated string. |  |  | `Problem Customer, User` |
| `filters[last_serviced_date][lte]` | string | No | Used to filter results by given `less than or equal` of last serviced date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[last_serviced_date][gte]` | string | No | Used to filter results by given `greater than or equal` of last serviced date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[agreement_date_effective][lte]` | string | No | Used to filter results by given `less than or equal` of agreement date effective (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[agreement_date_effective][gte]` | string | No | Used to filter results by given `greater than or equal` of agreement date effective (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[agreement_date_expires][lte]` | string | No | Used to filter results by given `less than or equal` of agreement date expires (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[agreement_date_expires][gte]` | string | No | Used to filter results by given `greater than or equal` of agreement date expires (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A customer's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The customer's identifier. |  |
| `customer_name` | string | No | The customer's name. |  |
| `fully_qualified_name` | string | No | The customer's fully qualified name. |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the customer (Note: `header` - [string] the parent customer's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `account_number` | string | No | The customer's account number. |  |
| `account_balance` | number (float) | No | The customer's account balance. |  |
| `private_notes` | string | No | The customer's private notes. |  |
| `public_notes` | string | No | The customer's public notes. |  |
| `credit_rating` | string | No | The customer's credit rating. |  |
| `labor_charge_type` | string | No | The customer's labor charge type. |  |
| `labor_charge_default_rate` | number (float) | No | The customer's labor charge default rate. |  |
| `last_serviced_date` | datetime | No | The customer's last serviced date. |  |
| `is_bill_for_drive_time` | boolean | No | The customer's is bill for drive time flag. |  |
| `is_vip` | boolean | No | The customer's is vip flag. |  |
| `referral_source` | string | No | The `header` of attached referral source to the customer (Note: `header` - [string] the referral source's fields concatenated by pattern `{short_name}`). |  |
| `agent` | string | No | The `header` of attached agent to the customer (Note: `header` - [string] the agent's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `discount` | number (float) | No | The customer's discount. |  |
| `discount_type` | string | No | The customer's discount type. |  |
| `payment_type` | string | No | The `header` of attached payment type to the customer (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `payment_terms` | string | No | The customer's payment terms. |  |
| `assigned_contract` | string | No | The `header` of attached contract to the customer (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `industry` | string | No | The `header` of attached industry to the customer (Note: `header` - [string] the industry's fields concatenated by pattern `{industry}`). |  |
| `is_taxable` | boolean | No | The customer's is taxable flag. |  |
| `tax_item_name` | string | No | The `header` of attached tax item to the customer (Note: `header` - [string] the tax item's fields concatenated by pattern `{short_name}` with space as separator). |  |
| `qbo_sync_token` | integer | No | The customer's qbo sync token. |  |
| `qbo_currency` | string | No | The customer's qbo currency. |  |
| `qbo_id` | integer | No | The customer's qbo id. |  |
| `qbd_id` | string | No | The customer's qbd id. |  |
| `created_at` | datetime | No | The customer's created date. |  |
| `updated_at` | datetime | No | The customer's updated date. |  |
| `contacts` | array[object] | No | The customer's contacts list. |  |
| `locations` | array[object] | No | The customer's locations list. |  |
| `custom_fields` | array[object] | No | The customer's custom fields list. |  |

**`items[].contacts[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | The contact's prefix. |  |
| `fname` | string | No | The contact's first name. |  |
| `lname` | string | No | The contact's last name. |  |
| `suffix` | string | No | The contact's suffix. |  |
| `contact_type` | string | No | The contact's type. |  |
| `dob` | string | No | The contact's dob. |  |
| `anniversary` | string | No | The contact's anniversary. |  |
| `job_title` | string | No | The contact's job title. |  |
| `department` | string | No | The contact's department. |  |
| `created_at` | datetime | No | The contact's created date. |  |
| `updated_at` | datetime | No | The contact's updated date. |  |
| `is_primary` | boolean | No | The contact's is primary flag. |  |
| `phones` | array[object] | No | The contact's phones list. |  |
| `emails` | array[object] | No | The contact's emails list. |  |

**`items[].contacts[].phones[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | No | The phone's number. |  |
| `ext` | integer | No | The phone's extension. |  |
| `type` | string | No | The phone's type. |  |
| `created_at` | datetime | No | The phone's created date. |  |
| `updated_at` | datetime | No | The phone's updated date. |  |
| `is_mobile` | boolean | No | The phone's is mobile flag. |  |

**`items[].contacts[].emails[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | No | The email's address. |  |
| `class` | string | No | The email's class. |  |
| `types_accepted` | string | No | The email's types accepted. |  |
| `created_at` | datetime | No | The email's created date. |  |
| `updated_at` | datetime | No | The email's updated date. |  |

**`items[].locations[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | No | The location's street 1. |  |
| `street_2` | string | No | The location's street 2. |  |
| `city` | string | No | The location's city. |  |
| `state_prov` | string | No | The location's state. |  |
| `postal_code` | string | No | The location's postal code. |  |
| `country` | string | No | The location's country. |  |
| `nickname` | string | No | The location's nickname. |  |
| `gate_instructions` | string | No | The location's gate instructions. |  |
| `latitude` | number (float) | No | The location's latitude. |  |
| `longitude` | number (float) | No | The location's longitude. |  |
| `location_type` | string | No | The location's type. |  |
| `created_at` | datetime | No | The location's created date. |  |
| `updated_at` | datetime | No | The location's updated date. |  |
| `is_primary` | boolean | No | The location's is primary flag. |  |
| `is_gated` | boolean | No | The location's is gated flag. |  |
| `is_bill_to` | boolean | No | The location's is bill to flag. |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the location (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

**`items[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 1472289,
      "customer_name": "Bob Marley",
      "fully_qualified_name": "Bob Marley",
      "parent_customer": "Jerry Wheeler",
      "account_number": "30000",
      "account_balance": 10.34,
      "private_notes": "None",
      "public_notes": "None",
      "credit_rating": "A+",
      "labor_charge_type": "flat",
      "labor_charge_default_rate": 50.45,
      "last_serviced_date": "2018-08-07",
      "is_bill_for_drive_time": true,
      "is_vip": true,
      "referral_source": "Google AdWords",
      "agent": "John Theowner",
      "discount": 10.23,
      "discount_type": "%",
      "payment_type": "Check",
      "payment_terms": "DUR",
      "assigned_contract": "Retail Service Contract",
      "industry": "Advertising Agencies",
      "is_taxable": false,
      "tax_item_name": "Sanity Tax",
      "qbo_sync_token": 385,
      "qbo_currency": "USD",
      "qbo_id": null,
      "qbd_id": null,
      "created_at": "2018-08-07T18:31:28+00:00",
      "updated_at": "2018-08-07T18:31:28+00:00",
      "contacts": [
        {
          "prefix": "Mr.",
          "fname": "Jerry",
          "lname": "Wheeler",
          "suffix": "suf",
          "contact_type": "Billing",
          "dob": "April 19",
          "anniversary": "October 4",
          "job_title": "Manager",
          "department": "executive",
          "created_at": "2016-12-21T14:12:08+00:00",
          "updated_at": "2016-12-21T14:12:08+00:00",
          "is_primary": true,
          "phones": [
            {
              "phone": "066-361-8172",
              "ext": 38,
              "type": "Mobile",
              "created_at": "2018-10-05T11:51:48+00:00",
              "updated_at": "2018-10-05T11:54:09+00:00",
              "is_mobile": true
            }
          ],
          "emails": [
            {
              "email": "anton.lyubch1@gmail.com",
              "class": "Personal",
              "types_accepted": "CONF,PMT",
              "created_at": "2018-10-05T11:51:48+00:00",
              "updated_at": "2018-10-05T11:54:09+00:00"
            }
          ]
        }
      ],
      "locations": [
        {
          "street_1": "1904 Industrial Blvd",
          "street_2": "103",
          "city": "Colleyville",
          "state_prov": "Texas",
          "postal_code": "76034",
          "country": "USA",
          "nickname": "Office",
          "gate_instructions": "Gate instructions",
          "latitude": 123.45,
          "longitude": 67.89,
          "location_type": "home",
          "created_at": "2018-08-07T18:31:28+00:00",
          "updated_at": "2018-08-07T18:31:28+00:00",
          "is_primary": false,
          "is_gated": false,
          "is_bill_to": false,
          "customer_contact": "Sam Smith"
        }
      ],
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value",
          "type": "text",
          "group": "Default",
          "created_at": "2018-10-11T11:52:33+00:00",
          "updated_at": "2018-10-11T11:52:33+00:00",
          "is_required": true
        }
      ]
    }
  ],
  "_expandable": [
    "contacts",
    "contacts.phones",
    "contacts.emails",
    "locations",
    "custom_fields"
  ],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/customers/{customer-id}`

Get a Customer by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `customer-id` | integer | Yes | Used to send an identifier of the Customer to be used. |

##### GET `/customers/{customer-id}`

Get a Customer by identifier.

**Traits applied:** `tra.customer-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `customer_name`, `fully_qualified_name`, `account_number`, `account_balance`, `private_notes`, `public_notes`, `payment_terms`, `discount`, `discount_type`, `credit_rating`, `labor_charge_type`, `labor_charge_default_rate`, `qbo_sync_token`, `qbo_currency`, `qbo_id`, `qbd_id`, `created_at`, `updated_at`, `last_serviced_date`, `is_bill_for_drive_time`, `is_vip`, `is_taxable`, `parent_customer`, `referral_source`, `agent`, `assigned_contract`, `payment_type`, `tax_item_name`, `industry` | `id,customer_name,discount` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `contacts`, `contacts.phones`, `contacts.emails`, `locations`, `custom_fields` | `contacts.phones,locations` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `CustomerView`

A customer's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The customer's identifier. |  |
| `customer_name` | string | No | The customer's name. |  |
| `fully_qualified_name` | string | No | The customer's fully qualified name. |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the customer (Note: `header` - [string] the parent customer's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `account_number` | string | No | The customer's account number. |  |
| `account_balance` | number (float) | No | The customer's account balance. |  |
| `private_notes` | string | No | The customer's private notes. |  |
| `public_notes` | string | No | The customer's public notes. |  |
| `credit_rating` | string | No | The customer's credit rating. |  |
| `labor_charge_type` | string | No | The customer's labor charge type. |  |
| `labor_charge_default_rate` | number (float) | No | The customer's labor charge default rate. |  |
| `last_serviced_date` | datetime | No | The customer's last serviced date. |  |
| `is_bill_for_drive_time` | boolean | No | The customer's is bill for drive time flag. |  |
| `is_vip` | boolean | No | The customer's is vip flag. |  |
| `referral_source` | string | No | The `header` of attached referral source to the customer (Note: `header` - [string] the referral source's fields concatenated by pattern `{short_name}`). |  |
| `agent` | string | No | The `header` of attached agent to the customer (Note: `header` - [string] the agent's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `discount` | number (float) | No | The customer's discount. |  |
| `discount_type` | string | No | The customer's discount type. |  |
| `payment_type` | string | No | The `header` of attached payment type to the customer (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `payment_terms` | string | No | The customer's payment terms. |  |
| `assigned_contract` | string | No | The `header` of attached contract to the customer (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `industry` | string | No | The `header` of attached industry to the customer (Note: `header` - [string] the industry's fields concatenated by pattern `{industry}`). |  |
| `is_taxable` | boolean | No | The customer's is taxable flag. |  |
| `tax_item_name` | string | No | The `header` of attached tax item to the customer (Note: `header` - [string] the tax item's fields concatenated by pattern `{short_name}` with space as separator). |  |
| `qbo_sync_token` | integer | No | The customer's qbo sync token. |  |
| `qbo_currency` | string | No | The customer's qbo currency. |  |
| `qbo_id` | integer | No | The customer's qbo id. |  |
| `qbd_id` | string | No | The customer's qbd id. |  |
| `created_at` | datetime | No | The customer's created date. |  |
| `updated_at` | datetime | No | The customer's updated date. |  |
| `contacts` | array[object] | No | The customer's contacts list. |  |
| `locations` | array[object] | No | The customer's locations list. |  |
| `custom_fields` | array[object] | No | The customer's custom fields list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**`contacts[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `prefix` | string | No | The contact's prefix. |  |
| `fname` | string | No | The contact's first name. |  |
| `lname` | string | No | The contact's last name. |  |
| `suffix` | string | No | The contact's suffix. |  |
| `contact_type` | string | No | The contact's type. |  |
| `dob` | string | No | The contact's dob. |  |
| `anniversary` | string | No | The contact's anniversary. |  |
| `job_title` | string | No | The contact's job title. |  |
| `department` | string | No | The contact's department. |  |
| `created_at` | datetime | No | The contact's created date. |  |
| `updated_at` | datetime | No | The contact's updated date. |  |
| `is_primary` | boolean | No | The contact's is primary flag. |  |
| `phones` | array[object] | No | The contact's phones list. |  |
| `emails` | array[object] | No | The contact's emails list. |  |

**`contacts[].phones[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `phone` | string | No | The phone's number. |  |
| `ext` | integer | No | The phone's extension. |  |
| `type` | string | No | The phone's type. |  |
| `created_at` | datetime | No | The phone's created date. |  |
| `updated_at` | datetime | No | The phone's updated date. |  |
| `is_mobile` | boolean | No | The phone's is mobile flag. |  |

**`contacts[].emails[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `email` | string | No | The email's address. |  |
| `class` | string | No | The email's class. |  |
| `types_accepted` | string | No | The email's types accepted. |  |
| `created_at` | datetime | No | The email's created date. |  |
| `updated_at` | datetime | No | The email's updated date. |  |

**`locations[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `street_1` | string | No | The location's street 1. |  |
| `street_2` | string | No | The location's street 2. |  |
| `city` | string | No | The location's city. |  |
| `state_prov` | string | No | The location's state. |  |
| `postal_code` | string | No | The location's postal code. |  |
| `country` | string | No | The location's country. |  |
| `nickname` | string | No | The location's nickname. |  |
| `gate_instructions` | string | No | The location's gate instructions. |  |
| `latitude` | number (float) | No | The location's latitude. |  |
| `longitude` | number (float) | No | The location's longitude. |  |
| `location_type` | string | No | The location's type. |  |
| `created_at` | datetime | No | The location's created date. |  |
| `updated_at` | datetime | No | The location's updated date. |  |
| `is_primary` | boolean | No | The location's is primary flag. |  |
| `is_gated` | boolean | No | The location's is gated flag. |  |
| `is_bill_to` | boolean | No | The location's is bill to flag. |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the location (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |

**`custom_fields[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**Example:**
```json
{
  "id": 1472289,
  "customer_name": "Bob Marley",
  "fully_qualified_name": "Bob Marley",
  "parent_customer": "Jerry Wheeler",
  "account_number": "30000",
  "account_balance": 10.34,
  "private_notes": "None",
  "public_notes": "None",
  "credit_rating": "A+",
  "labor_charge_type": "flat",
  "labor_charge_default_rate": 50.45,
  "last_serviced_date": "2018-08-07",
  "is_bill_for_drive_time": true,
  "is_vip": true,
  "referral_source": "Google AdWords",
  "agent": "John Theowner",
  "discount": 10.23,
  "discount_type": "%",
  "payment_type": "Check",
  "payment_terms": "DUR",
  "assigned_contract": "Retail Service Contract",
  "industry": "Advertising Agencies",
  "is_taxable": false,
  "tax_item_name": "Sanity Tax",
  "qbo_sync_token": 385,
  "qbo_currency": "USD",
  "qbo_id": null,
  "qbd_id": null,
  "created_at": "2018-08-07T18:31:28+00:00",
  "updated_at": "2018-08-07T18:31:28+00:00",
  "contacts": [
    {
      "prefix": "Mr.",
      "fname": "Jerry",
      "lname": "Wheeler",
      "suffix": "suf",
      "contact_type": "Billing",
      "dob": "April 19",
      "anniversary": "October 4",
      "job_title": "Manager",
      "department": "executive",
      "created_at": "2016-12-21T14:12:08+00:00",
      "updated_at": "2016-12-21T14:12:08+00:00",
      "is_primary": true,
      "phones": [
        {
          "phone": "066-361-8172",
          "ext": 38,
          "type": "Mobile",
          "created_at": "2018-10-05T11:51:48+00:00",
          "updated_at": "2018-10-05T11:54:09+00:00",
          "is_mobile": true
        }
      ],
      "emails": [
        {
          "email": "anton.lyubch1@gmail.com",
          "class": "Personal",
          "types_accepted": "CONF,PMT",
          "created_at": "2018-10-05T11:51:48+00:00",
          "updated_at": "2018-10-05T11:54:09+00:00"
        }
      ]
    }
  ],
  "locations": [
    {
      "street_1": "1904 Industrial Blvd",
      "street_2": "103",
      "city": "Colleyville",
      "state_prov": "Texas",
      "postal_code": "76034",
      "country": "USA",
      "nickname": "Office",
      "gate_instructions": "Gate instructions",
      "latitude": 123.45,
      "longitude": 67.89,
      "location_type": "home",
      "created_at": "2018-08-07T18:31:28+00:00",
      "updated_at": "2018-08-07T18:31:28+00:00",
      "is_primary": false,
      "is_gated": false,
      "is_bill_to": false,
      "customer_contact": "Sam Smith"
    }
  ],
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value",
      "type": "text",
      "group": "Default",
      "created_at": "2018-10-11T11:52:33+00:00",
      "updated_at": "2018-10-11T11:52:33+00:00",
      "is_required": true
    }
  ],
  "_expandable": [
    "contacts",
    "contacts.phones",
    "contacts.emails",
    "locations",
    "custom_fields"
  ]
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

##### `/customers/{customer-id}/equipment`

**Resource type:** `res.read-only`

###### GET `/customers/{customer-id}/equipment`

List all Equipment matching query criteria, if provided,
otherwise list all Equipment.

**Traits applied:** `tra.equipment-fieldable`, `tra.equipment-sortable`, `tra.equipment-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `type`, `make`, `model`, `sku`, `serial_number`, `location`, `notes`, `extended_warranty_provider`, `is_extended_warranty`, `extended_warranty_date`, `warranty_date`, `install_date`, `created_at`, `updated_at`, `customer_id`, `customer`, `customer_location` | `id,location,customer_location` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `custom_fields` | `custom_fields` |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `type`, `make`, `model`, `sku`, `serial_number`, `location`, `notes`, `extended_warranty_provider`, `is_extended_warranty`, `extended_warranty_date`, `warranty_date`, `install_date`, `created_at`, `updated_at`, `customer_id`, `customer`, `customer_location` | `created_at,-type` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A equipment's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

**`items[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 12,
      "type": "Test Equipment",
      "make": "New Test Manufacturer",
      "model": "TST1231MOD",
      "sku": "SK15432",
      "serial_number": "1231#SRN",
      "location": "Test Location",
      "notes": "Test notes for the Test Equipment",
      "extended_warranty_provider": "Test War Provider",
      "is_extended_warranty": false,
      "extended_warranty_date": "2015-02-17",
      "warranty_date": "2015-01-16",
      "install_date": "2014-12-15",
      "created_at": "2015-01-16T11:31:49+00:00",
      "updated_at": "2015-01-16T11:31:49+00:00",
      "customer_id": 87,
      "customer": "John Theowner",
      "customer_location": "Office",
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value",
          "type": "text",
          "group": "Default",
          "created_at": "2018-10-11T11:52:33+00:00",
          "updated_at": "2018-10-11T11:52:33+00:00",
          "is_required": true
        }
      ]
    }
  ],
  "_expandable": [
    "custom_fields"
  ],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

###### `/customers/{customer-id}/equipment/{equipment-id}`

Get a Equipment by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `equipment-id` | integer | Yes | Used to send an identifier of the Equipment to be used. |

###### GET `/customers/{customer-id}/equipment/{equipment-id}`

Get a Equipment by identifier.

**Traits applied:** `tra.equipment-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `type`, `make`, `model`, `sku`, `serial_number`, `location`, `notes`, `extended_warranty_provider`, `is_extended_warranty`, `extended_warranty_date`, `warranty_date`, `install_date`, `created_at`, `updated_at`, `customer_id`, `customer`, `customer_location` | `id,location,customer_location` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `custom_fields` | `custom_fields` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `EquipmentView`

An equipment's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**`custom_fields[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**Example:**
```json
{
  "id": 12,
  "type": "Test Equipment",
  "make": "New Test Manufacturer",
  "model": "TST1231MOD",
  "sku": "SK15432",
  "serial_number": "1231#SRN",
  "location": "Test Location",
  "notes": "Test notes for the Test Equipment",
  "extended_warranty_provider": "Test War Provider",
  "is_extended_warranty": false,
  "extended_warranty_date": "2015-02-17",
  "warranty_date": "2015-01-16",
  "install_date": "2014-12-15",
  "created_at": "2015-01-16T11:31:49+00:00",
  "updated_at": "2015-01-16T11:31:49+00:00",
  "customer_id": 87,
  "customer": "John Theowner",
  "customer_location": "Office",
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value",
      "type": "text",
      "group": "Default",
      "created_at": "2018-10-11T11:52:33+00:00",
      "updated_at": "2018-10-11T11:52:33+00:00",
      "is_required": true
    }
  ],
  "_expandable": [
    "custom_fields"
  ]
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/jobs`

**Resource type:** `res.create-read-only`

#### POST `/jobs`

Create a new Job.

**Traits applied:** `tra.job-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Content-Type` | string | Yes | Used to send a format of data of the request. Enum: `application/json`, `application/x-www-form-urlencoded` |  |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `check_number`, `priority`, `description`, `tech_notes`, `completion_notes`, `payment_status`, `taxes_fees_total`, `drive_labor_total`, `billable_expenses_total`, `total`, `payments_deposits_total`, `due_total`, `cost_total`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `end_date`, `created_at`, `updated_at`, `closed_at`, `customer_id`, `customer_name`, `parent_customer`, `status`, `sub_status`, `contact_first_name`, `contact_last_name`, `street_1`, `street_2`, `city`, `state_prov`, `postal_code`, `location_name`, `is_gated`, `gate_instructions`, `category`, `source`, `payment_type`, `customer_payment_terms`, `project`, `phase`, `po_number`, `contract`, `note_to_customer`, `called_in_by`, `is_requires_follow_up` | `id,number,description` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `agents`, `custom_fields`, `pictures`, `documents`, `equipment`, `equipment.custom_fields`, `techs_assigned`, `tasks`, `notes`, `products`, `services`, `other_charges`, `labor_charges`, `expenses`, `payments`, `invoices`, `signatures`, `printable_work_order`, `visits`, `visits.techs_assigned` | `agents,equipment.custom_fields,visits.techs_assigned` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Request Body:**

Content-Type: `application/json` (Schema: `JobBody`)

A job's body schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `check_number` | string | No | Used to send the job's check number that will be set. |  |
| `priority` | string | No | Used to send the job's priority that will be set. | Enum: `Low`, `Normal`, `High`; Default: Normal |
| `description` | string | No | Used to send the job's description that will be set. |  |
| `tech_notes` | string | No | Used to send the job's tech notes that will be set. |  |
| `completion_notes` | string | No | Used to send the job's completion notes that will be set. |  |
| `duration` | integer | No | Used to send the job's duration (in seconds) that will be set. | Min: 0; Max: 86400; Default: 3600 |
| `time_frame_promised_start` | string | No | Used to send the job's time frame promised start that will be set. |  |
| `time_frame_promised_end` | string | No | Used to send the job's time frame promised end that will be set. |  |
| `start_date` | datetime | No | Used to send the job's start date that will be set. |  |
| `end_date` | datetime | No | Used to send the job's end date that will be set. |  |
| `customer_name` | string | Yes | Used to send a customer's `id` or `header` that will be attached to the job (Note: `id` - [integer] the customer's identifier, `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | Used to send a status'es `id` or `header` that will be attached to the job (Note: `id` - [integer] the status'es identifier, `header` - [string] the status'es fields concatenated by pattern `{name}`). Optionally required (configurable into the company preferences). | Default: If not passed, it takes the default status for jobs. |
| `contact_first_name` | string | No | Used to send the job's contact first name that will be set. If a contact with the passed name and surname already exists, then a new contact will not be created, but the existing one will be attached. | Max length: 255; Default: If not passed, it takes the first name from primary contact of the customer (if exists), otherwise a primary contact will be created for the customer. |
| `contact_last_name` | string | No | Used to send the job's contact last name that will be set. If a contact with the passed name and surname already exists, then a new contact will not be created, but the existing one will be attached. | Max length: 255; Default: If not passed, it takes the last name from primary contact of the customer (if exists), otherwise a primary contact will be created for the customer. |
| `street_1` | string | No | Used to send the job's location street 1 that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `street_2` | string | No | Used to send the job's location street 2 that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `city` | string | No | Used to send the job's location city that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `state_prov` | string | No | Used to send the job's location state prov that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `postal_code` | string | No | Used to send the job's location postal code that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `location_name` | string | No | Used to send the job's location name that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `is_gated` | boolean | No | Used to send the job's location is gated flag that will be set. | Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `gate_instructions` | string | No | Used to send the job's location gate instructions that will be set. | Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `category` | string | No | Used to send a category's `id` or `header` that will be attached to the job (Note: `id` - [integer] the category's identifier, `header` - [string] the category's fields concatenated by pattern `{category}`). Optionally required (configurable into the company preferences). |  |
| `source` | string | No | Used to send a source's `id` or `header` that will be attached to the job (Note: `id` - [integer] the source's identifier, `header` - [string] the source's fields concatenated by pattern `{short_name}`). | Default: If not passed, it takes the value from the customer. |
| `payment_type` | string | No | Used to send a payment type's `id` or `header` that will be attached to the job (Note: `id` - [integer] the payment type's identifier, `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). Optionally required (configurable into the company preferences). | Default: If not passed, it takes the value from the customer. |
| `customer_payment_terms` | string | No | Used to send a customer payment term's `id` or `header` that will be attached to the job (Note: `id` - [integer] the customer payment term's identifier, `header` - [string] the customer payment term's fields concatenated by pattern `{name}`). | Default: If not passed, it takes the value from the customer. |
| `project` | string | No | Used to send a project's `id` or `header` that will be attached to the job (Note: `id` - [integer] the project's identifier, `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | Used to send a phase's `id` or `header` that will be attached to the job (Note: `id` - [integer] the phase's identifier, `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | Used to send the job's po number that will be set. | Max length: 255 |
| `contract` | string | No | Used to send a contract's `id` or `header` that will be attached to the job (Note: `id` - [integer] the contract's identifier, `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). | Default: If not passed, it takes the value from the customer. |
| `note_to_customer` | string | No | Used to send the job's note to customer that will be set. | Default: If not passed, it takes the value from the company preferences. |
| `called_in_by` | string | No | Used to send the job's called in by that will be set. |  |
| `is_requires_follow_up` | boolean | No | Used to send the job's is requires follow up flag that will be set. | Default: False |
| `agents` | array[object] | No | Used to send the job's agents list that will be set. | Default: array |
| `custom_fields` | array[object] | No | Used to send the job's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |
| `equipment` | array[object] | No | Used to send the job's equipments list that will be set. | Default: array |
| `techs_assigned` | array[object] | No | Used to send the job's techs assigned list that will be set. | Default: array |
| `tasks` | array[object] | No | Used to send the job's tasks list that will be set. | Default: array |
| `notes` | array[object] | No | Used to send the job's notes list that will be set. | Default: array |
| `products` | array[object] | No | Used to send the job's products list that will be set. | Default: array |
| `services` | array[object] | No | Used to send the job's services list that will be set. | Default: array |
| `other_charges` | array[object] | No | Used to send the job's other charges list that will be set. | Default: If not passed, it creates all entries with `auto added` option enabled. Also it creates all not passed other charges declared into `products` and `services`. |
| `labor_charges` | array[object] | No | Used to send the job's labor charges list that will be set. | Default: array |
| `expenses` | array[object] | No | Used to send the job's expenses list that will be set. | Default: array |

**`agents[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | Used to send the agent's identifier that will be searched. If this field is set then the entry will be searched by it, otherwise the search will be performed by its full name. | Default: If not passed, it takes the value from full name search entry. |
| `first_name` | string | No | Used to send the agent's first name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |
| `last_name` | string | No | Used to send the agent's last name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |

**`custom_fields[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

**`equipment[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | string | No | Used to send the equipment's identifier that will be searched. You may pass this parameter if you do not want to create new entry but assign existing one. You may assign by `identifier` or `header` (Note: `identifier` - [integer] the equipment's identifier, `header` - [string] the equipment's fields concatenated by pattern `{type}:{make}:{model}:{serial_number}` with colon as separator). | Default: If not passed, it creates new one. |
| `type` | string | No | Used to send the equipment's type that will be set. |  |
| `make` | string | No | Used to send the equipment's make that will be set. |  |
| `model` | string | No | Used to send the equipment's model that will be set. | Max length: 255 |
| `sku` | string | No | Used to send the equipment's sku that will be set. | Max length: 50 |
| `serial_number` | string | No | Used to send the equipment's serial number that will be set. | Max length: 255 |
| `location` | string | No | Used to send the equipment's location that will be set. | Max length: 255 |
| `notes` | string | No | Used to send the equipment's notes that will be set. | Max length: 250 |
| `extended_warranty_provider` | string | No | Used to send the equipment's extended warranty provider that will be set. | Max length: 255 |
| `is_extended_warranty` | boolean | No | Used to send the equipment's is extended warranty flag that will be set. | Default: False |
| `extended_warranty_date` | datetime | No | Used to send the equipment's extended warranty date that will be set. |  |
| `warranty_date` | datetime | No | Used to send the equipment's warranty date that will be set. |  |
| `install_date` | datetime | No | Used to send the equipment's install date that will be set. |  |
| `customer_location` | string | No | Used to send a customer location's `id` or `header` that will be attached to the equipment (Note: `id` - [integer] the customer location's identifier, `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | Used to send the equipment's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |

**`equipment[].custom_fields[]` (nested array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

**`techs_assigned[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | Used to send the assigned tech's identifier that will be searched. If this field is set then the entry will be searched by it, otherwise the search will be performed by its full name. | Default: If not passed, it takes the value from full name search entry. |
| `first_name` | string | No | Used to send the assigned tech's first name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |
| `last_name` | string | No | Used to send the assigned tech's last name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |

**`tasks[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `description` | string | Yes | Used to send the task's description that will be set. | Max length: 500 |
| `is_completed` | boolean | No | Used to send the task's is completed flag that will be set. | Default: False |

**`notes[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | Yes | Used to send the note's text that will be set. |  |

**`products[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the product's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed product. |
| `description` | string | No | Used to send the product's description that will be set. | Default: If not passed, it takes the value of passed product. |
| `multiplier` | integer | No | Used to send the product's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed product. |
| `rate` | number (float) | No | Used to send the product's rate that will be set. | Default: If not passed, it takes the value of passed product. |
| `cost` | number (float) | No | Used to send the product's cost that will be set. | Default: If not passed, it takes the value of passed product. |
| `is_show_rate_items` | boolean | No | Used to send the product's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the product (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | Yes | Used to send a product's `id` or `header` that will be attached to the product (Note: `id` - [integer] the product's identifier, `header` - [string] the product's fields concatenated by pattern `{make}`). |  |

**`services[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the service's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed service. |
| `description` | string | No | Used to send the service's description that will be set. | Default: If not passed, it takes the value of passed service. |
| `multiplier` | integer | No | Used to send the service's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed service. |
| `rate` | number (float) | No | Used to send the service's rate that will be set. | Default: If not passed, it takes the value of passed service. |
| `cost` | number (float) | No | Used to send the service's cost that will be set. | Default: If not passed, it takes the value of passed service. |
| `is_show_rate_items` | boolean | No | Used to send the service's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the service (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | Yes | Used to send a service's `id` or `header` that will be attached to the service (Note: `id` - [integer] the service's identifier, `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |

**`other_charges[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the other charge's name that will be set. | Max length: 255; Default: If not passed, it takes the value of passed other charge. |
| `rate` | number (float) | No | Used to send the other charge's rate that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `is_percentage` | boolean | No | Used to send the other charge's is percentage flag that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `other_charge` | string | Yes | Used to send an other charge's `id` or `header` that will be attached to the other charge (Note: `id` - [integer] the other charge's identifier, `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |

**`labor_charges[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | Used to send the labor charge's drive time that will be set. Forbidden if drive times start/end passed. | Default: If drive times start/end passed, it takes the calculated difference time (in minutes), otherwise it takes the value `0`. |
| `drive_time_rate` | number (float) | No | Used to send the labor charge's drive time rate that will be set. | Default: 0 |
| `drive_time_cost` | number (float) | No | Used to send the labor charge's drive time cost that will be set. | Default: 0 |
| `drive_time_start` | string | No | Used to send the labor charge's drive time start that will be set. Required if drive time end passed. |  |
| `drive_time_end` | string | No | Used to send the labor charge's drive time end that will be set. Required if drive time start passed. Must be greater than drive time start. |  |
| `is_drive_time_billed` | boolean | No | Used to send the labor charge's is drive time billed flag that will be set. | Default: False |
| `labor_time` | integer | No | Used to send the labor charge's labor time that will be set. Forbidden if labor times start/end passed. | Default: If labor times start/end passed, it takes the calculated difference time (in minutes), otherwise it takes the value `0`. |
| `labor_time_rate` | number (float) | No | Used to send the labor charge's labor time rate that will be set. | Default: 0 |
| `labor_time_cost` | number (float) | No | Used to send the labor charge's labor time cost that will be set. | Default: 0 |
| `labor_time_start` | string | No | Used to send the labor charge's labor time start that will be set. Required if labor time end passed. |  |
| `labor_time_end` | string | No | Used to send the labor charge's labor time end that will be set. Required if labor time start passed. Must be greater than labor time start. |  |
| `labor_date` | datetime | No | Used to send the labor charge's labor date that will be set. |  |
| `is_labor_time_billed` | boolean | No | Used to send the labor charge's is labor time billed flag that will be set. | Default: False |
| `user` | string | No | Used to send a user's `id` or `header` that will be attached to the labor charge (Note: `id` - [integer] the user's identifier, `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |

**`expenses[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | Used to send the expense's purchased from that will be set. | Max length: 255 |
| `notes` | string | No | Used to send the expense's notes that will be set. |  |
| `amount` | number (float) | No | Used to send the expense's amount that will be set. | Default: 0 |
| `is_billable` | boolean | No | Used to send the expense's is billable flag that will be set. | Default: False |
| `date` | datetime | No | Used to send the expense's date that will be set. |  |
| `user` | string | No | Used to send a user's `id` or `header` that will be attached to the expense (Note: `id` - [integer] the user's identifier, `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | Used to send a category's `id` or `header` that will be attached to the expense (Note: `id` - [integer] the category's identifier, `header` - [string] the category's fields concatenated by pattern `{category_name}`). | Default: If not passed, it takes the name of first existing category. |

**Example (0):**
```json
{
  "check_number": "1877",
  "priority": "Normal",
  "description": "This is a test",
  "tech_notes": "You guys know what to do.",
  "completion_notes": "Work is done.",
  "duration": 3600,
  "time_frame_promised_start": "14:10",
  "time_frame_promised_end": "14:10",
  "start_date": "2015-01-08",
  "end_date": "2016-01-08",
  "customer_name": "Max Paltsev",
  "status": "Cancelled",
  "contact_first_name": "Sam",
  "contact_last_name": "Smith",
  "street_1": "1904 Industrial Blvd",
  "street_2": "103",
  "city": "Colleyville",
  "state_prov": "Texas",
  "postal_code": "76034",
  "location_name": "Office",
  "is_gated": false,
  "gate_instructions": null,
  "category": "Quick Home Energy Check-ups",
  "source": "Yellow Pages",
  "payment_type": "Direct Bill",
  "customer_payment_terms": "COD",
  "project": "reshma",
  "phase": "Closeup",
  "po_number": "86305",
  "contract": "Retail Service Contract",
  "note_to_customer": "Sample Note To Customer.",
  "called_in_by": "Sample Called In By",
  "is_requires_follow_up": true,
  "agents": [
    {
      "id": 31
    },
    {
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value"
    },
    {
      "name": "Textarea",
      "value": "Example text area value"
    },
    {
      "name": "Date",
      "value": "2018-10-05"
    },
    {
      "name": "Numeric",
      "value": "157.25"
    },
    {
      "name": "Select",
      "value": "1 one"
    },
    {
      "name": "Checkbox",
      "value": true
    }
  ],
  "equipment": [
    {
      "id": "COIL:ABUS:LMU-2620i:445577998871",
      "type": "Test Equipment",
      "make": "New Test Manufacturer",
      "model": "TST1231MOD",
      "sku": "SK15432",
      "serial_number": "1231#SRN",
      "location": "Test Location",
      "notes": "Test notes for the Test Equipment",
      "extended_warranty_provider": "Test War Provider",
      "is_extended_warranty": false,
      "extended_warranty_date": "2015-02-17",
      "warranty_date": "2015-01-16",
      "install_date": "2014-12-15",
      "customer_location": "Office",
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value"
        },
        {
          "name": "Textarea",
          "value": "Example text area value"
        },
        {
          "name": "Date",
          "value": "2018-10-05"
        },
        {
          "name": "Numeric",
          "value": "157.25"
        },
        {
          "name": "Select",
          "value": "1 one"
        },
        {
          "name": "Checkbox",
          "value": true
        }
      ]
    }
  ],
  "techs_assigned": [
    {
      "id": 31
    },
    {
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "tasks": [
    {
      "description": "x",
      "is_completed": false
    }
  ],
  "notes": [
    {
      "notes": "SHOULD BE DELIVERED TO US 6/1/15 AND RICHARD NEEDS TO PAINT"
    }
  ],
  "products": [
    {
      "name": "1755LFB-NEW",
      "description": "Finishing Trim Kit - 1\" - Black\r\nModel: \r\nSKU: \r\nType: \r\nPart Number: ",
      "multiplier": 2,
      "rate": 500,
      "cost": 100,
      "is_show_rate_items": false,
      "tax": "FIXED",
      "product": "1755LFB"
    }
  ],
  "services": [
    {
      "name": "Service Call Fee",
      "description": null,
      "multiplier": 1,
      "rate": "33.15",
      "cost": "121",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "service": "Nabeeel"
    }
  ],
  "other_charges": [
    {
      "name": "fee1 new",
      "rate": "15.15",
      "is_percentage": false,
      "other_charge": "fee1"
    }
  ],
  "labor_charges": [
    {
      "drive_time_rate": "10.25",
      "drive_time_cost": "75.75",
      "drive_time_start": "10:00",
      "drive_time_end": "12:00",
      "is_drive_time_billed": false,
      "labor_time": 75,
      "labor_time_rate": "11.25",
      "labor_time_cost": "50",
      "labor_date": "2015-11-19",
      "is_labor_time_billed": true,
      "user": "Test qa"
    }
  ],
  "expenses": [
    {
      "purchased_from": "test",
      "notes": null,
      "amount": "15.25",
      "is_billable": true,
      "date": "2016-01-19",
      "user": null,
      "category": "Accounting fees"
    }
  ]
}
```

**Responses:**

**`201`** ### 201 Created (Success)
The request has been fulfilled, resulting in the creation of a new resource.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Location` | string | Uri of new resource. | `https://api.servicefusion.com/v1/customers/1472281` |

Body schema: `JobView`

A job's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job's identifier. |  |
| `number` | string | No | The job's number. |  |
| `check_number` | string | No | The job's check number. |  |
| `priority` | string | No | The job's priority. |  |
| `description` | string | No | The job's description. |  |
| `tech_notes` | string | No | The job's tech notes. |  |
| `completion_notes` | string | No | The job's completion notes. |  |
| `payment_status` | string | No | The job's payment status. |  |
| `taxes_fees_total` | number (float) | No | The job's taxes and fees total. |  |
| `drive_labor_total` | number (float) | No | The job's drive and labor total. |  |
| `billable_expenses_total` | number (float) | No | The job's billable expenses total. |  |
| `total` | number (float) | No | The job's total. |  |
| `payments_deposits_total` | number (float) | No | The job's payments and deposits total. |  |
| `due_total` | number (float) | No | The job's due total. |  |
| `cost_total` | number (float) | No | The job's cost total. |  |
| `duration` | integer | No | The job's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The job's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The job's time frame promised end. |  |
| `start_date` | datetime | No | The job's start date. |  |
| `end_date` | datetime | No | The job's end date. |  |
| `created_at` | datetime | No | The job's created date. |  |
| `updated_at` | datetime | No | The job's updated date. |  |
| `closed_at` | datetime | No | The job's closed date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the job (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the job (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the job (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the job (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the job (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The job's contact first name. |  |
| `contact_last_name` | string | No | The job's contact last name. |  |
| `street_1` | string | No | The job's location street 1. |  |
| `street_2` | string | No | The job's location street 2. |  |
| `city` | string | No | The job's location city. |  |
| `state_prov` | string | No | The job's location state prov. |  |
| `postal_code` | string | No | The job's location postal code. |  |
| `location_name` | string | No | The job's location name. |  |
| `is_gated` | boolean | No | The job's location is gated flag. |  |
| `gate_instructions` | string | No | The job's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the job (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the job (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the job (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `customer_payment_terms` | string | No | The `header` of attached customer payment term to the job (Note: `header` - [string] the customer payment term's fields concatenated by pattern `{name}`). |  |
| `project` | string | No | The `header` of attached project to the job (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the job (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The job's po number. |  |
| `contract` | string | No | The `header` of attached contract to the job (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The job's note to customer. |  |
| `called_in_by` | string | No | The job's called in by. |  |
| `is_requires_follow_up` | boolean | No | The job's is requires follow up flag. |  |
| `agents` | array[object] | No | The job's agents list. |  |
| `custom_fields` | array[object] | No | The job's custom fields list. |  |
| `pictures` | array[object] | No | The job's pictures list. |  |
| `documents` | array[object] | No | The job's documents list. |  |
| `equipment` | array[object] | No | The job's equipments list. |  |
| `techs_assigned` | array[object] | No | The job's techs assigned list. |  |
| `tasks` | array[object] | No | The job's tasks list. |  |
| `notes` | array[object] | No | The job's notes list. |  |
| `products` | array[object] | No | The job's products list. |  |
| `services` | array[object] | No | The job's services list. |  |
| `other_charges` | array[object] | No | The job's other charges list. |  |
| `labor_charges` | array[object] | No | The job's labor charges list. |  |
| `expenses` | array[object] | No | The job's expenses list. |  |
| `payments` | array[object] | No | The job's payments list. |  |
| `invoices` | array[object] | No | The job's invoices list. |  |
| `signatures` | array[object] | No | The job's signatures list. |  |
| `printable_work_order` | array[object] | No | The job's printable work order list. |  |
| `visits` | array[object] | No | The job's visits list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**`agents[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

**`custom_fields[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`pictures[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`documents[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`equipment[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

**`equipment[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`techs_assigned[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

**`tasks[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

**`notes[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

**`products[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

**`services[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

**`other_charges[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`labor_charges[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | The labor charge's drive time. |  |
| `drive_time_rate` | number (float) | No | The labor charge's drive time rate. |  |
| `drive_time_cost` | number (float) | No | The labor charge's drive time cost. |  |
| `drive_time_start` | string | No | The labor charge's drive time start. |  |
| `drive_time_end` | string | No | The labor charge's drive time end. |  |
| `is_drive_time_billed` | boolean | No | The labor charge's is drive time billed flag. |  |
| `labor_time` | integer | No | The labor charge's labor time. |  |
| `labor_time_rate` | number (float) | No | The labor charge's labor time rate. |  |
| `labor_time_cost` | number (float) | No | The labor charge's labor time cost. |  |
| `labor_time_start` | string | No | The labor charge's labor time start. |  |
| `labor_time_end` | string | No | The labor charge's labor time end. |  |
| `labor_date` | datetime | No | The labor charge's labor date. |  |
| `is_labor_time_billed` | boolean | No | The labor charge's is labor time billed flag. |  |
| `total` | number (float) | No | The labor charge's total. |  |
| `created_at` | datetime | No | The labor charge's created date. |  |
| `updated_at` | datetime | No | The labor charge's updated date. |  |
| `is_status_generated` | boolean | No | The labor charge's is status generated flag. |  |
| `user` | string | No | The `header` of attached user to the labor charge (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `visit_id` | integer | No | The `id` of attached visit to the labor charge (Note: `id` - [integer] the visit's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the labor charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the labor charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`expenses[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | The expense's purchased from. |  |
| `notes` | string | No | The expense's notes. |  |
| `amount` | number (float) | No | The expense's amount. |  |
| `is_billable` | boolean | No | The expense's is billable flag. |  |
| `date` | datetime | No | The expense's date. |  |
| `created_at` | datetime | No | The expense's created date. |  |
| `updated_at` | datetime | No | The expense's updated date. |  |
| `user` | string | No | The `header` of attached user to the expense (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | The `header` of attached category to the expense (Note: `header` - [string] the category's fields concatenated by pattern `{category_name}`). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the expense (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the expense (Note: `id` - [integer] the qbd class'es identifier). |  |

**`payments[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

**`invoices[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |

**`signatures[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

**`printable_work_order[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

**`visits[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes_for_techs` | string | No | The visit's notes for techs. |  |
| `time_frame_promised_start` | string | No | The visit's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The visit's time frame promised end. |  |
| `duration` | integer | No | The visit's duration (in seconds). |  |
| `is_text_notified` | boolean | No | The visit's is text notified flag. |  |
| `is_voice_notified` | boolean | No | The visit's is voice notified flag. |  |
| `start_date` | datetime | No | The visit's start date. |  |
| `techs_assigned` | array[object] | No | The visit's techs assigned list. |  |

**`visits[].techs_assigned[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |
| `status` | string | No | The assigned tech's status. |  |

**Example:**
```json
{
  "id": 13,
  "number": "1152157",
  "check_number": "1877",
  "priority": "Normal",
  "description": "This is a test",
  "tech_notes": "You guys know what to do.",
  "completion_notes": "Work is done.",
  "customer_payment_terms": "COD",
  "payment_status": "Unpaid",
  "taxes_fees_total": 193.25,
  "drive_labor_total": 0,
  "billable_expenses_total": 0,
  "total": 193,
  "payments_deposits_total": 0,
  "due_total": 193,
  "cost_total": 0,
  "duration": 3600,
  "time_frame_promised_start": "14:10",
  "time_frame_promised_end": "14:10",
  "start_date": "2015-01-08",
  "end_date": "2016-01-08",
  "created_at": "2014-09-08T20:42:04+00:00",
  "updated_at": "2016-01-07T17:20:36+00:00",
  "closed_at": "2016-01-07T17:20:36+00:00",
  "customer_id": 11,
  "customer_name": "Max Paltsev",
  "parent_customer": "Jerry Wheeler",
  "status": "Cancelled",
  "sub_status": "job1",
  "contact_first_name": "Sam",
  "contact_last_name": "Smith",
  "street_1": "1904 Industrial Blvd",
  "street_2": "103",
  "city": "Colleyville",
  "state_prov": "Texas",
  "postal_code": "76034",
  "location_name": "Office",
  "is_gated": false,
  "gate_instructions": null,
  "category": "Quick Home Energy Check-ups",
  "source": "Yellow Pages",
  "payment_type": "Direct Bill",
  "project": "reshma",
  "phase": "Closeup",
  "po_number": "86305",
  "contract": "Retail Service Contract",
  "note_to_customer": "Sample Note To Customer.",
  "called_in_by": "Sample Called In By",
  "is_requires_follow_up": true,
  "agents": [
    {
      "id": 31,
      "first_name": "Justin",
      "last_name": "Wormell"
    },
    {
      "id": 32,
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value",
      "type": "text",
      "group": "Default",
      "created_at": "2018-10-11T11:52:33+00:00",
      "updated_at": "2018-10-11T11:52:33+00:00",
      "is_required": true
    }
  ],
  "pictures": [
    {
      "name": "1442951633_images.jpeg",
      "file_location": "1442951633_images.jpeg",
      "doc_type": "IMG",
      "comment": null,
      "sort": 2,
      "is_private": false,
      "created_at": "2015-09-22T19:53:53+00:00",
      "updated_at": "2015-09-22T19:53:53+00:00",
      "customer_doc_id": 992
    }
  ],
  "documents": [
    {
      "name": "test1John.pdf",
      "file_location": "1421408539_test1John.pdf",
      "doc_type": "DOC",
      "comment": null,
      "sort": 1,
      "is_private": false,
      "created_at": "2015-01-16T11:42:19+00:00",
      "updated_at": "2018-08-21T08:21:14+00:00",
      "customer_doc_id": 998
    }
  ],
  "equipment": [
    {
      "id": 12,
      "type": "Test Equipment",
      "make": "New Test Manufacturer",
      "model": "TST1231MOD",
      "sku": "SK15432",
      "serial_number": "1231#SRN",
      "location": "Test Location",
      "notes": "Test notes for the Test Equipment",
      "extended_warranty_provider": "Test War Provider",
      "is_extended_warranty": false,
      "extended_warranty_date": "2015-02-17",
      "warranty_date": "2015-01-16",
      "install_date": "2014-12-15",
      "created_at": "2015-01-16T11:31:49+00:00",
      "updated_at": "2015-01-16T11:31:49+00:00",
      "customer_id": 87,
      "customer": "John Theowner",
      "customer_location": "Office",
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value",
          "type": "text",
          "group": "Default",
          "created_at": "2018-10-11T11:52:33+00:00",
          "updated_at": "2018-10-11T11:52:33+00:00",
          "is_required": true
        }
      ]
    }
  ],
  "techs_assigned": [
    {
      "id": 31,
      "first_name": "Justin",
      "last_name": "Wormell"
    },
    {
      "id": 32,
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "tasks": [
    {
      "type": "Misc",
      "description": "x",
      "start_time": null,
      "start_date": null,
      "end_date": null,
      "is_completed": false,
      "created_at": "2017-03-20T10:48:38+00:00",
      "updated_at": "2017-03-20T10:48:38+00:00"
    }
  ],
  "notes": [
    {
      "notes": "SHOULD BE DELIVERED TO US 6/1/15 AND RICHARD NEEDS TO PAINT",
      "created_at": "2015-05-27T16:32:06+00:00",
      "updated_at": "2015-05-27T16:32:06+00:00"
    }
  ],
  "products": [
    {
      "name": "1755LFB",
      "description": "Finishing Trim Kit - 1\" - Black\r\nModel: \r\nSKU: \r\nType: \r\nPart Number: ",
      "multiplier": 3,
      "rate": 459,
      "total": 1377,
      "cost": 0,
      "actual_cost": 0,
      "item_index": 0,
      "parent_index": 0,
      "created_at": "2015-08-20T09:08:36+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "product": "1755LFB",
      "product_list_id": 45302,
      "warehouse_id": 200,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "services": [
    {
      "name": "Service Call Fee",
      "description": null,
      "multiplier": 1,
      "rate": 33.15,
      "total": 121,
      "cost": 121,
      "actual_cost": 121,
      "item_index": 3,
      "parent_index": 0,
      "created_at": "2015-08-20T09:08:36+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "service": "Nabeeel",
      "service_list_id": 45302,
      "service_rate_id": 200,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "other_charges": [
    {
      "name": "fee1",
      "rate": 5.15,
      "total": 14.3,
      "charge_index": 1,
      "parent_index": 1,
      "is_percentage": true,
      "is_discount": false,
      "created_at": "2015-08-20T09:08:52+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "other_charge": "fee1",
      "applies_to": null,
      "service_list_id": null,
      "other_charge_id": 248,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "labor_charges": [
    {
      "drive_time": 0,
      "drive_time_rate": 10.25,
      "drive_time_cost": 0,
      "drive_time_start": null,
      "drive_time_end": null,
      "is_drive_time_billed": false,
      "labor_time": 0,
      "labor_time_rate": 11.25,
      "labor_time_cost": 0,
      "labor_time_start": null,
      "labor_time_end": null,
      "labor_date": "2015-11-19",
      "is_labor_time_billed": true,
      "total": 0,
      "created_at": "2015-11-19T20:38:10+00:00",
      "updated_at": "-0001-11-30T00:00:00+00:00",
      "is_status_generated": true,
      "user": "Test qa",
      "visit_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "expenses": [
    {
      "purchased_from": "test",
      "notes": null,
      "amount": 15.25,
      "is_billable": true,
      "date": "2016-01-19",
      "created_at": "2016-01-07T17:20:36+00:00",
      "updated_at": "-0001-11-30T00:00:00+00:00",
      "user": null,
      "category": "Accounting fees",
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "payments": [
    {
      "transaction_type": "AUTH_CAPTURE",
      "transaction_token": "4Tczi4OI12MeoSaC4FG2VPKj1",
      "transaction_id": "257494-0_10",
      "payment_transaction_id": 10,
      "original_transaction_id": 110,
      "apply_to": "JOB",
      "amount": 10.35,
      "memo": null,
      "authorization_code": "755972",
      "bill_to_street_address": "adddad",
      "bill_to_postal_code": "adadadd",
      "bill_to_country": null,
      "reference_number": "1976/1410",
      "is_resync_qbo": false,
      "created_at": "2015-09-25T09:56:57+00:00",
      "updated_at": "2015-09-25T09:56:57+00:00",
      "received_on": "2015-09-25T00:00:00+00:00",
      "qbo_synced_date": "2015-09-25T00:00:00+00:00",
      "qbo_id": 5,
      "qbd_id": "3792-1438659918",
      "customer": "Max Paltsev",
      "type": "Cash",
      "invoice_id": 124,
      "gateway_id": 980190963,
      "receipt_id": "ord-250915-9:56:56"
    }
  ],
  "invoices": [
    {
      "id": 13,
      "number": 1001,
      "currency": "$",
      "po_number": null,
      "terms": "DUR",
      "customer_message": null,
      "notes": null,
      "pay_online_url": "https://app.servicefusion.com/invoiceOnline?id=WP7y6F6Ff48NqjQym4qX1maGXL_1oljugHAP0fNVaBg&key=0DtZ_Q5p4UZNqQHcx08U1k2dx8B3ZHKg3pBxavOtH61",
      "qbo_invoice_no": null,
      "qbo_sync_token": null,
      "qbo_synced_date": "2014-01-21T22:11:31+00:00",
      "qbo_id": null,
      "qbd_id": null,
      "total": 268.32,
      "is_paid": false,
      "date": "2014-01-21T00:00:00+00:00",
      "mail_send_date": null,
      "created_at": "2014-01-21T22:11:31+00:00",
      "updated_at": "2014-01-21T22:11:31+00:00",
      "customer": null,
      "customer_contact": null,
      "payment_terms": "Due Upon Receipt",
      "bill_to_customer_id": null,
      "bill_to_customer_location_id": null,
      "bill_to_customer_contact_id": null,
      "bill_to_email_id": null,
      "bill_to_phone_id": null
    }
  ],
  "signatures": [
    {
      "type": "PREWORK",
      "file_name": "https://servicefusion.s3.amazonaws.com/images/sign/139350-2015-08-25-11-35-14.png",
      "created_at": "2015-08-25T11:35:14+00:00",
      "updated_at": "2015-08-25T11:35:14+00:00"
    }
  ],
  "printable_work_order": [
    {
      "name": "Print With Rates",
      "url": "https://servicefusion.com/printJobWithRates?jobId=fF7HY2Dew1E9vw2mm8FHzSOrpDrKnSl-m2WKf0Yg_Kw"
    }
  ],
  "visits": [
    {
      "notes_for_techs": "Hahahaha",
      "time_frame_promised_start": "00:00",
      "time_frame_promised_end": "00:30",
      "duration": 3600,
      "is_text_notified": false,
      "is_voice_notified": false,
      "start_date": "2018-08-21",
      "techs_assigned": [
        {
          "id": 31,
          "first_name": "Justin",
          "last_name": "Wormell",
          "status": "Started"
        },
        {
          "id": 32,
          "first_name": "John",
          "last_name": "Theowner",
          "status": "Paused"
        }
      ]
    }
  ],
  "_expandable": [
    "agents",
    "custom_fields",
    "pictures",
    "documents",
    "equipment",
    "equipment.custom_fields",
    "techs_assigned",
    "tasks",
    "notes",
    "products",
    "services",
    "other_charges",
    "labor_charges",
    "expenses",
    "payments",
    "invoices",
    "signatures",
    "printable_work_order",
    "visits",
    "visits.techs_assigned"
  ]
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`422`** ### 422 Unprocessable Entity (Client Error)
The request was well-formed but was unable to be followed due to semantic errors.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `422Error`

Unprocessable entity client's error schema.

**Example:**
```json
[
  {
    "field": "name",
    "message": "Name is too long (maximum is 45 characters)."
  }
]
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### GET `/jobs`

List all Jobs matching query criteria, if provided,
otherwise list all Jobs.

**Traits applied:** `tra.job-fieldable`, `tra.job-sortable`, `tra.job-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `check_number`, `priority`, `description`, `tech_notes`, `completion_notes`, `payment_status`, `taxes_fees_total`, `drive_labor_total`, `billable_expenses_total`, `total`, `payments_deposits_total`, `due_total`, `cost_total`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `end_date`, `created_at`, `updated_at`, `closed_at`, `customer_id`, `customer_name`, `parent_customer`, `status`, `sub_status`, `contact_first_name`, `contact_last_name`, `street_1`, `street_2`, `city`, `state_prov`, `postal_code`, `location_name`, `is_gated`, `gate_instructions`, `category`, `source`, `payment_type`, `customer_payment_terms`, `project`, `phase`, `po_number`, `contract`, `note_to_customer`, `called_in_by`, `is_requires_follow_up` | `id,number,description` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `agents`, `custom_fields`, `pictures`, `documents`, `equipment`, `equipment.custom_fields`, `techs_assigned`, `tasks`, `notes`, `products`, `services`, `other_charges`, `labor_charges`, `expenses`, `payments`, `invoices`, `signatures`, `printable_work_order`, `visits`, `visits.techs_assigned` | `agents,equipment.custom_fields,visits.techs_assigned` |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `number`, `po_number`, `check_number`, `description`, `tech_notes`, `completion_notes`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `end_date`, `created_at`, `updated_at`, `closed_at`, `customer_id`, `customer_name`, `status`, `sub_status`, `category`, `source`, `payment_type`, `customer_payment_terms`, `contract`, `called_in_by` | `number,-start_date` |
| `filters[status]` | string | No | Used to filter results by given statuses (full match). Accepted value is comma-separated string. |  |  | `Job Closed, Cancelled` |
| `filters[number]` | string | No | Used to filter results by given number (partial match). |  |  | `101` |
| `filters[po_number]` | string | No | Used to filter results by given po number (partial match). |  |  | `101` |
| `filters[invoice_number]` | string | No | Used to filter results by given invoice number (partial match). |  |  | `101` |
| `filters[customer_name]` | string | No | Used to filter results by given customer's name (partial match). |  |  | `John Walter` |
| `filters[parent_customer_name]` | string | No | Used to filter results by given parent customer's name (partial match). |  |  | `John Walter` |
| `filters[contact_first_name]` | string | No | Used to filter results by given contact's first name (partial match). |  |  | `John` |
| `filters[contact_last_name]` | string | No | Used to filter results by given contact's last name (partial match). |  |  | `Walter` |
| `filters[address]` | string | No | Used to filter results by given address (partial match). |  |  | `3210 Midway Ave` |
| `filters[city]` | string | No | Used to filter results by given city (full match). |  |  | `Dallas` |
| `filters[zip_code]` | integer | No | Used to filter results by given zip code (full match). |  |  | `75242` |
| `filters[phone]` | string | No | Used to filter results by given phone (partial match). |  |  | `214-555-1212` |
| `filters[email]` | string | No | Used to filter results by given email (full match). |  |  | `john.walter@gmail.com` |
| `filters[category]` | string | No | Used to filter results by given categories (full match). Accepted value is comma-separated string. |  |  | `Install, Service Call` |
| `filters[source]` | string | No | Used to filter results by given sources (full match). Accepted value is comma-separated string. |  |  | `Google, Yelp` |
| `filters[start_date][lte]` | string | No | Used to filter results by given `less than or equal` of start date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[start_date][gte]` | string | No | Used to filter results by given `greater than or equal` of start date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[end_date][lte]` | string | No | Used to filter results by given `less than or equal` of end date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[end_date][gte]` | string | No | Used to filter results by given `greater than or equal` of end date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[updated_date][lte]` | string | No | Used to filter results by given `less than or equal` of updated date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[updated_date][gte]` | string | No | Used to filter results by given `greater than or equal` of updated date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[closed_date][lte]` | string | No | Used to filter results by given `less than or equal` of closed date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[closed_date][gte]` | string | No | Used to filter results by given `greater than or equal` of closed date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A job's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job's identifier. |  |
| `number` | string | No | The job's number. |  |
| `check_number` | string | No | The job's check number. |  |
| `priority` | string | No | The job's priority. |  |
| `description` | string | No | The job's description. |  |
| `tech_notes` | string | No | The job's tech notes. |  |
| `completion_notes` | string | No | The job's completion notes. |  |
| `payment_status` | string | No | The job's payment status. |  |
| `taxes_fees_total` | number (float) | No | The job's taxes and fees total. |  |
| `drive_labor_total` | number (float) | No | The job's drive and labor total. |  |
| `billable_expenses_total` | number (float) | No | The job's billable expenses total. |  |
| `total` | number (float) | No | The job's total. |  |
| `payments_deposits_total` | number (float) | No | The job's payments and deposits total. |  |
| `due_total` | number (float) | No | The job's due total. |  |
| `cost_total` | number (float) | No | The job's cost total. |  |
| `duration` | integer | No | The job's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The job's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The job's time frame promised end. |  |
| `start_date` | datetime | No | The job's start date. |  |
| `end_date` | datetime | No | The job's end date. |  |
| `created_at` | datetime | No | The job's created date. |  |
| `updated_at` | datetime | No | The job's updated date. |  |
| `closed_at` | datetime | No | The job's closed date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the job (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the job (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the job (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the job (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the job (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The job's contact first name. |  |
| `contact_last_name` | string | No | The job's contact last name. |  |
| `street_1` | string | No | The job's location street 1. |  |
| `street_2` | string | No | The job's location street 2. |  |
| `city` | string | No | The job's location city. |  |
| `state_prov` | string | No | The job's location state prov. |  |
| `postal_code` | string | No | The job's location postal code. |  |
| `location_name` | string | No | The job's location name. |  |
| `is_gated` | boolean | No | The job's location is gated flag. |  |
| `gate_instructions` | string | No | The job's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the job (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the job (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the job (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `customer_payment_terms` | string | No | The `header` of attached customer payment term to the job (Note: `header` - [string] the customer payment term's fields concatenated by pattern `{name}`). |  |
| `project` | string | No | The `header` of attached project to the job (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the job (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The job's po number. |  |
| `contract` | string | No | The `header` of attached contract to the job (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The job's note to customer. |  |
| `called_in_by` | string | No | The job's called in by. |  |
| `is_requires_follow_up` | boolean | No | The job's is requires follow up flag. |  |
| `agents` | array[object] | No | The job's agents list. |  |
| `custom_fields` | array[object] | No | The job's custom fields list. |  |
| `pictures` | array[object] | No | The job's pictures list. |  |
| `documents` | array[object] | No | The job's documents list. |  |
| `equipment` | array[object] | No | The job's equipments list. |  |
| `techs_assigned` | array[object] | No | The job's techs assigned list. |  |
| `tasks` | array[object] | No | The job's tasks list. |  |
| `notes` | array[object] | No | The job's notes list. |  |
| `products` | array[object] | No | The job's products list. |  |
| `services` | array[object] | No | The job's services list. |  |
| `other_charges` | array[object] | No | The job's other charges list. |  |
| `labor_charges` | array[object] | No | The job's labor charges list. |  |
| `expenses` | array[object] | No | The job's expenses list. |  |
| `payments` | array[object] | No | The job's payments list. |  |
| `invoices` | array[object] | No | The job's invoices list. |  |
| `signatures` | array[object] | No | The job's signatures list. |  |
| `printable_work_order` | array[object] | No | The job's printable work order list. |  |
| `visits` | array[object] | No | The job's visits list. |  |

**`items[].agents[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

**`items[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`items[].pictures[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`items[].documents[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`items[].equipment[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

**`items[].equipment[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`items[].techs_assigned[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

**`items[].tasks[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

**`items[].notes[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

**`items[].products[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

**`items[].services[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

**`items[].other_charges[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`items[].labor_charges[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | The labor charge's drive time. |  |
| `drive_time_rate` | number (float) | No | The labor charge's drive time rate. |  |
| `drive_time_cost` | number (float) | No | The labor charge's drive time cost. |  |
| `drive_time_start` | string | No | The labor charge's drive time start. |  |
| `drive_time_end` | string | No | The labor charge's drive time end. |  |
| `is_drive_time_billed` | boolean | No | The labor charge's is drive time billed flag. |  |
| `labor_time` | integer | No | The labor charge's labor time. |  |
| `labor_time_rate` | number (float) | No | The labor charge's labor time rate. |  |
| `labor_time_cost` | number (float) | No | The labor charge's labor time cost. |  |
| `labor_time_start` | string | No | The labor charge's labor time start. |  |
| `labor_time_end` | string | No | The labor charge's labor time end. |  |
| `labor_date` | datetime | No | The labor charge's labor date. |  |
| `is_labor_time_billed` | boolean | No | The labor charge's is labor time billed flag. |  |
| `total` | number (float) | No | The labor charge's total. |  |
| `created_at` | datetime | No | The labor charge's created date. |  |
| `updated_at` | datetime | No | The labor charge's updated date. |  |
| `is_status_generated` | boolean | No | The labor charge's is status generated flag. |  |
| `user` | string | No | The `header` of attached user to the labor charge (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `visit_id` | integer | No | The `id` of attached visit to the labor charge (Note: `id` - [integer] the visit's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the labor charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the labor charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`items[].expenses[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | The expense's purchased from. |  |
| `notes` | string | No | The expense's notes. |  |
| `amount` | number (float) | No | The expense's amount. |  |
| `is_billable` | boolean | No | The expense's is billable flag. |  |
| `date` | datetime | No | The expense's date. |  |
| `created_at` | datetime | No | The expense's created date. |  |
| `updated_at` | datetime | No | The expense's updated date. |  |
| `user` | string | No | The `header` of attached user to the expense (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | The `header` of attached category to the expense (Note: `header` - [string] the category's fields concatenated by pattern `{category_name}`). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the expense (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the expense (Note: `id` - [integer] the qbd class'es identifier). |  |

**`items[].payments[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

**`items[].invoices[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |

**`items[].signatures[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

**`items[].printable_work_order[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

**`items[].visits[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes_for_techs` | string | No | The visit's notes for techs. |  |
| `time_frame_promised_start` | string | No | The visit's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The visit's time frame promised end. |  |
| `duration` | integer | No | The visit's duration (in seconds). |  |
| `is_text_notified` | boolean | No | The visit's is text notified flag. |  |
| `is_voice_notified` | boolean | No | The visit's is voice notified flag. |  |
| `start_date` | datetime | No | The visit's start date. |  |
| `techs_assigned` | array[object] | No | The visit's techs assigned list. |  |

**`items[].visits[].techs_assigned[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |
| `status` | string | No | The assigned tech's status. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 13,
      "number": "1152157",
      "check_number": "1877",
      "priority": "Normal",
      "description": "This is a test",
      "tech_notes": "You guys know what to do.",
      "completion_notes": "Work is done.",
      "customer_payment_terms": "COD",
      "payment_status": "Unpaid",
      "taxes_fees_total": 193.25,
      "drive_labor_total": 0,
      "billable_expenses_total": 0,
      "total": 193,
      "payments_deposits_total": 0,
      "due_total": 193,
      "cost_total": 0,
      "duration": 3600,
      "time_frame_promised_start": "14:10",
      "time_frame_promised_end": "14:10",
      "start_date": "2015-01-08",
      "end_date": "2016-01-08",
      "created_at": "2014-09-08T20:42:04+00:00",
      "updated_at": "2016-01-07T17:20:36+00:00",
      "closed_at": "2016-01-07T17:20:36+00:00",
      "customer_id": 11,
      "customer_name": "Max Paltsev",
      "parent_customer": "Jerry Wheeler",
      "status": "Cancelled",
      "sub_status": "job1",
      "contact_first_name": "Sam",
      "contact_last_name": "Smith",
      "street_1": "1904 Industrial Blvd",
      "street_2": "103",
      "city": "Colleyville",
      "state_prov": "Texas",
      "postal_code": "76034",
      "location_name": "Office",
      "is_gated": false,
      "gate_instructions": null,
      "category": "Quick Home Energy Check-ups",
      "source": "Yellow Pages",
      "payment_type": "Direct Bill",
      "project": "reshma",
      "phase": "Closeup",
      "po_number": "86305",
      "contract": "Retail Service Contract",
      "note_to_customer": "Sample Note To Customer.",
      "called_in_by": "Sample Called In By",
      "is_requires_follow_up": true,
      "agents": [
        {
          "id": 31,
          "first_name": "Justin",
          "last_name": "Wormell"
        },
        {
          "id": 32,
          "first_name": "John",
          "last_name": "Theowner"
        }
      ],
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value",
          "type": "text",
          "group": "Default",
          "created_at": "2018-10-11T11:52:33+00:00",
          "updated_at": "2018-10-11T11:52:33+00:00",
          "is_required": true
        }
      ],
      "pictures": [
        {
          "name": "1442951633_images.jpeg",
          "file_location": "1442951633_images.jpeg",
          "doc_type": "IMG",
          "comment": null,
          "sort": 2,
          "is_private": false,
          "created_at": "2015-09-22T19:53:53+00:00",
          "updated_at": "2015-09-22T19:53:53+00:00",
          "customer_doc_id": 992
        }
      ],
      "documents": [
        {
          "name": "test1John.pdf",
          "file_location": "1421408539_test1John.pdf",
          "doc_type": "DOC",
          "comment": null,
          "sort": 1,
          "is_private": false,
          "created_at": "2015-01-16T11:42:19+00:00",
          "updated_at": "2018-08-21T08:21:14+00:00",
          "customer_doc_id": 998
        }
      ],
      "equipment": [
        {
          "id": 12,
          "type": "Test Equipment",
          "make": "New Test Manufacturer",
          "model": "TST1231MOD",
          "sku": "SK15432",
          "serial_number": "1231#SRN",
          "location": "Test Location",
          "notes": "Test notes for the Test Equipment",
          "extended_warranty_provider": "Test War Provider",
          "is_extended_warranty": false,
          "extended_warranty_date": "2015-02-17",
          "warranty_date": "2015-01-16",
          "install_date": "2014-12-15",
          "created_at": "2015-01-16T11:31:49+00:00",
          "updated_at": "2015-01-16T11:31:49+00:00",
          "customer_id": 87,
          "customer": "John Theowner",
          "customer_location": "Office",
          "custom_fields": [
            {
              "name": "Text",
              "value": "Example text value",
              "type": "text",
              "group": "Default",
              "created_at": "2018-10-11T11:52:33+00:00",
              "updated_at": "2018-10-11T11:52:33+00:00",
              "is_required": true
            }
          ]
        }
      ],
      "techs_assigned": [
        {
          "id": 31,
          "first_name": "Justin",
          "last_name": "Wormell"
        },
        {
          "id": 32,
          "first_name": "John",
          "last_name": "Theowner"
        }
      ],
      "tasks": [
        {
          "type": "Misc",
          "description": "x",
          "start_time": null,
          "start_date": null,
          "end_date": null,
          "is_completed": false,
          "created_at": "2017-03-20T10:48:38+00:00",
          "updated_at": "2017-03-20T10:48:38+00:00"
        }
      ],
      "notes": [
        {
          "notes": "SHOULD BE DELIVERED TO US 6/1/15 AND RICHARD NEEDS TO PAINT",
          "created_at": "2015-05-27T16:32:06+00:00",
          "updated_at": "2015-05-27T16:32:06+00:00"
        }
      ],
      "products": [
        {
          "name": "1755LFB",
          "description": "Finishing Trim Kit - 1\" - Black\r\nModel: \r\nSKU: \r\nType: \r\nPart Number: ",
          "multiplier": 3,
          "rate": 459,
          "total": 1377,
          "cost": 0,
          "actual_cost": 0,
          "item_index": 0,
          "parent_index": 0,
          "created_at": "2015-08-20T09:08:36+00:00",
          "updated_at": "2015-11-19T20:38:07+00:00",
          "is_show_rate_items": true,
          "tax": "City Tax",
          "product": "1755LFB",
          "product_list_id": 45302,
          "warehouse_id": 200,
          "pattern_row_id": null,
          "qbo_class_id": null,
          "qbd_class_id": null
        }
      ],
      "services": [
        {
          "name": "Service Call Fee",
          "description": null,
          "multiplier": 1,
          "rate": 33.15,
          "total": 121,
          "cost": 121,
          "actual_cost": 121,
          "item_index": 3,
          "parent_index": 0,
          "created_at": "2015-08-20T09:08:36+00:00",
          "updated_at": "2015-11-19T20:38:07+00:00",
          "is_show_rate_items": true,
          "tax": "City Tax",
          "service": "Nabeeel",
          "service_list_id": 45302,
          "service_rate_id": 200,
          "pattern_row_id": null,
          "qbo_class_id": null,
          "qbd_class_id": null
        }
      ],
      "other_charges": [
        {
          "name": "fee1",
          "rate": 5.15,
          "total": 14.3,
          "charge_index": 1,
          "parent_index": 1,
          "is_percentage": true,
          "is_discount": false,
          "created_at": "2015-08-20T09:08:52+00:00",
          "updated_at": "2015-11-19T20:38:07+00:00",
          "other_charge": "fee1",
          "applies_to": null,
          "service_list_id": null,
          "other_charge_id": 248,
          "pattern_row_id": null,
          "qbo_class_id": null,
          "qbd_class_id": null
        }
      ],
      "labor_charges": [
        {
          "drive_time": 0,
          "drive_time_rate": 10.25,
          "drive_time_cost": 0,
          "drive_time_start": null,
          "drive_time_end": null,
          "is_drive_time_billed": false,
          "labor_time": 0,
          "labor_time_rate": 11.25,
          "labor_time_cost": 0,
          "labor_time_start": null,
          "labor_time_end": null,
          "labor_date": "2015-11-19",
          "is_labor_time_billed": true,
          "total": 0,
          "created_at": "2015-11-19T20:38:10+00:00",
          "updated_at": "-0001-11-30T00:00:00+00:00",
          "is_status_generated": true,
          "user": "Test qa",
          "visit_id": null,
          "qbo_class_id": null,
          "qbd_class_id": null
        }
      ],
      "expenses": [
        {
          "purchased_from": "test",
          "notes": null,
          "amount": 15.25,
          "is_billable": true,
          "date": "2016-01-19",
          "created_at": "2016-01-07T17:20:36+00:00",
          "updated_at": "-0001-11-30T00:00:00+00:00",
          "user": null,
          "category": "Accounting fees",
          "qbo_class_id": null,
          "qbd_class_id": null
        }
      ],
      "payments": [
        {
          "transaction_type": "AUTH_CAPTURE",
          "transaction_token": "4Tczi4OI12MeoSaC4FG2VPKj1",
          "transaction_id": "257494-0_10",
          "payment_transaction_id": 10,
          "original_transaction_id": 110,
          "apply_to": "JOB",
          "amount": 10.35,
          "memo": null,
          "authorization_code": "755972",
          "bill_to_street_address": "adddad",
          "bill_to_postal_code": "adadadd",
          "bill_to_country": null,
          "reference_number": "1976/1410",
          "is_resync_qbo": false,
          "created_at": "2015-09-25T09:56:57+00:00",
          "updated_at": "2015-09-25T09:56:57+00:00",
          "received_on": "2015-09-25T00:00:00+00:00",
          "qbo_synced_date": "2015-09-25T00:00:00+00:00",
          "qbo_id": 5,
          "qbd_id": "3792-1438659918",
          "customer": "Max Paltsev",
          "type": "Cash",
          "invoice_id": 124,
          "gateway_id": 980190963,
          "receipt_id": "ord-250915-9:56:56"
        }
      ],
      "invoices": [
        {
          "id": 13,
          "number": 1001,
          "currency": "$",
          "po_number": null,
          "terms": "DUR",
          "customer_message": null,
          "notes": null,
          "pay_online_url": "https://app.servicefusion.com/invoiceOnline?id=WP7y6F6Ff48NqjQym4qX1maGXL_1oljugHAP0fNVaBg&key=0DtZ_Q5p4UZNqQHcx08U1k2dx8B3ZHKg3pBxavOtH61",
          "qbo_invoice_no": null,
          "qbo_sync_token": null,
          "qbo_synced_date": "2014-01-21T22:11:31+00:00",
          "qbo_id": null,
          "qbd_id": null,
          "total": 268.32,
          "is_paid": false,
          "date": "2014-01-21T00:00:00+00:00",
          "mail_send_date": null,
          "created_at": "2014-01-21T22:11:31+00:00",
          "updated_at": "2014-01-21T22:11:31+00:00",
          "customer": null,
          "customer_contact": null,
          "payment_terms": "Due Upon Receipt",
          "bill_to_customer_id": null,
          "bill_to_customer_location_id": null,
          "bill_to_customer_contact_id": null,
          "bill_to_email_id": null,
          "bill_to_phone_id": null
        }
      ],
      "signatures": [
        {
          "type": "PREWORK",
          "file_name": "https://servicefusion.s3.amazonaws.com/images/sign/139350-2015-08-25-11-35-14.png",
          "created_at": "2015-08-25T11:35:14+00:00",
          "updated_at": "2015-08-25T11:35:14+00:00"
        }
      ],
      "printable_work_order": [
        {
          "name": "Print With Rates",
          "url": "https://servicefusion.com/printJobWithRates?jobId=fF7HY2Dew1E9vw2mm8FHzSOrpDrKnSl-m2WKf0Yg_Kw"
        }
      ],
      "visits": [
        {
          "notes_for_techs": "Hahahaha",
          "time_frame_promised_start": "00:00",
          "time_frame_promised_end": "00:30",
          "duration": 3600,
          "is_text_notified": false,
          "is_voice_notified": false,
          "start_date": "2018-08-21",
          "techs_assigned": [
            {
              "id": 31,
              "first_name": "Justin",
              "last_name": "Wormell",
              "status": "Started"
            },
            {
              "id": 32,
              "first_name": "John",
              "last_name": "Theowner",
              "status": "Paused"
            }
          ]
        }
      ]
    }
  ],
  "_expandable": [
    "agents",
    "custom_fields",
    "pictures",
    "documents",
    "equipment",
    "equipment.custom_fields",
    "techs_assigned",
    "tasks",
    "notes",
    "products",
    "services",
    "other_charges",
    "labor_charges",
    "expenses",
    "payments",
    "invoices",
    "signatures",
    "printable_work_order",
    "visits",
    "visits.techs_assigned"
  ],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/jobs/{job-id}`

Get a Job by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `job-id` | integer | Yes | Used to send an identifier of the Job to be used. |

##### GET `/jobs/{job-id}`

Get a Job by identifier.

**Traits applied:** `tra.job-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `check_number`, `priority`, `description`, `tech_notes`, `completion_notes`, `payment_status`, `taxes_fees_total`, `drive_labor_total`, `billable_expenses_total`, `total`, `payments_deposits_total`, `due_total`, `cost_total`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `end_date`, `created_at`, `updated_at`, `closed_at`, `customer_id`, `customer_name`, `parent_customer`, `status`, `sub_status`, `contact_first_name`, `contact_last_name`, `street_1`, `street_2`, `city`, `state_prov`, `postal_code`, `location_name`, `is_gated`, `gate_instructions`, `category`, `source`, `payment_type`, `customer_payment_terms`, `project`, `phase`, `po_number`, `contract`, `note_to_customer`, `called_in_by`, `is_requires_follow_up` | `id,number,description` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `agents`, `custom_fields`, `pictures`, `documents`, `equipment`, `equipment.custom_fields`, `techs_assigned`, `tasks`, `notes`, `products`, `services`, `other_charges`, `labor_charges`, `expenses`, `payments`, `invoices`, `signatures`, `printable_work_order`, `visits`, `visits.techs_assigned` | `agents,equipment.custom_fields,visits.techs_assigned` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `JobView`

A job's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job's identifier. |  |
| `number` | string | No | The job's number. |  |
| `check_number` | string | No | The job's check number. |  |
| `priority` | string | No | The job's priority. |  |
| `description` | string | No | The job's description. |  |
| `tech_notes` | string | No | The job's tech notes. |  |
| `completion_notes` | string | No | The job's completion notes. |  |
| `payment_status` | string | No | The job's payment status. |  |
| `taxes_fees_total` | number (float) | No | The job's taxes and fees total. |  |
| `drive_labor_total` | number (float) | No | The job's drive and labor total. |  |
| `billable_expenses_total` | number (float) | No | The job's billable expenses total. |  |
| `total` | number (float) | No | The job's total. |  |
| `payments_deposits_total` | number (float) | No | The job's payments and deposits total. |  |
| `due_total` | number (float) | No | The job's due total. |  |
| `cost_total` | number (float) | No | The job's cost total. |  |
| `duration` | integer | No | The job's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The job's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The job's time frame promised end. |  |
| `start_date` | datetime | No | The job's start date. |  |
| `end_date` | datetime | No | The job's end date. |  |
| `created_at` | datetime | No | The job's created date. |  |
| `updated_at` | datetime | No | The job's updated date. |  |
| `closed_at` | datetime | No | The job's closed date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the job (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the job (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the job (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the job (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the job (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The job's contact first name. |  |
| `contact_last_name` | string | No | The job's contact last name. |  |
| `street_1` | string | No | The job's location street 1. |  |
| `street_2` | string | No | The job's location street 2. |  |
| `city` | string | No | The job's location city. |  |
| `state_prov` | string | No | The job's location state prov. |  |
| `postal_code` | string | No | The job's location postal code. |  |
| `location_name` | string | No | The job's location name. |  |
| `is_gated` | boolean | No | The job's location is gated flag. |  |
| `gate_instructions` | string | No | The job's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the job (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the job (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the job (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `customer_payment_terms` | string | No | The `header` of attached customer payment term to the job (Note: `header` - [string] the customer payment term's fields concatenated by pattern `{name}`). |  |
| `project` | string | No | The `header` of attached project to the job (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the job (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The job's po number. |  |
| `contract` | string | No | The `header` of attached contract to the job (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The job's note to customer. |  |
| `called_in_by` | string | No | The job's called in by. |  |
| `is_requires_follow_up` | boolean | No | The job's is requires follow up flag. |  |
| `agents` | array[object] | No | The job's agents list. |  |
| `custom_fields` | array[object] | No | The job's custom fields list. |  |
| `pictures` | array[object] | No | The job's pictures list. |  |
| `documents` | array[object] | No | The job's documents list. |  |
| `equipment` | array[object] | No | The job's equipments list. |  |
| `techs_assigned` | array[object] | No | The job's techs assigned list. |  |
| `tasks` | array[object] | No | The job's tasks list. |  |
| `notes` | array[object] | No | The job's notes list. |  |
| `products` | array[object] | No | The job's products list. |  |
| `services` | array[object] | No | The job's services list. |  |
| `other_charges` | array[object] | No | The job's other charges list. |  |
| `labor_charges` | array[object] | No | The job's labor charges list. |  |
| `expenses` | array[object] | No | The job's expenses list. |  |
| `payments` | array[object] | No | The job's payments list. |  |
| `invoices` | array[object] | No | The job's invoices list. |  |
| `signatures` | array[object] | No | The job's signatures list. |  |
| `printable_work_order` | array[object] | No | The job's printable work order list. |  |
| `visits` | array[object] | No | The job's visits list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**`agents[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

**`custom_fields[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`pictures[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`documents[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`equipment[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

**`equipment[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`techs_assigned[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

**`tasks[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

**`notes[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

**`products[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

**`services[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

**`other_charges[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`labor_charges[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `drive_time` | integer | No | The labor charge's drive time. |  |
| `drive_time_rate` | number (float) | No | The labor charge's drive time rate. |  |
| `drive_time_cost` | number (float) | No | The labor charge's drive time cost. |  |
| `drive_time_start` | string | No | The labor charge's drive time start. |  |
| `drive_time_end` | string | No | The labor charge's drive time end. |  |
| `is_drive_time_billed` | boolean | No | The labor charge's is drive time billed flag. |  |
| `labor_time` | integer | No | The labor charge's labor time. |  |
| `labor_time_rate` | number (float) | No | The labor charge's labor time rate. |  |
| `labor_time_cost` | number (float) | No | The labor charge's labor time cost. |  |
| `labor_time_start` | string | No | The labor charge's labor time start. |  |
| `labor_time_end` | string | No | The labor charge's labor time end. |  |
| `labor_date` | datetime | No | The labor charge's labor date. |  |
| `is_labor_time_billed` | boolean | No | The labor charge's is labor time billed flag. |  |
| `total` | number (float) | No | The labor charge's total. |  |
| `created_at` | datetime | No | The labor charge's created date. |  |
| `updated_at` | datetime | No | The labor charge's updated date. |  |
| `is_status_generated` | boolean | No | The labor charge's is status generated flag. |  |
| `user` | string | No | The `header` of attached user to the labor charge (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `visit_id` | integer | No | The `id` of attached visit to the labor charge (Note: `id` - [integer] the visit's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the labor charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the labor charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`expenses[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `purchased_from` | string | No | The expense's purchased from. |  |
| `notes` | string | No | The expense's notes. |  |
| `amount` | number (float) | No | The expense's amount. |  |
| `is_billable` | boolean | No | The expense's is billable flag. |  |
| `date` | datetime | No | The expense's date. |  |
| `created_at` | datetime | No | The expense's created date. |  |
| `updated_at` | datetime | No | The expense's updated date. |  |
| `user` | string | No | The `header` of attached user to the expense (Note: `header` - [string] the user's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `category` | string | No | The `header` of attached category to the expense (Note: `header` - [string] the category's fields concatenated by pattern `{category_name}`). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the expense (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the expense (Note: `id` - [integer] the qbd class'es identifier). |  |

**`payments[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

**`invoices[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |

**`signatures[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

**`printable_work_order[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

**`visits[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes_for_techs` | string | No | The visit's notes for techs. |  |
| `time_frame_promised_start` | string | No | The visit's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The visit's time frame promised end. |  |
| `duration` | integer | No | The visit's duration (in seconds). |  |
| `is_text_notified` | boolean | No | The visit's is text notified flag. |  |
| `is_voice_notified` | boolean | No | The visit's is voice notified flag. |  |
| `start_date` | datetime | No | The visit's start date. |  |
| `techs_assigned` | array[object] | No | The visit's techs assigned list. |  |

**`visits[].techs_assigned[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |
| `status` | string | No | The assigned tech's status. |  |

**Example:**
```json
{
  "id": 13,
  "number": "1152157",
  "check_number": "1877",
  "priority": "Normal",
  "description": "This is a test",
  "tech_notes": "You guys know what to do.",
  "completion_notes": "Work is done.",
  "customer_payment_terms": "COD",
  "payment_status": "Unpaid",
  "taxes_fees_total": 193.25,
  "drive_labor_total": 0,
  "billable_expenses_total": 0,
  "total": 193,
  "payments_deposits_total": 0,
  "due_total": 193,
  "cost_total": 0,
  "duration": 3600,
  "time_frame_promised_start": "14:10",
  "time_frame_promised_end": "14:10",
  "start_date": "2015-01-08",
  "end_date": "2016-01-08",
  "created_at": "2014-09-08T20:42:04+00:00",
  "updated_at": "2016-01-07T17:20:36+00:00",
  "closed_at": "2016-01-07T17:20:36+00:00",
  "customer_id": 11,
  "customer_name": "Max Paltsev",
  "parent_customer": "Jerry Wheeler",
  "status": "Cancelled",
  "sub_status": "job1",
  "contact_first_name": "Sam",
  "contact_last_name": "Smith",
  "street_1": "1904 Industrial Blvd",
  "street_2": "103",
  "city": "Colleyville",
  "state_prov": "Texas",
  "postal_code": "76034",
  "location_name": "Office",
  "is_gated": false,
  "gate_instructions": null,
  "category": "Quick Home Energy Check-ups",
  "source": "Yellow Pages",
  "payment_type": "Direct Bill",
  "project": "reshma",
  "phase": "Closeup",
  "po_number": "86305",
  "contract": "Retail Service Contract",
  "note_to_customer": "Sample Note To Customer.",
  "called_in_by": "Sample Called In By",
  "is_requires_follow_up": true,
  "agents": [
    {
      "id": 31,
      "first_name": "Justin",
      "last_name": "Wormell"
    },
    {
      "id": 32,
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value",
      "type": "text",
      "group": "Default",
      "created_at": "2018-10-11T11:52:33+00:00",
      "updated_at": "2018-10-11T11:52:33+00:00",
      "is_required": true
    }
  ],
  "pictures": [
    {
      "name": "1442951633_images.jpeg",
      "file_location": "1442951633_images.jpeg",
      "doc_type": "IMG",
      "comment": null,
      "sort": 2,
      "is_private": false,
      "created_at": "2015-09-22T19:53:53+00:00",
      "updated_at": "2015-09-22T19:53:53+00:00",
      "customer_doc_id": 992
    }
  ],
  "documents": [
    {
      "name": "test1John.pdf",
      "file_location": "1421408539_test1John.pdf",
      "doc_type": "DOC",
      "comment": null,
      "sort": 1,
      "is_private": false,
      "created_at": "2015-01-16T11:42:19+00:00",
      "updated_at": "2018-08-21T08:21:14+00:00",
      "customer_doc_id": 998
    }
  ],
  "equipment": [
    {
      "id": 12,
      "type": "Test Equipment",
      "make": "New Test Manufacturer",
      "model": "TST1231MOD",
      "sku": "SK15432",
      "serial_number": "1231#SRN",
      "location": "Test Location",
      "notes": "Test notes for the Test Equipment",
      "extended_warranty_provider": "Test War Provider",
      "is_extended_warranty": false,
      "extended_warranty_date": "2015-02-17",
      "warranty_date": "2015-01-16",
      "install_date": "2014-12-15",
      "created_at": "2015-01-16T11:31:49+00:00",
      "updated_at": "2015-01-16T11:31:49+00:00",
      "customer_id": 87,
      "customer": "John Theowner",
      "customer_location": "Office",
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value",
          "type": "text",
          "group": "Default",
          "created_at": "2018-10-11T11:52:33+00:00",
          "updated_at": "2018-10-11T11:52:33+00:00",
          "is_required": true
        }
      ]
    }
  ],
  "techs_assigned": [
    {
      "id": 31,
      "first_name": "Justin",
      "last_name": "Wormell"
    },
    {
      "id": 32,
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "tasks": [
    {
      "type": "Misc",
      "description": "x",
      "start_time": null,
      "start_date": null,
      "end_date": null,
      "is_completed": false,
      "created_at": "2017-03-20T10:48:38+00:00",
      "updated_at": "2017-03-20T10:48:38+00:00"
    }
  ],
  "notes": [
    {
      "notes": "SHOULD BE DELIVERED TO US 6/1/15 AND RICHARD NEEDS TO PAINT",
      "created_at": "2015-05-27T16:32:06+00:00",
      "updated_at": "2015-05-27T16:32:06+00:00"
    }
  ],
  "products": [
    {
      "name": "1755LFB",
      "description": "Finishing Trim Kit - 1\" - Black\r\nModel: \r\nSKU: \r\nType: \r\nPart Number: ",
      "multiplier": 3,
      "rate": 459,
      "total": 1377,
      "cost": 0,
      "actual_cost": 0,
      "item_index": 0,
      "parent_index": 0,
      "created_at": "2015-08-20T09:08:36+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "product": "1755LFB",
      "product_list_id": 45302,
      "warehouse_id": 200,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "services": [
    {
      "name": "Service Call Fee",
      "description": null,
      "multiplier": 1,
      "rate": 33.15,
      "total": 121,
      "cost": 121,
      "actual_cost": 121,
      "item_index": 3,
      "parent_index": 0,
      "created_at": "2015-08-20T09:08:36+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "service": "Nabeeel",
      "service_list_id": 45302,
      "service_rate_id": 200,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "other_charges": [
    {
      "name": "fee1",
      "rate": 5.15,
      "total": 14.3,
      "charge_index": 1,
      "parent_index": 1,
      "is_percentage": true,
      "is_discount": false,
      "created_at": "2015-08-20T09:08:52+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "other_charge": "fee1",
      "applies_to": null,
      "service_list_id": null,
      "other_charge_id": 248,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "labor_charges": [
    {
      "drive_time": 0,
      "drive_time_rate": 10.25,
      "drive_time_cost": 0,
      "drive_time_start": null,
      "drive_time_end": null,
      "is_drive_time_billed": false,
      "labor_time": 0,
      "labor_time_rate": 11.25,
      "labor_time_cost": 0,
      "labor_time_start": null,
      "labor_time_end": null,
      "labor_date": "2015-11-19",
      "is_labor_time_billed": true,
      "total": 0,
      "created_at": "2015-11-19T20:38:10+00:00",
      "updated_at": "-0001-11-30T00:00:00+00:00",
      "is_status_generated": true,
      "user": "Test qa",
      "visit_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "expenses": [
    {
      "purchased_from": "test",
      "notes": null,
      "amount": 15.25,
      "is_billable": true,
      "date": "2016-01-19",
      "created_at": "2016-01-07T17:20:36+00:00",
      "updated_at": "-0001-11-30T00:00:00+00:00",
      "user": null,
      "category": "Accounting fees",
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "payments": [
    {
      "transaction_type": "AUTH_CAPTURE",
      "transaction_token": "4Tczi4OI12MeoSaC4FG2VPKj1",
      "transaction_id": "257494-0_10",
      "payment_transaction_id": 10,
      "original_transaction_id": 110,
      "apply_to": "JOB",
      "amount": 10.35,
      "memo": null,
      "authorization_code": "755972",
      "bill_to_street_address": "adddad",
      "bill_to_postal_code": "adadadd",
      "bill_to_country": null,
      "reference_number": "1976/1410",
      "is_resync_qbo": false,
      "created_at": "2015-09-25T09:56:57+00:00",
      "updated_at": "2015-09-25T09:56:57+00:00",
      "received_on": "2015-09-25T00:00:00+00:00",
      "qbo_synced_date": "2015-09-25T00:00:00+00:00",
      "qbo_id": 5,
      "qbd_id": "3792-1438659918",
      "customer": "Max Paltsev",
      "type": "Cash",
      "invoice_id": 124,
      "gateway_id": 980190963,
      "receipt_id": "ord-250915-9:56:56"
    }
  ],
  "invoices": [
    {
      "id": 13,
      "number": 1001,
      "currency": "$",
      "po_number": null,
      "terms": "DUR",
      "customer_message": null,
      "notes": null,
      "pay_online_url": "https://app.servicefusion.com/invoiceOnline?id=WP7y6F6Ff48NqjQym4qX1maGXL_1oljugHAP0fNVaBg&key=0DtZ_Q5p4UZNqQHcx08U1k2dx8B3ZHKg3pBxavOtH61",
      "qbo_invoice_no": null,
      "qbo_sync_token": null,
      "qbo_synced_date": "2014-01-21T22:11:31+00:00",
      "qbo_id": null,
      "qbd_id": null,
      "total": 268.32,
      "is_paid": false,
      "date": "2014-01-21T00:00:00+00:00",
      "mail_send_date": null,
      "created_at": "2014-01-21T22:11:31+00:00",
      "updated_at": "2014-01-21T22:11:31+00:00",
      "customer": null,
      "customer_contact": null,
      "payment_terms": "Due Upon Receipt",
      "bill_to_customer_id": null,
      "bill_to_customer_location_id": null,
      "bill_to_customer_contact_id": null,
      "bill_to_email_id": null,
      "bill_to_phone_id": null
    }
  ],
  "signatures": [
    {
      "type": "PREWORK",
      "file_name": "https://servicefusion.s3.amazonaws.com/images/sign/139350-2015-08-25-11-35-14.png",
      "created_at": "2015-08-25T11:35:14+00:00",
      "updated_at": "2015-08-25T11:35:14+00:00"
    }
  ],
  "printable_work_order": [
    {
      "name": "Print With Rates",
      "url": "https://servicefusion.com/printJobWithRates?jobId=fF7HY2Dew1E9vw2mm8FHzSOrpDrKnSl-m2WKf0Yg_Kw"
    }
  ],
  "visits": [
    {
      "notes_for_techs": "Hahahaha",
      "time_frame_promised_start": "00:00",
      "time_frame_promised_end": "00:30",
      "duration": 3600,
      "is_text_notified": false,
      "is_voice_notified": false,
      "start_date": "2018-08-21",
      "techs_assigned": [
        {
          "id": 31,
          "first_name": "Justin",
          "last_name": "Wormell",
          "status": "Started"
        },
        {
          "id": 32,
          "first_name": "John",
          "last_name": "Theowner",
          "status": "Paused"
        }
      ]
    }
  ],
  "_expandable": [
    "agents",
    "custom_fields",
    "pictures",
    "documents",
    "equipment",
    "equipment.custom_fields",
    "techs_assigned",
    "tasks",
    "notes",
    "products",
    "services",
    "other_charges",
    "labor_charges",
    "expenses",
    "payments",
    "invoices",
    "signatures",
    "printable_work_order",
    "visits",
    "visits.techs_assigned"
  ]
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/job-categories`

**Resource type:** `res.read-only`

#### GET `/job-categories`

List all JobCategories matching query criteria, if provided,
otherwise list all JobCategories.

**Traits applied:** `tra.jobCategory-fieldable`, `tra.jobCategory-sortable`, `tra.jobCategory-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `name` | `id` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `name` | `-id,name` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A job-category's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job category's identifier. |  |
| `name` | string | No | The job category's name. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 490,
      "name": "Job Category for Testing"
    }
  ],
  "_expandable": [],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/job-categories/{job-category-id}`

Get a JobCategory by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `job-category-id` | integer | Yes | Used to send an identifier of the JobCategory to be used. |

##### GET `/job-categories/{job-category-id}`

Get a JobCategory by identifier.

**Traits applied:** `tra.jobCategory-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `name` | `id` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `JobCategoryView`

A job category's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job category's identifier. |  |
| `name` | string | No | The job category's name. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**Example:**
```json
{
  "id": 490,
  "name": "Job Category for Testing",
  "_expandable": []
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/job-statuses`

**Resource type:** `res.read-only`

#### GET `/job-statuses`

List all JobStatuses matching query criteria, if provided,
otherwise list all JobStatuses.

**Traits applied:** `tra.jobStatus-fieldable`, `tra.jobStatus-sortable`, `tra.jobStatus-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `code`, `name`, `is_custom`, `category` | `id,code,is_custom` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `code`, `name`, `is_custom`, `category` | `-id,code` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A job-status's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job statuse's identifier. |  |
| `code` | string | No | The job statuse's code. |  |
| `name` | string | No | The job statuse's name. |  |
| `is_custom` | string | No | The job statuse's is custom flag. |  |
| `category` | string | No | The `header` of attached category to the status (Note: `header` - [string] the category's fields concatenated by pattern `{code}`). |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 1018351032,
      "code": "06_ONS",
      "name": "On Site",
      "is_custom": true,
      "category": "OPEN_ACTIVE"
    }
  ],
  "_expandable": [],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/job-statuses/{job-status-id}`

Get a JobStatus by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `job-status-id` | integer | Yes | Used to send an identifier of the JobStatus to be used. |

##### GET `/job-statuses/{job-status-id}`

Get a JobStatus by identifier.

**Traits applied:** `tra.jobStatus-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `code`, `name`, `is_custom`, `category` | `id,code,is_custom` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `JobStatusView`

A job statuse's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The job statuse's identifier. |  |
| `code` | string | No | The job statuse's code. |  |
| `name` | string | No | The job statuse's name. |  |
| `is_custom` | string | No | The job statuse's is custom flag. |  |
| `category` | string | No | The `header` of attached category to the status (Note: `header` - [string] the category's fields concatenated by pattern `{code}`). |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**Example:**
```json
{
  "id": 1018351032,
  "code": "06_ONS",
  "name": "On Site",
  "is_custom": true,
  "category": "OPEN_ACTIVE",
  "_expandable": []
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/estimates`

**Resource type:** `res.create-read-only`

#### POST `/estimates`

Create a new Estimate.

**Traits applied:** `tra.estimate-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Content-Type` | string | Yes | Used to send a format of data of the request. Enum: `application/json`, `application/x-www-form-urlencoded` |  |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `description`, `tech_notes`, `payment_status`, `taxes_fees_total`, `total`, `due_total`, `cost_total`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `created_at`, `updated_at`, `customer_id`, `customer_name`, `parent_customer`, `status`, `sub_status`, `contact_first_name`, `contact_last_name`, `street_1`, `street_2`, `city`, `state_prov`, `postal_code`, `location_name`, `is_gated`, `gate_instructions`, `category`, `source`, `payment_type`, `customer_payment_terms`, `project`, `phase`, `po_number`, `contract`, `note_to_customer`, `opportunity_rating`, `opportunity_owner` | `id,tech_notes` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `agents`, `custom_fields`, `pictures`, `documents`, `equipment`, `equipment.custom_fields`, `techs_assigned`, `tasks`, `notes`, `products`, `services`, `other_charges`, `payments`, `signatures`, `printable_work_order`, `tags` | `agents,printable_work_order` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Request Body:**

Content-Type: `application/json` (Schema: `EstimateBody`)

An estimate's body schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `description` | string | No | Used to send the estimate's description that will be set. |  |
| `tech_notes` | string | No | Used to send the estimate's tech notes that will be set. |  |
| `duration` | integer | No | Used to send the estimate's duration (in seconds) that will be set. | Min: 0; Max: 86400; Default: 3600 |
| `time_frame_promised_start` | string | No | Used to send the estimate's time frame promised start that will be set. |  |
| `time_frame_promised_end` | string | No | Used to send the estimate's time frame promised end that will be set. |  |
| `start_date` | datetime | No | Used to send the estimate's start date that will be set. |  |
| `created_at` | datetime | No | Used to send the estimate's created date that will be set. | Default: If not passed, it takes the value as current date and time. |
| `customer_name` | string | Yes | Used to send a customer's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the customer's identifier, `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | Used to send a status'es `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the status'es identifier, `header` - [string] the status'es fields concatenated by pattern `{name}`). | Default: If not passed, it takes the default status for estimates. |
| `contact_first_name` | string | No | Used to send the estimate's contact first name that will be set. If a contact with the passed name and surname already exists, then a new contact will not be created, but the existing one will be attached. | Max length: 255; Default: If not passed, it takes the first name from primary contact of the customer (if exists), otherwise a primary contact will be created for the customer. |
| `contact_last_name` | string | No | Used to send the estimate's contact last name that will be set. If a contact with the passed name and surname already exists, then a new contact will not be created, but the existing one will be attached. | Max length: 255; Default: If not passed, it takes the last name from primary contact of the customer (if exists), otherwise a primary contact will be created for the customer. |
| `street_1` | string | No | Used to send the estimate's location street 1 that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `street_2` | string | No | Used to send the estimate's location street 2 that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `city` | string | No | Used to send the estimate's location city that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `state_prov` | string | No | Used to send the estimate's location state prov that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `postal_code` | string | No | Used to send the estimate's location postal code that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `location_name` | string | No | Used to send the estimate's location name that will be set. | Max length: 255; Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `is_gated` | boolean | No | Used to send the estimate's location is gated flag that will be set. | Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `gate_instructions` | string | No | Used to send the estimate's location gate instructions that will be set. | Default: If not passed, it takes the value from a primary location (if any) of passed customer. |
| `category` | string | No | Used to send a category's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the category's identifier, `header` - [string] the category's fields concatenated by pattern `{category}`). Optionally required (configurable into the company preferences). |  |
| `source` | string | No | Used to send a source's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the source's identifier, `header` - [string] the source's fields concatenated by pattern `{short_name}`). | Default: If not passed, it takes the value from the customer. |
| `project` | string | No | Used to send a project's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the project's identifier, `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | Used to send a phase's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the phase's identifier, `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | Used to send the estimate's po number that will be set. | Max length: 255 |
| `contract` | string | No | Used to send a contract's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the contract's identifier, `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). | Default: If not passed, it takes the value from the customer. |
| `note_to_customer` | string | No | Used to send the estimate's note to customer that will be set. | Default: If not passed, it takes the value from the company preferences. |
| `opportunity_rating` | integer | No | Used to send the estimate's opportunity rating that will be set. | Min: 0; Max: 5 |
| `opportunity_owner` | string | No | Used to send an opportunity owner's `id` or `header` that will be attached to the estimate (Note: `id` - [integer] the opportunity owner's identifier, `header` - [string] the opportunity owner's fields concatenated by pattern `{first_name} {last_name}` with space as separator). | Default: If not passed, it takes the value from the authenticated user. |
| `custom_fields` | array[object] | No | Used to send the estimate's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |
| `equipment` | array[object] | No | Used to send the estimate's equipments list that will be set. | Default: array |
| `techs_assigned` | array[object] | No | Used to send the estimate's techs assigned list that will be set. | Default: array |
| `tasks` | array[object] | No | Used to send the estimate's tasks list that will be set. | Default: array |
| `notes` | array[object] | No | Used to send the estimate's notes list that will be set. | Default: array |
| `products` | array[object] | No | Used to send the estimate's products list that will be set. | Default: array |
| `services` | array[object] | No | Used to send the estimate's services list that will be set. | Default: array |
| `other_charges` | array[object] | No | Used to send the estimate's other charges list that will be set. | Default: If not passed, it creates all entries with `auto added` option enabled. Also it creates all not passed other charges declared into `products` and `services`. |
| `tags` | array[object] | No | Used to send the estimate's tags list that will be set. | Default: array |

**`custom_fields[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

**`equipment[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | string | No | Used to send the equipment's identifier that will be searched. You may pass this parameter if you do not want to create new entry but assign existing one. You may assign by `identifier` or `header` (Note: `identifier` - [integer] the equipment's identifier, `header` - [string] the equipment's fields concatenated by pattern `{type}:{make}:{model}:{serial_number}` with colon as separator). | Default: If not passed, it creates new one. |
| `type` | string | No | Used to send the equipment's type that will be set. |  |
| `make` | string | No | Used to send the equipment's make that will be set. |  |
| `model` | string | No | Used to send the equipment's model that will be set. | Max length: 255 |
| `sku` | string | No | Used to send the equipment's sku that will be set. | Max length: 50 |
| `serial_number` | string | No | Used to send the equipment's serial number that will be set. | Max length: 255 |
| `location` | string | No | Used to send the equipment's location that will be set. | Max length: 255 |
| `notes` | string | No | Used to send the equipment's notes that will be set. | Max length: 250 |
| `extended_warranty_provider` | string | No | Used to send the equipment's extended warranty provider that will be set. | Max length: 255 |
| `is_extended_warranty` | boolean | No | Used to send the equipment's is extended warranty flag that will be set. | Default: False |
| `extended_warranty_date` | datetime | No | Used to send the equipment's extended warranty date that will be set. |  |
| `warranty_date` | datetime | No | Used to send the equipment's warranty date that will be set. |  |
| `install_date` | datetime | No | Used to send the equipment's install date that will be set. |  |
| `customer_location` | string | No | Used to send a customer location's `id` or `header` that will be attached to the equipment (Note: `id` - [integer] the customer location's identifier, `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | Used to send the equipment's custom fields list that will be set. | Default: If some custom field (configured into the custom fields settings) not passed, it creates the new one with its default value. |

**`equipment[].custom_fields[]` (nested array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | Yes | Used to send the custom field's name that will be set. |  |
| `value` | any | Yes | Used to send the custom field's value that will be set. |  |

**`techs_assigned[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | Used to send the assigned tech's identifier that will be searched. If this field is set then the entry will be searched by it, otherwise the search will be performed by its full name. | Default: If not passed, it takes the value from full name search entry. |
| `first_name` | string | No | Used to send the assigned tech's first name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |
| `last_name` | string | No | Used to send the assigned tech's last name that will be searched. Required field for full name search. | Default: If field `id` passed, it takes the value from search entry. |

**`tasks[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `description` | string | Yes | Used to send the task's description that will be set. | Max length: 500 |
| `is_completed` | boolean | No | Used to send the task's is completed flag that will be set. | Default: False |

**`notes[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | Yes | Used to send the note's text that will be set. |  |

**`products[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the product's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed product. |
| `description` | string | No | Used to send the product's description that will be set. | Default: If not passed, it takes the value of passed product. |
| `multiplier` | integer | No | Used to send the product's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed product. |
| `rate` | number (float) | No | Used to send the product's rate that will be set. | Default: If not passed, it takes the value of passed product. |
| `cost` | number (float) | No | Used to send the product's cost that will be set. | Default: If not passed, it takes the value of passed product. |
| `is_show_rate_items` | boolean | No | Used to send the product's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the product (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | Yes | Used to send a product's `id` or `header` that will be attached to the product (Note: `id` - [integer] the product's identifier, `header` - [string] the product's fields concatenated by pattern `{make}`). |  |

**`services[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the service's name that will be set. | Max length: 1024; Default: If not passed, it takes the value of passed service. |
| `description` | string | No | Used to send the service's description that will be set. | Default: If not passed, it takes the value of passed service. |
| `multiplier` | integer | No | Used to send the service's quantity that will be set. | Min: 1; Default: If not passed, it takes the value of passed service. |
| `rate` | number (float) | No | Used to send the service's rate that will be set. | Default: If not passed, it takes the value of passed service. |
| `cost` | number (float) | No | Used to send the service's cost that will be set. | Default: If not passed, it takes the value of passed service. |
| `is_show_rate_items` | boolean | No | Used to send the service's is show rate items flag that will be set. | Default: False |
| `tax` | string | No | Used to send a tax'es `id` or `header` that will be attached to the service (Note: `id` - [integer] the tax'es identifier, `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | Yes | Used to send a service's `id` or `header` that will be attached to the service (Note: `id` - [integer] the service's identifier, `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |

**`other_charges[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | Used to send the other charge's name that will be set. | Max length: 255; Default: If not passed, it takes the value of passed other charge. |
| `rate` | number (float) | No | Used to send the other charge's rate that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `is_percentage` | boolean | No | Used to send the other charge's is percentage flag that will be set. | Default: If not passed, it takes the value of passed other charge. |
| `other_charge` | string | Yes | Used to send an other charge's `id` or `header` that will be attached to the other charge (Note: `id` - [integer] the other charge's identifier, `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |

**`tags[]` (array of objects):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | Yes | Used to send the tag's unique tag that will be set. | Max length: 254 |

**Example (0):**
```json
{
  "description": "This is a test",
  "tech_notes": "You guys know what to do.",
  "duration": 3600,
  "time_frame_promised_start": "14:10",
  "time_frame_promised_end": "15:10",
  "start_date": "2015-01-08",
  "created_at": "2014-09-08T20:42:04+00:00",
  "customer_name": "Max Paltsev",
  "status": "Cancelled",
  "contact_first_name": "Sam",
  "contact_last_name": "Smith",
  "street_1": "1904 Industrial Blvd",
  "street_2": "103",
  "city": "Colleyville",
  "state_prov": "Texas",
  "postal_code": "76034",
  "location_name": "Office",
  "is_gated": true,
  "gate_instructions": "Gate instructions for customer",
  "category": "Quick Home Energy Check-ups",
  "source": "Yellow Pages",
  "project": "reshma",
  "phase": "Closeup",
  "po_number": "86305",
  "contract": "Retail Service Contract",
  "note_to_customer": "Sample Note To Customer.",
  "opportunity_rating": 4,
  "opportunity_owner": "John Theowner",
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value"
    },
    {
      "name": "Textarea",
      "value": "Example text area value"
    },
    {
      "name": "Date",
      "value": "2018-10-05"
    },
    {
      "name": "Numeric",
      "value": "157.25"
    },
    {
      "name": "Select",
      "value": "1 one"
    },
    {
      "name": "Checkbox",
      "value": true
    }
  ],
  "equipment": [
    {
      "id": "COIL:ABUS:LMU-2620i:445577998871",
      "type": "Test Equipment",
      "make": "New Test Manufacturer",
      "model": "TST1231MOD",
      "sku": "SK15432",
      "serial_number": "1231#SRN",
      "location": "Test Location",
      "notes": "Test notes for the Test Equipment",
      "extended_warranty_provider": "Test War Provider",
      "is_extended_warranty": false,
      "extended_warranty_date": "2015-02-17",
      "warranty_date": "2015-01-16",
      "install_date": "2014-12-15",
      "customer_location": "Office",
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value"
        },
        {
          "name": "Textarea",
          "value": "Example text area value"
        },
        {
          "name": "Date",
          "value": "2018-10-05"
        },
        {
          "name": "Numeric",
          "value": "157.25"
        },
        {
          "name": "Select",
          "value": "1 one"
        },
        {
          "name": "Checkbox",
          "value": true
        }
      ]
    }
  ],
  "techs_assigned": [
    {
      "id": 31
    },
    {
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "tasks": [
    {
      "description": "x",
      "is_completed": false
    }
  ],
  "notes": [
    {
      "notes": "SHOULD BE DELIVERED TO US 6/1/15 AND RICHARD NEEDS TO PAINT"
    }
  ],
  "products": [
    {
      "name": "1755LFB-NEW",
      "description": "Finishing Trim Kit - 1\" - Black\r\nModel: \r\nSKU: \r\nType: \r\nPart Number: ",
      "multiplier": 2,
      "rate": 500,
      "cost": 100,
      "is_show_rate_items": false,
      "tax": "FIXED",
      "product": "1755LFB"
    }
  ],
  "services": [
    {
      "name": "Service Call Fee",
      "description": null,
      "multiplier": 1,
      "rate": "33.15",
      "cost": "121",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "service": "Nabeeel"
    }
  ],
  "other_charges": [
    {
      "name": "fee1 new",
      "rate": "15.15",
      "is_percentage": false,
      "other_charge": "fee1"
    }
  ],
  "tags": [
    {
      "tag": "Referral"
    }
  ]
}
```

**Responses:**

**`201`** ### 201 Created (Success)
The request has been fulfilled, resulting in the creation of a new resource.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Location` | string | Uri of new resource. | `https://api.servicefusion.com/v1/customers/1472281` |

Body schema: `EstimateView`

An estimate's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The estimate's identifier. |  |
| `number` | string | No | The estimate's number. |  |
| `description` | string | No | The estimate's description. |  |
| `tech_notes` | string | No | The estimate's tech notes. |  |
| `customer_payment_terms` | string | No | The estimate's customer payment terms. |  |
| `payment_status` | string | No | The estimate's payment status. |  |
| `taxes_fees_total` | number (float) | No | The estimate's taxes and fees total. |  |
| `total` | number (float) | No | The estimate's total. |  |
| `due_total` | number (float) | No | The estimate's due total. |  |
| `cost_total` | number (float) | No | The estimate's cost total. |  |
| `duration` | integer | No | The estimate's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The estimate's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The estimate's time frame promised end. |  |
| `start_date` | datetime | No | The estimate's start date. |  |
| `created_at` | datetime | No | The estimate's created date. |  |
| `updated_at` | datetime | No | The estimate's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the estimate (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the estimate (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the estimate (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the estimate (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the estimate (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The estimate's contact first name. |  |
| `contact_last_name` | string | No | The estimate's contact last name. |  |
| `street_1` | string | No | The estimate's location street 1. |  |
| `street_2` | string | No | The estimate's location street 2. |  |
| `city` | string | No | The estimate's location city. |  |
| `state_prov` | string | No | The estimate's location state prov. |  |
| `postal_code` | string | No | The estimate's location postal code. |  |
| `location_name` | string | No | The estimate's location name. |  |
| `is_gated` | boolean | No | The estimate's location is gated flag. |  |
| `gate_instructions` | string | No | The estimate's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the estimate (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the estimate (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the estimate (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `project` | string | No | The `header` of attached project to the estimate (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the estimate (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The estimate's po number. |  |
| `contract` | string | No | The `header` of attached contract to the estimate (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The estimate's note to customer. |  |
| `opportunity_rating` | integer | No | The estimate's opportunity rating. |  |
| `opportunity_owner` | string | No | The `header` of attached opportunity owner to the estimate (Note: `header` - [string] the opportunity owner's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `agents` | array[object] | No | The estimate's agents list. |  |
| `custom_fields` | array[object] | No | The estimate's custom fields list. |  |
| `pictures` | array[object] | No | The estimate's pictures list. |  |
| `documents` | array[object] | No | The estimate's documents list. |  |
| `equipment` | array[object] | No | The estimate's equipments list. |  |
| `techs_assigned` | array[object] | No | The estimate's techs assigned list. |  |
| `tasks` | array[object] | No | The estimate's tasks list. |  |
| `notes` | array[object] | No | The estimate's notes list. |  |
| `products` | array[object] | No | The estimate's products list. |  |
| `services` | array[object] | No | The estimate's services list. |  |
| `other_charges` | array[object] | No | The estimate's other charges list. |  |
| `payments` | array[object] | No | The estimate's payments list. |  |
| `signatures` | array[object] | No | The estimate's signatures list. |  |
| `printable_work_order` | array[object] | No | The estimate's printable work order list. |  |
| `tags` | array[object] | No | The estimate's tags list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**`agents[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

**`custom_fields[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`pictures[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`documents[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`equipment[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

**`equipment[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`techs_assigned[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

**`tasks[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

**`notes[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

**`products[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

**`services[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

**`other_charges[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`payments[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

**`signatures[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

**`printable_work_order[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

**`tags[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | No | The tag's unique tag. |  |
| `created_at` | datetime | No | The tag's created date. |  |
| `updated_at` | datetime | No | The tag's updated date. |  |

**Example:**
```json
{
  "id": 13,
  "number": "1152157",
  "description": "This is a test",
  "tech_notes": "You guys know what to do.",
  "customer_payment_terms": "COD",
  "payment_status": "Unpaid",
  "taxes_fees_total": 193.25,
  "total": 193,
  "due_total": 193,
  "cost_total": 0,
  "duration": 3600,
  "time_frame_promised_start": "14:10",
  "time_frame_promised_end": "14:10",
  "start_date": "2015-01-08",
  "created_at": "2014-09-08T20:42:04+00:00",
  "updated_at": "2016-01-07T17:20:36+00:00",
  "customer_id": 11,
  "customer_name": "Max Paltsev",
  "parent_customer": "Jerry Wheeler",
  "status": "Cancelled",
  "sub_status": "job1",
  "contact_first_name": "Sam",
  "contact_last_name": "Smith",
  "street_1": "1904 Industrial Blvd",
  "street_2": "103",
  "city": "Colleyville",
  "state_prov": "Texas",
  "postal_code": "76034",
  "location_name": "Office",
  "is_gated": false,
  "gate_instructions": null,
  "category": "Quick Home Energy Check-ups",
  "source": "Yellow Pages",
  "payment_type": "Direct Bill",
  "project": "reshma",
  "phase": "Closeup",
  "po_number": "86305",
  "contract": "Retail Service Contract",
  "note_to_customer": "Sample Note To Customer.",
  "opportunity_rating": 4,
  "opportunity_owner": "John Theowner",
  "agents": [
    {
      "id": 31,
      "first_name": "Justin",
      "last_name": "Wormell"
    },
    {
      "id": 32,
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value",
      "type": "text",
      "group": "Default",
      "created_at": "2018-10-11T11:52:33+00:00",
      "updated_at": "2018-10-11T11:52:33+00:00",
      "is_required": true
    }
  ],
  "pictures": [
    {
      "name": "1442951633_images.jpeg",
      "file_location": "1442951633_images.jpeg",
      "doc_type": "IMG",
      "comment": null,
      "sort": 2,
      "is_private": false,
      "created_at": "2015-09-22T19:53:53+00:00",
      "updated_at": "2015-09-22T19:53:53+00:00",
      "customer_doc_id": 992
    }
  ],
  "documents": [
    {
      "name": "test1John.pdf",
      "file_location": "1421408539_test1John.pdf",
      "doc_type": "DOC",
      "comment": null,
      "sort": 1,
      "is_private": false,
      "created_at": "2015-01-16T11:42:19+00:00",
      "updated_at": "2018-08-21T08:21:14+00:00",
      "customer_doc_id": 998
    }
  ],
  "equipment": [
    {
      "id": 12,
      "type": "Test Equipment",
      "make": "New Test Manufacturer",
      "model": "TST1231MOD",
      "sku": "SK15432",
      "serial_number": "1231#SRN",
      "location": "Test Location",
      "notes": "Test notes for the Test Equipment",
      "extended_warranty_provider": "Test War Provider",
      "is_extended_warranty": false,
      "extended_warranty_date": "2015-02-17",
      "warranty_date": "2015-01-16",
      "install_date": "2014-12-15",
      "created_at": "2015-01-16T11:31:49+00:00",
      "updated_at": "2015-01-16T11:31:49+00:00",
      "customer_id": 87,
      "customer": "John Theowner",
      "customer_location": "Office",
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value",
          "type": "text",
          "group": "Default",
          "created_at": "2018-10-11T11:52:33+00:00",
          "updated_at": "2018-10-11T11:52:33+00:00",
          "is_required": true
        }
      ]
    }
  ],
  "techs_assigned": [
    {
      "id": 31,
      "first_name": "Justin",
      "last_name": "Wormell"
    },
    {
      "id": 32,
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "tasks": [
    {
      "type": "Misc",
      "description": "x",
      "start_time": null,
      "start_date": null,
      "end_date": null,
      "is_completed": false,
      "created_at": "2017-03-20T10:48:38+00:00",
      "updated_at": "2017-03-20T10:48:38+00:00"
    }
  ],
  "notes": [
    {
      "notes": "SHOULD BE DELIVERED TO US 6/1/15 AND RICHARD NEEDS TO PAINT",
      "created_at": "2015-05-27T16:32:06+00:00",
      "updated_at": "2015-05-27T16:32:06+00:00"
    }
  ],
  "products": [
    {
      "name": "1755LFB",
      "description": "Finishing Trim Kit - 1\" - Black\r\nModel: \r\nSKU: \r\nType: \r\nPart Number: ",
      "multiplier": 3,
      "rate": 459,
      "total": 1377,
      "cost": 0,
      "actual_cost": 0,
      "item_index": 0,
      "parent_index": 0,
      "created_at": "2015-08-20T09:08:36+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "product": "1755LFB",
      "product_list_id": 45302,
      "warehouse_id": 200,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "services": [
    {
      "name": "Service Call Fee",
      "description": null,
      "multiplier": 1,
      "rate": 33.15,
      "total": 121,
      "cost": 121,
      "actual_cost": 121,
      "item_index": 3,
      "parent_index": 0,
      "created_at": "2015-08-20T09:08:36+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "service": "Nabeeel",
      "service_list_id": 45302,
      "service_rate_id": 200,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "other_charges": [
    {
      "name": "fee1",
      "rate": 5.15,
      "total": 14.3,
      "charge_index": 1,
      "parent_index": 1,
      "is_percentage": true,
      "is_discount": false,
      "created_at": "2015-08-20T09:08:52+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "other_charge": "fee1",
      "applies_to": null,
      "service_list_id": null,
      "other_charge_id": 248,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "payments": [
    {
      "transaction_type": "AUTH_CAPTURE",
      "transaction_token": "4Tczi4OI12MeoSaC4FG2VPKj1",
      "transaction_id": "257494-0_10",
      "payment_transaction_id": 10,
      "original_transaction_id": 110,
      "apply_to": "JOB",
      "amount": 10.35,
      "memo": null,
      "authorization_code": "755972",
      "bill_to_street_address": "adddad",
      "bill_to_postal_code": "adadadd",
      "bill_to_country": null,
      "reference_number": "1976/1410",
      "is_resync_qbo": false,
      "created_at": "2015-09-25T09:56:57+00:00",
      "updated_at": "2015-09-25T09:56:57+00:00",
      "received_on": "2015-09-25T00:00:00+00:00",
      "qbo_synced_date": "2015-09-25T00:00:00+00:00",
      "qbo_id": 5,
      "qbd_id": "3792-1438659918",
      "customer": "Max Paltsev",
      "type": "Cash",
      "invoice_id": 124,
      "gateway_id": 980190963,
      "receipt_id": "ord-250915-9:56:56"
    }
  ],
  "signatures": [
    {
      "type": "PREWORK",
      "file_name": "https://servicefusion.s3.amazonaws.com/images/sign/139350-2015-08-25-11-35-14.png",
      "created_at": "2015-08-25T11:35:14+00:00",
      "updated_at": "2015-08-25T11:35:14+00:00"
    }
  ],
  "printable_work_order": [
    {
      "name": "Print With Rates",
      "url": "https://servicefusion.com/printJobWithRates?jobId=fF7HY2Dew1E9vw2mm8FHzSOrpDrKnSl-m2WKf0Yg_Kw"
    }
  ],
  "tags": [
    {
      "tag": "Referral",
      "created_at": "2017-03-20T10:48:38+00:00",
      "updated_at": "2017-03-20T10:48:38+00:00"
    }
  ],
  "_expandable": [
    "agents",
    "custom_fields",
    "pictures",
    "documents",
    "equipment",
    "equipment.custom_fields",
    "techs_assigned",
    "tasks",
    "notes",
    "products",
    "services",
    "other_charges",
    "payments",
    "signatures",
    "printable_work_order",
    "tags"
  ]
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`422`** ### 422 Unprocessable Entity (Client Error)
The request was well-formed but was unable to be followed due to semantic errors.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `422Error`

Unprocessable entity client's error schema.

**Example:**
```json
[
  {
    "field": "name",
    "message": "Name is too long (maximum is 45 characters)."
  }
]
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### GET `/estimates`

List all Estimates matching query criteria, if provided,
otherwise list all Estimates.

**Traits applied:** `tra.estimate-fieldable`, `tra.estimate-sortable`, `tra.estimate-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `description`, `tech_notes`, `payment_status`, `taxes_fees_total`, `total`, `due_total`, `cost_total`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `created_at`, `updated_at`, `customer_id`, `customer_name`, `parent_customer`, `status`, `sub_status`, `contact_first_name`, `contact_last_name`, `street_1`, `street_2`, `city`, `state_prov`, `postal_code`, `location_name`, `is_gated`, `gate_instructions`, `category`, `source`, `payment_type`, `customer_payment_terms`, `project`, `phase`, `po_number`, `contract`, `note_to_customer`, `opportunity_rating`, `opportunity_owner` | `id,tech_notes` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `agents`, `custom_fields`, `pictures`, `documents`, `equipment`, `equipment.custom_fields`, `techs_assigned`, `tasks`, `notes`, `products`, `services`, `other_charges`, `payments`, `signatures`, `printable_work_order`, `tags` | `agents,printable_work_order` |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `number`, `po_number`, `description`, `tech_notes`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `created_at`, `updated_at`, `customer_id`, `customer_name`, `status`, `sub_status`, `category`, `source`, `payment_type`, `customer_payment_terms`, `contract`, `opportunity_rating` | `number,-start_date` |
| `filters[status]` | string | No | Used to filter results by given statuses (full match). Accepted value is comma-separated string. |  |  | `Estimate Closed, Cancelled` |
| `filters[number]` | string | No | Used to filter results by given number (partial match). |  |  | `101` |
| `filters[po_number]` | string | No | Used to filter results by given po number (partial match). |  |  | `101` |
| `filters[customer_name]` | string | No | Used to filter results by given customer's name (partial match). |  |  | `John Walter` |
| `filters[parent_customer_name]` | string | No | Used to filter results by given parent customer's name (partial match). |  |  | `John Walter` |
| `filters[contact_first_name]` | string | No | Used to filter results by given contact's first name (partial match). |  |  | `John` |
| `filters[contact_last_name]` | string | No | Used to filter results by given contact's last name (partial match). |  |  | `Walter` |
| `filters[address]` | string | No | Used to filter results by given address (partial match). |  |  | `3210 Midway Ave` |
| `filters[city]` | string | No | Used to filter results by given city (full match). |  |  | `Dallas` |
| `filters[zip_code]` | integer | No | Used to filter results by given zip code (full match). |  |  | `75242` |
| `filters[phone]` | string | No | Used to filter results by given phone (partial match). |  |  | `214-555-1212` |
| `filters[email]` | string | No | Used to filter results by given email (full match). |  |  | `john.walter@gmail.com` |
| `filters[category]` | string | No | Used to filter results by given categories (full match). Accepted value is comma-separated string. |  |  | `Install, Service Call` |
| `filters[source]` | string | No | Used to filter results by given sources (full match). Accepted value is comma-separated string. |  |  | `Google, Yelp` |
| `filters[start_date][lte]` | string | No | Used to filter results by given `less than or equal` of start date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[start_date][gte]` | string | No | Used to filter results by given `greater than or equal` of start date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[end_date][lte]` | string | No | Used to filter results by given `less than or equal` of end date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[end_date][gte]` | string | No | Used to filter results by given `greater than or equal` of end date (format: `Y-m-d`). |  |  | `2002-10-02` |
| `filters[requested_date][lte]` | string | No | Used to filter results by given `less than or equal` of requested date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `filters[requested_date][gte]` | string | No | Used to filter results by given `greater than or equal` of requested date (format `RFC 3339`: `Y-m-d\TH:i:sP`). |  |  | `2002-10-02T10:00:00-05:00` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A estimate's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The estimate's identifier. |  |
| `number` | string | No | The estimate's number. |  |
| `description` | string | No | The estimate's description. |  |
| `tech_notes` | string | No | The estimate's tech notes. |  |
| `customer_payment_terms` | string | No | The estimate's customer payment terms. |  |
| `payment_status` | string | No | The estimate's payment status. |  |
| `taxes_fees_total` | number (float) | No | The estimate's taxes and fees total. |  |
| `total` | number (float) | No | The estimate's total. |  |
| `due_total` | number (float) | No | The estimate's due total. |  |
| `cost_total` | number (float) | No | The estimate's cost total. |  |
| `duration` | integer | No | The estimate's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The estimate's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The estimate's time frame promised end. |  |
| `start_date` | datetime | No | The estimate's start date. |  |
| `created_at` | datetime | No | The estimate's created date. |  |
| `updated_at` | datetime | No | The estimate's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the estimate (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the estimate (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the estimate (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the estimate (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the estimate (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The estimate's contact first name. |  |
| `contact_last_name` | string | No | The estimate's contact last name. |  |
| `street_1` | string | No | The estimate's location street 1. |  |
| `street_2` | string | No | The estimate's location street 2. |  |
| `city` | string | No | The estimate's location city. |  |
| `state_prov` | string | No | The estimate's location state prov. |  |
| `postal_code` | string | No | The estimate's location postal code. |  |
| `location_name` | string | No | The estimate's location name. |  |
| `is_gated` | boolean | No | The estimate's location is gated flag. |  |
| `gate_instructions` | string | No | The estimate's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the estimate (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the estimate (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the estimate (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `project` | string | No | The `header` of attached project to the estimate (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the estimate (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The estimate's po number. |  |
| `contract` | string | No | The `header` of attached contract to the estimate (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The estimate's note to customer. |  |
| `opportunity_rating` | integer | No | The estimate's opportunity rating. |  |
| `opportunity_owner` | string | No | The `header` of attached opportunity owner to the estimate (Note: `header` - [string] the opportunity owner's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `agents` | array[object] | No | The estimate's agents list. |  |
| `custom_fields` | array[object] | No | The estimate's custom fields list. |  |
| `pictures` | array[object] | No | The estimate's pictures list. |  |
| `documents` | array[object] | No | The estimate's documents list. |  |
| `equipment` | array[object] | No | The estimate's equipments list. |  |
| `techs_assigned` | array[object] | No | The estimate's techs assigned list. |  |
| `tasks` | array[object] | No | The estimate's tasks list. |  |
| `notes` | array[object] | No | The estimate's notes list. |  |
| `products` | array[object] | No | The estimate's products list. |  |
| `services` | array[object] | No | The estimate's services list. |  |
| `other_charges` | array[object] | No | The estimate's other charges list. |  |
| `payments` | array[object] | No | The estimate's payments list. |  |
| `signatures` | array[object] | No | The estimate's signatures list. |  |
| `printable_work_order` | array[object] | No | The estimate's printable work order list. |  |
| `tags` | array[object] | No | The estimate's tags list. |  |

**`items[].agents[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

**`items[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`items[].pictures[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`items[].documents[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`items[].equipment[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

**`items[].equipment[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`items[].techs_assigned[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

**`items[].tasks[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

**`items[].notes[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

**`items[].products[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

**`items[].services[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

**`items[].other_charges[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`items[].payments[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

**`items[].signatures[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

**`items[].printable_work_order[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

**`items[].tags[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | No | The tag's unique tag. |  |
| `created_at` | datetime | No | The tag's created date. |  |
| `updated_at` | datetime | No | The tag's updated date. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 13,
      "number": "1152157",
      "description": "This is a test",
      "tech_notes": "You guys know what to do.",
      "customer_payment_terms": "COD",
      "payment_status": "Unpaid",
      "taxes_fees_total": 193.25,
      "total": 193,
      "due_total": 193,
      "cost_total": 0,
      "duration": 3600,
      "time_frame_promised_start": "14:10",
      "time_frame_promised_end": "14:10",
      "start_date": "2015-01-08",
      "created_at": "2014-09-08T20:42:04+00:00",
      "updated_at": "2016-01-07T17:20:36+00:00",
      "customer_id": 11,
      "customer_name": "Max Paltsev",
      "parent_customer": "Jerry Wheeler",
      "status": "Cancelled",
      "sub_status": "job1",
      "contact_first_name": "Sam",
      "contact_last_name": "Smith",
      "street_1": "1904 Industrial Blvd",
      "street_2": "103",
      "city": "Colleyville",
      "state_prov": "Texas",
      "postal_code": "76034",
      "location_name": "Office",
      "is_gated": false,
      "gate_instructions": null,
      "category": "Quick Home Energy Check-ups",
      "source": "Yellow Pages",
      "payment_type": "Direct Bill",
      "project": "reshma",
      "phase": "Closeup",
      "po_number": "86305",
      "contract": "Retail Service Contract",
      "note_to_customer": "Sample Note To Customer.",
      "opportunity_rating": 4,
      "opportunity_owner": "John Theowner",
      "agents": [
        {
          "id": 31,
          "first_name": "Justin",
          "last_name": "Wormell"
        },
        {
          "id": 32,
          "first_name": "John",
          "last_name": "Theowner"
        }
      ],
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value",
          "type": "text",
          "group": "Default",
          "created_at": "2018-10-11T11:52:33+00:00",
          "updated_at": "2018-10-11T11:52:33+00:00",
          "is_required": true
        }
      ],
      "pictures": [
        {
          "name": "1442951633_images.jpeg",
          "file_location": "1442951633_images.jpeg",
          "doc_type": "IMG",
          "comment": null,
          "sort": 2,
          "is_private": false,
          "created_at": "2015-09-22T19:53:53+00:00",
          "updated_at": "2015-09-22T19:53:53+00:00",
          "customer_doc_id": 992
        }
      ],
      "documents": [
        {
          "name": "test1John.pdf",
          "file_location": "1421408539_test1John.pdf",
          "doc_type": "DOC",
          "comment": null,
          "sort": 1,
          "is_private": false,
          "created_at": "2015-01-16T11:42:19+00:00",
          "updated_at": "2018-08-21T08:21:14+00:00",
          "customer_doc_id": 998
        }
      ],
      "equipment": [
        {
          "id": 12,
          "type": "Test Equipment",
          "make": "New Test Manufacturer",
          "model": "TST1231MOD",
          "sku": "SK15432",
          "serial_number": "1231#SRN",
          "location": "Test Location",
          "notes": "Test notes for the Test Equipment",
          "extended_warranty_provider": "Test War Provider",
          "is_extended_warranty": false,
          "extended_warranty_date": "2015-02-17",
          "warranty_date": "2015-01-16",
          "install_date": "2014-12-15",
          "created_at": "2015-01-16T11:31:49+00:00",
          "updated_at": "2015-01-16T11:31:49+00:00",
          "customer_id": 87,
          "customer": "John Theowner",
          "customer_location": "Office",
          "custom_fields": [
            {
              "name": "Text",
              "value": "Example text value",
              "type": "text",
              "group": "Default",
              "created_at": "2018-10-11T11:52:33+00:00",
              "updated_at": "2018-10-11T11:52:33+00:00",
              "is_required": true
            }
          ]
        }
      ],
      "techs_assigned": [
        {
          "id": 31,
          "first_name": "Justin",
          "last_name": "Wormell"
        },
        {
          "id": 32,
          "first_name": "John",
          "last_name": "Theowner"
        }
      ],
      "tasks": [
        {
          "type": "Misc",
          "description": "x",
          "start_time": null,
          "start_date": null,
          "end_date": null,
          "is_completed": false,
          "created_at": "2017-03-20T10:48:38+00:00",
          "updated_at": "2017-03-20T10:48:38+00:00"
        }
      ],
      "notes": [
        {
          "notes": "SHOULD BE DELIVERED TO US 6/1/15 AND RICHARD NEEDS TO PAINT",
          "created_at": "2015-05-27T16:32:06+00:00",
          "updated_at": "2015-05-27T16:32:06+00:00"
        }
      ],
      "products": [
        {
          "name": "1755LFB",
          "description": "Finishing Trim Kit - 1\" - Black\r\nModel: \r\nSKU: \r\nType: \r\nPart Number: ",
          "multiplier": 3,
          "rate": 459,
          "total": 1377,
          "cost": 0,
          "actual_cost": 0,
          "item_index": 0,
          "parent_index": 0,
          "created_at": "2015-08-20T09:08:36+00:00",
          "updated_at": "2015-11-19T20:38:07+00:00",
          "is_show_rate_items": true,
          "tax": "City Tax",
          "product": "1755LFB",
          "product_list_id": 45302,
          "warehouse_id": 200,
          "pattern_row_id": null,
          "qbo_class_id": null,
          "qbd_class_id": null
        }
      ],
      "services": [
        {
          "name": "Service Call Fee",
          "description": null,
          "multiplier": 1,
          "rate": 33.15,
          "total": 121,
          "cost": 121,
          "actual_cost": 121,
          "item_index": 3,
          "parent_index": 0,
          "created_at": "2015-08-20T09:08:36+00:00",
          "updated_at": "2015-11-19T20:38:07+00:00",
          "is_show_rate_items": true,
          "tax": "City Tax",
          "service": "Nabeeel",
          "service_list_id": 45302,
          "service_rate_id": 200,
          "pattern_row_id": null,
          "qbo_class_id": null,
          "qbd_class_id": null
        }
      ],
      "other_charges": [
        {
          "name": "fee1",
          "rate": 5.15,
          "total": 14.3,
          "charge_index": 1,
          "parent_index": 1,
          "is_percentage": true,
          "is_discount": false,
          "created_at": "2015-08-20T09:08:52+00:00",
          "updated_at": "2015-11-19T20:38:07+00:00",
          "other_charge": "fee1",
          "applies_to": null,
          "service_list_id": null,
          "other_charge_id": 248,
          "pattern_row_id": null,
          "qbo_class_id": null,
          "qbd_class_id": null
        }
      ],
      "payments": [
        {
          "transaction_type": "AUTH_CAPTURE",
          "transaction_token": "4Tczi4OI12MeoSaC4FG2VPKj1",
          "transaction_id": "257494-0_10",
          "payment_transaction_id": 10,
          "original_transaction_id": 110,
          "apply_to": "JOB",
          "amount": 10.35,
          "memo": null,
          "authorization_code": "755972",
          "bill_to_street_address": "adddad",
          "bill_to_postal_code": "adadadd",
          "bill_to_country": null,
          "reference_number": "1976/1410",
          "is_resync_qbo": false,
          "created_at": "2015-09-25T09:56:57+00:00",
          "updated_at": "2015-09-25T09:56:57+00:00",
          "received_on": "2015-09-25T00:00:00+00:00",
          "qbo_synced_date": "2015-09-25T00:00:00+00:00",
          "qbo_id": 5,
          "qbd_id": "3792-1438659918",
          "customer": "Max Paltsev",
          "type": "Cash",
          "invoice_id": 124,
          "gateway_id": 980190963,
          "receipt_id": "ord-250915-9:56:56"
        }
      ],
      "signatures": [
        {
          "type": "PREWORK",
          "file_name": "https://servicefusion.s3.amazonaws.com/images/sign/139350-2015-08-25-11-35-14.png",
          "created_at": "2015-08-25T11:35:14+00:00",
          "updated_at": "2015-08-25T11:35:14+00:00"
        }
      ],
      "printable_work_order": [
        {
          "name": "Print With Rates",
          "url": "https://servicefusion.com/printJobWithRates?jobId=fF7HY2Dew1E9vw2mm8FHzSOrpDrKnSl-m2WKf0Yg_Kw"
        }
      ],
      "tags": [
        {
          "tag": "Referral",
          "created_at": "2017-03-20T10:48:38+00:00",
          "updated_at": "2017-03-20T10:48:38+00:00"
        }
      ]
    }
  ],
  "_expandable": [
    "agents",
    "custom_fields",
    "pictures",
    "documents",
    "equipment",
    "equipment.custom_fields",
    "techs_assigned",
    "tasks",
    "notes",
    "products",
    "services",
    "other_charges",
    "payments",
    "signatures",
    "printable_work_order",
    "tags"
  ],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/estimates/{estimate-id}`

Get a Estimate by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `estimate-id` | integer | Yes | Used to send an identifier of the Estimate to be used. |

##### GET `/estimates/{estimate-id}`

Get a Estimate by identifier.

**Traits applied:** `tra.estimate-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `description`, `tech_notes`, `payment_status`, `taxes_fees_total`, `total`, `due_total`, `cost_total`, `duration`, `time_frame_promised_start`, `time_frame_promised_end`, `start_date`, `created_at`, `updated_at`, `customer_id`, `customer_name`, `parent_customer`, `status`, `sub_status`, `contact_first_name`, `contact_last_name`, `street_1`, `street_2`, `city`, `state_prov`, `postal_code`, `location_name`, `is_gated`, `gate_instructions`, `category`, `source`, `payment_type`, `customer_payment_terms`, `project`, `phase`, `po_number`, `contract`, `note_to_customer`, `opportunity_rating`, `opportunity_owner` | `id,tech_notes` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. | `agents`, `custom_fields`, `pictures`, `documents`, `equipment`, `equipment.custom_fields`, `techs_assigned`, `tasks`, `notes`, `products`, `services`, `other_charges`, `payments`, `signatures`, `printable_work_order`, `tags` | `agents,printable_work_order` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `EstimateView`

An estimate's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The estimate's identifier. |  |
| `number` | string | No | The estimate's number. |  |
| `description` | string | No | The estimate's description. |  |
| `tech_notes` | string | No | The estimate's tech notes. |  |
| `customer_payment_terms` | string | No | The estimate's customer payment terms. |  |
| `payment_status` | string | No | The estimate's payment status. |  |
| `taxes_fees_total` | number (float) | No | The estimate's taxes and fees total. |  |
| `total` | number (float) | No | The estimate's total. |  |
| `due_total` | number (float) | No | The estimate's due total. |  |
| `cost_total` | number (float) | No | The estimate's cost total. |  |
| `duration` | integer | No | The estimate's duration (in seconds). |  |
| `time_frame_promised_start` | string | No | The estimate's time frame promised start. |  |
| `time_frame_promised_end` | string | No | The estimate's time frame promised end. |  |
| `start_date` | datetime | No | The estimate's start date. |  |
| `created_at` | datetime | No | The estimate's created date. |  |
| `updated_at` | datetime | No | The estimate's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the estimate (Note: `id` - [integer] the customer's identifier). |  |
| `customer_name` | string | No | The `header` of attached customer to the estimate (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `parent_customer` | string | No | The `header` of attached parent customer to the estimate (Note: `header` - [string] the parent customer's fields concatenated by pattern `{customer_name}`). |  |
| `status` | string | No | The `header` of attached status to the estimate (Note: `header` - [string] the status'es fields concatenated by pattern `{name}`). |  |
| `sub_status` | string | No | The `header` of attached sub status to the estimate (Note: `header` - [string] the sub status's fields concatenated by pattern `{name}`). |  |
| `contact_first_name` | string | No | The estimate's contact first name. |  |
| `contact_last_name` | string | No | The estimate's contact last name. |  |
| `street_1` | string | No | The estimate's location street 1. |  |
| `street_2` | string | No | The estimate's location street 2. |  |
| `city` | string | No | The estimate's location city. |  |
| `state_prov` | string | No | The estimate's location state prov. |  |
| `postal_code` | string | No | The estimate's location postal code. |  |
| `location_name` | string | No | The estimate's location name. |  |
| `is_gated` | boolean | No | The estimate's location is gated flag. |  |
| `gate_instructions` | string | No | The estimate's location gate instructions. |  |
| `category` | string | No | The `header` of attached category to the estimate (Note: `header` - [string] the category's fields concatenated by pattern `{category}`). |  |
| `source` | string | No | The `header` of attached source to the estimate (Note: `header` - [string] the source's fields concatenated by pattern `{short_name}`). |  |
| `payment_type` | string | No | The `header` of attached payment type to the estimate (Note: `header` - [string] the payment type's fields concatenated by pattern `{short_name}`). |  |
| `project` | string | No | The `header` of attached project to the estimate (Note: `header` - [string] the project's fields concatenated by pattern `{name}`). |  |
| `phase` | string | No | The `header` of attached phase to the estimate (Note: `header` - [string] the phase's fields concatenated by pattern `{name}`). |  |
| `po_number` | string | No | The estimate's po number. |  |
| `contract` | string | No | The `header` of attached contract to the estimate (Note: `header` - [string] the contract's fields concatenated by pattern `{contract_title}`). |  |
| `note_to_customer` | string | No | The estimate's note to customer. |  |
| `opportunity_rating` | integer | No | The estimate's opportunity rating. |  |
| `opportunity_owner` | string | No | The `header` of attached opportunity owner to the estimate (Note: `header` - [string] the opportunity owner's fields concatenated by pattern `{first_name} {last_name}` with space as separator). |  |
| `agents` | array[object] | No | The estimate's agents list. |  |
| `custom_fields` | array[object] | No | The estimate's custom fields list. |  |
| `pictures` | array[object] | No | The estimate's pictures list. |  |
| `documents` | array[object] | No | The estimate's documents list. |  |
| `equipment` | array[object] | No | The estimate's equipments list. |  |
| `techs_assigned` | array[object] | No | The estimate's techs assigned list. |  |
| `tasks` | array[object] | No | The estimate's tasks list. |  |
| `notes` | array[object] | No | The estimate's notes list. |  |
| `products` | array[object] | No | The estimate's products list. |  |
| `services` | array[object] | No | The estimate's services list. |  |
| `other_charges` | array[object] | No | The estimate's other charges list. |  |
| `payments` | array[object] | No | The estimate's payments list. |  |
| `signatures` | array[object] | No | The estimate's signatures list. |  |
| `printable_work_order` | array[object] | No | The estimate's printable work order list. |  |
| `tags` | array[object] | No | The estimate's tags list. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**`agents[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The agent's identifier. |  |
| `first_name` | string | No | The agent's first name. |  |
| `last_name` | string | No | The agent's last name. |  |

**`custom_fields[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`pictures[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`documents[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The document's name. |  |
| `file_location` | string | No | The document's file location. |  |
| `doc_type` | string | No | The document's type. |  |
| `comment` | string | No | The document's comment. |  |
| `sort` | integer | No | The document's sort. |  |
| `is_private` | boolean | No | The document's is private flag. |  |
| `created_at` | datetime | No | The document's created date. |  |
| `updated_at` | datetime | No | The document's updated date. |  |
| `customer_doc_id` | integer | No | The `id` of attached customer doc to the document (Note: `id` - [integer] the customer doc's identifier). |  |

**`equipment[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The equipment's identifier. |  |
| `type` | string | No | The equipment's type. |  |
| `make` | string | No | The equipment's make. |  |
| `model` | string | No | The equipment's model. |  |
| `sku` | string | No | The equipment's sku. |  |
| `serial_number` | string | No | The equipment's serial number. |  |
| `location` | string | No | The equipment's location. |  |
| `notes` | string | No | The equipment's notes. |  |
| `extended_warranty_provider` | string | No | The equipment's extended warranty provider. |  |
| `is_extended_warranty` | boolean | No | The equipment's is extended warranty flag. |  |
| `extended_warranty_date` | datetime | No | The equipment's extended warranty date. |  |
| `warranty_date` | datetime | No | The equipment's warranty date. |  |
| `install_date` | datetime | No | The equipment's install date. |  |
| `created_at` | datetime | No | The equipment's created date. |  |
| `updated_at` | datetime | No | The equipment's updated date. |  |
| `customer_id` | integer | No | The `id` of attached customer to the equipment (Note: `id` - [integer] the customer's identifier). |  |
| `customer` | string | No | The `header` of attached customer to the equipment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_location` | string | No | The `header` of attached customer location to the equipment (Note: `header` - [string] the customer location's fields concatenated by pattern `{nickname} {street_1} {city}` with space as separator). |  |
| `custom_fields` | array[object] | No | The equipment's custom fields list. |  |

**`equipment[].custom_fields[]` (nested array item):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The custom field's name. |  |
| `value` | any | No | The custom field's value. |  |
| `type` | string | No | The custom field's type. |  |
| `group` | string | No | The custom field's group. |  |
| `created_at` | datetime | No | The custom field's created date. |  |
| `updated_at` | datetime | No | The custom field's updated date. |  |
| `is_required` | boolean | No | The custom field's is required flag. |  |

**`techs_assigned[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The assigned tech's identifier. |  |
| `first_name` | string | No | The assigned tech's first name. |  |
| `last_name` | string | No | The assigned tech's last name. |  |

**`tasks[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The task's type. |  |
| `description` | string | No | The task's description. |  |
| `start_time` | string | No | The task's start time. |  |
| `start_date` | datetime | No | The task's start date. |  |
| `end_date` | datetime | No | The task's end date. |  |
| `is_completed` | boolean | No | The task's is completed flag. |  |
| `created_at` | datetime | No | The task's created date. |  |
| `updated_at` | datetime | No | The task's updated date. |  |

**`notes[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `notes` | string | No | The note's text. |  |
| `created_at` | datetime | No | The note's created date. |  |
| `updated_at` | datetime | No | The note's updated date. |  |

**`products[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The product's name. |  |
| `description` | string | No | The product's description. |  |
| `multiplier` | integer | No | The product's quantity. |  |
| `rate` | number (float) | No | The product's rate. |  |
| `total` | number (float) | No | The product's total. |  |
| `cost` | number (float) | No | The product's cost. |  |
| `actual_cost` | number (float) | No | The product's actual cost. |  |
| `item_index` | integer | No | The product's item index. |  |
| `parent_index` | integer | No | The product's parent index. |  |
| `created_at` | datetime | No | The product's created date. |  |
| `updated_at` | datetime | No | The product's updated date. |  |
| `is_show_rate_items` | boolean | No | The product's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the product (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `product` | string | No | The `header` of attached product to the product (Note: `header` - [string] the product's fields concatenated by pattern `{make}`). |  |
| `product_list_id` | integer | No | The `id` of attached product list to the product (Note: `id` - [integer] the product list's identifier). |  |
| `warehouse_id` | integer | No | The `id` of attached warehouse to the product (Note: `id` - [integer] the warehouse's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the product (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the product (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the product (Note: `id` - [integer] the qbd class'es identifier). |  |

**`services[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The service's name. |  |
| `description` | string | No | The service's description. |  |
| `multiplier` | integer | No | The service's quantity. |  |
| `rate` | number (float) | No | The service's rate. |  |
| `total` | number (float) | No | The service's total. |  |
| `cost` | number (float) | No | The service's cost. |  |
| `actual_cost` | number (float) | No | The service's actual cost. |  |
| `item_index` | integer | No | The service's item index. |  |
| `parent_index` | integer | No | The service's parent index. |  |
| `created_at` | datetime | No | The service's created date. |  |
| `updated_at` | datetime | No | The service's updated date. |  |
| `is_show_rate_items` | boolean | No | The service's is show rate items flag. |  |
| `tax` | string | No | The `header` of attached tax to the service (Note: `header` - [string] the tax'es fields concatenated by pattern `{short_name}`). |  |
| `service` | string | No | The `header` of attached service to the service (Note: `header` - [string] the service's fields concatenated by pattern `{short_description}`). |  |
| `service_list_id` | integer | No | The `id` of attached service list to the service (Note: `id` - [integer] the service list's identifier). |  |
| `service_rate_id` | integer | No | The `id` of attached service rate to the service (Note: `id` - [integer] the service rate's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the service (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the service (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the service (Note: `id` - [integer] the qbd class'es identifier). |  |

**`other_charges[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The other charge's name. |  |
| `rate` | number (float) | No | The other charge's rate. |  |
| `total` | number (float) | No | The other charge's total. |  |
| `charge_index` | integer | No | The other charge's index. |  |
| `parent_index` | integer | No | The other charge's parent index. |  |
| `is_percentage` | boolean | No | The other charge's is percentage flag. |  |
| `is_discount` | boolean | No | The other charge's is discount flag. |  |
| `created_at` | datetime | No | The other charge's created date. |  |
| `updated_at` | datetime | No | The other charge's updated date. |  |
| `other_charge` | string | No | The `header` of attached other charge to the other charge (Note: `header` - [string] the other charge's fields concatenated by pattern `{short_name}`). |  |
| `applies_to` | string | No | The other charge's applies to. |  |
| `service_list_id` | integer | No | The `id` of attached service list to the other charge (Note: `id` - [integer] the service list's identifier). |  |
| `other_charge_id` | integer | No | The `id` of attached other charge to the other charge (Note: `id` - [integer] the other charge's identifier). |  |
| `pattern_row_id` | integer | No | The `id` of attached pattern row to the other charge (Note: `id` - [integer] the pattern row's identifier). |  |
| `qbo_class_id` | integer | No | The `id` of attached qbo class to the other charge (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_class_id` | integer | No | The `id` of attached qbd class to the other charge (Note: `id` - [integer] the qbd class'es identifier). |  |

**`payments[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `transaction_type` | string | No | The payment's transaction type. |  |
| `transaction_token` | string | No | The payment's transaction token. |  |
| `transaction_id` | string | No | The `id` of attached transaction to the payment (Note: `id` - [integer] the transaction's identifier). |  |
| `payment_transaction_id` | integer | No | The `id` of attached payment transaction to the payment (Note: `id` - [integer] the payment transaction's identifier). |  |
| `original_transaction_id` | integer | No | The `id` of attached original transaction to the payment (Note: `id` - [integer] the original transaction's identifier). |  |
| `apply_to` | string | No | The payment's apply to. |  |
| `amount` | number (float) | No | The payment's amount. |  |
| `memo` | string | No | The payment's memo. |  |
| `authorization_code` | string | No | The payment's authorization code. |  |
| `bill_to_street_address` | string | No | The payment's bill to street address. |  |
| `bill_to_postal_code` | string | No | The payment's bill to postal code. |  |
| `bill_to_country` | string | No | The payment's bill to country. |  |
| `reference_number` | string | No | The payment's reference number. |  |
| `is_resync_qbo` | boolean | No | The payment's is resync qbo flag. |  |
| `created_at` | datetime | No | The payment's created date. |  |
| `updated_at` | datetime | No | The payment's updated date. |  |
| `received_on` | datetime | No | The payment's received date. |  |
| `qbo_synced_date` | datetime | No | The payment's qbo synced date. |  |
| `qbo_id` | integer | No | The `id` of attached qbo class to the payment (Note: `id` - [integer] the qbo class'es identifier). |  |
| `qbd_id` | string | No | The `id` of attached qbd class to the payment (Note: `id` - [integer] the qbd class'es identifier). |  |
| `customer` | string | No | The `header` of attached customer to the payment (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `type` | string | No | The `header` of attached customer payment method to the payment (Note: `header` - [string] the customer payment method's fields concatenated by pattern `{cc_type} {first_four} {last_four}` with space as separator). If customer payment method does not attached - it returns the `header` of attached payment type to the job payment (Note: `header` - [string] the payment type's fields concatenated by pattern `{name}`). |  |
| `invoice_id` | integer | No | The `id` of attached invoice to the payment (Note: `id` - [integer] the invoice's identifier). |  |
| `gateway_id` | integer | No | The `id` of attached gateway to the payment (Note: `id` - [integer] the gateway's identifier). |  |
| `receipt_id` | string | No | The `id` of attached receipt to the payment (Note: `id` - [integer] the receipt's identifier). |  |

**`signatures[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `type` | string | No | The signature's type. |  |
| `file_name` | string | No | The signature's file name. |  |
| `created_at` | datetime | No | The signature's created date. |  |
| `updated_at` | datetime | No | The signature's updated date. |  |

**`printable_work_order[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `name` | string | No | The printable work order's name. |  |
| `url` | string | No | The printable work order's url. |  |

**`tags[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `tag` | string | No | The tag's unique tag. |  |
| `created_at` | datetime | No | The tag's created date. |  |
| `updated_at` | datetime | No | The tag's updated date. |  |

**Example:**
```json
{
  "id": 13,
  "number": "1152157",
  "description": "This is a test",
  "tech_notes": "You guys know what to do.",
  "customer_payment_terms": "COD",
  "payment_status": "Unpaid",
  "taxes_fees_total": 193.25,
  "total": 193,
  "due_total": 193,
  "cost_total": 0,
  "duration": 3600,
  "time_frame_promised_start": "14:10",
  "time_frame_promised_end": "14:10",
  "start_date": "2015-01-08",
  "created_at": "2014-09-08T20:42:04+00:00",
  "updated_at": "2016-01-07T17:20:36+00:00",
  "customer_id": 11,
  "customer_name": "Max Paltsev",
  "parent_customer": "Jerry Wheeler",
  "status": "Cancelled",
  "sub_status": "job1",
  "contact_first_name": "Sam",
  "contact_last_name": "Smith",
  "street_1": "1904 Industrial Blvd",
  "street_2": "103",
  "city": "Colleyville",
  "state_prov": "Texas",
  "postal_code": "76034",
  "location_name": "Office",
  "is_gated": false,
  "gate_instructions": null,
  "category": "Quick Home Energy Check-ups",
  "source": "Yellow Pages",
  "payment_type": "Direct Bill",
  "project": "reshma",
  "phase": "Closeup",
  "po_number": "86305",
  "contract": "Retail Service Contract",
  "note_to_customer": "Sample Note To Customer.",
  "opportunity_rating": 4,
  "opportunity_owner": "John Theowner",
  "agents": [
    {
      "id": 31,
      "first_name": "Justin",
      "last_name": "Wormell"
    },
    {
      "id": 32,
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "custom_fields": [
    {
      "name": "Text",
      "value": "Example text value",
      "type": "text",
      "group": "Default",
      "created_at": "2018-10-11T11:52:33+00:00",
      "updated_at": "2018-10-11T11:52:33+00:00",
      "is_required": true
    }
  ],
  "pictures": [
    {
      "name": "1442951633_images.jpeg",
      "file_location": "1442951633_images.jpeg",
      "doc_type": "IMG",
      "comment": null,
      "sort": 2,
      "is_private": false,
      "created_at": "2015-09-22T19:53:53+00:00",
      "updated_at": "2015-09-22T19:53:53+00:00",
      "customer_doc_id": 992
    }
  ],
  "documents": [
    {
      "name": "test1John.pdf",
      "file_location": "1421408539_test1John.pdf",
      "doc_type": "DOC",
      "comment": null,
      "sort": 1,
      "is_private": false,
      "created_at": "2015-01-16T11:42:19+00:00",
      "updated_at": "2018-08-21T08:21:14+00:00",
      "customer_doc_id": 998
    }
  ],
  "equipment": [
    {
      "id": 12,
      "type": "Test Equipment",
      "make": "New Test Manufacturer",
      "model": "TST1231MOD",
      "sku": "SK15432",
      "serial_number": "1231#SRN",
      "location": "Test Location",
      "notes": "Test notes for the Test Equipment",
      "extended_warranty_provider": "Test War Provider",
      "is_extended_warranty": false,
      "extended_warranty_date": "2015-02-17",
      "warranty_date": "2015-01-16",
      "install_date": "2014-12-15",
      "created_at": "2015-01-16T11:31:49+00:00",
      "updated_at": "2015-01-16T11:31:49+00:00",
      "customer_id": 87,
      "customer": "John Theowner",
      "customer_location": "Office",
      "custom_fields": [
        {
          "name": "Text",
          "value": "Example text value",
          "type": "text",
          "group": "Default",
          "created_at": "2018-10-11T11:52:33+00:00",
          "updated_at": "2018-10-11T11:52:33+00:00",
          "is_required": true
        }
      ]
    }
  ],
  "techs_assigned": [
    {
      "id": 31,
      "first_name": "Justin",
      "last_name": "Wormell"
    },
    {
      "id": 32,
      "first_name": "John",
      "last_name": "Theowner"
    }
  ],
  "tasks": [
    {
      "type": "Misc",
      "description": "x",
      "start_time": null,
      "start_date": null,
      "end_date": null,
      "is_completed": false,
      "created_at": "2017-03-20T10:48:38+00:00",
      "updated_at": "2017-03-20T10:48:38+00:00"
    }
  ],
  "notes": [
    {
      "notes": "SHOULD BE DELIVERED TO US 6/1/15 AND RICHARD NEEDS TO PAINT",
      "created_at": "2015-05-27T16:32:06+00:00",
      "updated_at": "2015-05-27T16:32:06+00:00"
    }
  ],
  "products": [
    {
      "name": "1755LFB",
      "description": "Finishing Trim Kit - 1\" - Black\r\nModel: \r\nSKU: \r\nType: \r\nPart Number: ",
      "multiplier": 3,
      "rate": 459,
      "total": 1377,
      "cost": 0,
      "actual_cost": 0,
      "item_index": 0,
      "parent_index": 0,
      "created_at": "2015-08-20T09:08:36+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "product": "1755LFB",
      "product_list_id": 45302,
      "warehouse_id": 200,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "services": [
    {
      "name": "Service Call Fee",
      "description": null,
      "multiplier": 1,
      "rate": 33.15,
      "total": 121,
      "cost": 121,
      "actual_cost": 121,
      "item_index": 3,
      "parent_index": 0,
      "created_at": "2015-08-20T09:08:36+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "is_show_rate_items": true,
      "tax": "City Tax",
      "service": "Nabeeel",
      "service_list_id": 45302,
      "service_rate_id": 200,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "other_charges": [
    {
      "name": "fee1",
      "rate": 5.15,
      "total": 14.3,
      "charge_index": 1,
      "parent_index": 1,
      "is_percentage": true,
      "is_discount": false,
      "created_at": "2015-08-20T09:08:52+00:00",
      "updated_at": "2015-11-19T20:38:07+00:00",
      "other_charge": "fee1",
      "applies_to": null,
      "service_list_id": null,
      "other_charge_id": 248,
      "pattern_row_id": null,
      "qbo_class_id": null,
      "qbd_class_id": null
    }
  ],
  "payments": [
    {
      "transaction_type": "AUTH_CAPTURE",
      "transaction_token": "4Tczi4OI12MeoSaC4FG2VPKj1",
      "transaction_id": "257494-0_10",
      "payment_transaction_id": 10,
      "original_transaction_id": 110,
      "apply_to": "JOB",
      "amount": 10.35,
      "memo": null,
      "authorization_code": "755972",
      "bill_to_street_address": "adddad",
      "bill_to_postal_code": "adadadd",
      "bill_to_country": null,
      "reference_number": "1976/1410",
      "is_resync_qbo": false,
      "created_at": "2015-09-25T09:56:57+00:00",
      "updated_at": "2015-09-25T09:56:57+00:00",
      "received_on": "2015-09-25T00:00:00+00:00",
      "qbo_synced_date": "2015-09-25T00:00:00+00:00",
      "qbo_id": 5,
      "qbd_id": "3792-1438659918",
      "customer": "Max Paltsev",
      "type": "Cash",
      "invoice_id": 124,
      "gateway_id": 980190963,
      "receipt_id": "ord-250915-9:56:56"
    }
  ],
  "signatures": [
    {
      "type": "PREWORK",
      "file_name": "https://servicefusion.s3.amazonaws.com/images/sign/139350-2015-08-25-11-35-14.png",
      "created_at": "2015-08-25T11:35:14+00:00",
      "updated_at": "2015-08-25T11:35:14+00:00"
    }
  ],
  "printable_work_order": [
    {
      "name": "Print With Rates",
      "url": "https://servicefusion.com/printJobWithRates?jobId=fF7HY2Dew1E9vw2mm8FHzSOrpDrKnSl-m2WKf0Yg_Kw"
    }
  ],
  "tags": [
    {
      "tag": "Referral",
      "created_at": "2017-03-20T10:48:38+00:00",
      "updated_at": "2017-03-20T10:48:38+00:00"
    }
  ],
  "_expandable": [
    "agents",
    "custom_fields",
    "pictures",
    "documents",
    "equipment",
    "equipment.custom_fields",
    "techs_assigned",
    "tasks",
    "notes",
    "products",
    "services",
    "other_charges",
    "payments",
    "signatures",
    "printable_work_order",
    "tags"
  ]
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/invoices`

**Resource type:** `res.read-only`

#### GET `/invoices`

List all Invoices matching query criteria, if provided,
otherwise list all Invoices.

**Traits applied:** `tra.invoice-fieldable`, `tra.invoice-sortable`, `tra.invoice-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `currency`, `po_number`, `terms`, `customer_message`, `notes`, `pay_online_url`, `qbo_invoice_no`, `qbo_sync_token`, `qbo_synced_date`, `qbo_id`, `qbd_id`, `total`, `is_paid`, `date`, `mail_send_date`, `created_at`, `updated_at`, `customer`, `customer_contact`, `payment_terms`, `bill_to_customer_id`, `bill_to_customer_location_id`, `bill_to_customer_contact_id`, `bill_to_email_id`, `bill_to_phone_id` | `id,notes` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `number`, `currency`, `po_number`, `terms`, `customer_message`, `notes`, `qbo_invoice_no`, `qbo_sync_token`, `qbo_synced_date`, `qbo_id`, `qbd_id`, `total`, `is_paid`, `date`, `mail_send_date`, `created_at`, `updated_at`, `customer`, `customer_contact`, `payment_terms`, `bill_to_customer_id`, `bill_to_customer_location_id`, `bill_to_customer_contact_id`, `bill_to_email_id`, `bill_to_phone_id` | `created_at,-number` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A invoice's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 13,
      "number": 1001,
      "currency": "$",
      "po_number": null,
      "terms": "DUR",
      "customer_message": null,
      "notes": null,
      "pay_online_url": "https://app.servicefusion.com/invoiceOnline?id=WP7y6F6Ff48NqjQym4qX1maGXL_1oljugHAP0fNVaBg&key=0DtZ_Q5p4UZNqQHcx08U1k2dx8B3ZHKg3pBxavOtH61",
      "qbo_invoice_no": null,
      "qbo_sync_token": null,
      "qbo_synced_date": "2014-01-21T22:11:31+00:00",
      "qbo_id": null,
      "qbd_id": null,
      "total": 268.32,
      "is_paid": false,
      "date": "2014-01-21T00:00:00+00:00",
      "mail_send_date": null,
      "created_at": "2014-01-21T22:11:31+00:00",
      "updated_at": "2014-01-21T22:11:31+00:00",
      "customer": null,
      "customer_contact": null,
      "payment_terms": "Due Upon Receipt",
      "bill_to_customer_id": null,
      "bill_to_customer_location_id": null,
      "bill_to_customer_contact_id": null,
      "bill_to_email_id": null,
      "bill_to_phone_id": null
    }
  ],
  "_expandable": [],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/invoices/{invoice-id}`

Get a Invoice by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `invoice-id` | integer | Yes | Used to send an identifier of the Invoice to be used. |

##### GET `/invoices/{invoice-id}`

Get a Invoice by identifier.

**Traits applied:** `tra.invoice-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `number`, `currency`, `po_number`, `terms`, `customer_message`, `notes`, `pay_online_url`, `qbo_invoice_no`, `qbo_sync_token`, `qbo_synced_date`, `qbo_id`, `qbd_id`, `total`, `is_paid`, `date`, `mail_send_date`, `created_at`, `updated_at`, `customer`, `customer_contact`, `payment_terms`, `bill_to_customer_id`, `bill_to_customer_location_id`, `bill_to_customer_contact_id`, `bill_to_email_id`, `bill_to_phone_id` | `id,notes` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `InvoiceView`

An invoice's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The invoice's identifier. |  |
| `number` | integer | No | The invoice's number. |  |
| `currency` | string | No | The invoice's currency. |  |
| `po_number` | string | No | The invoice's po number. |  |
| `terms` | string | No | The invoice's terms. |  |
| `customer_message` | string | No | The invoice's customer message. |  |
| `notes` | string | No | The invoice's notes. |  |
| `pay_online_url` | string | No | The invoice's pay online url. |  |
| `qbo_invoice_no` | integer | No | The invoice's qbo invoice no. |  |
| `qbo_sync_token` | integer | No | The invoice's qbo sync token. |  |
| `qbo_synced_date` | datetime | No | The invoice's qbo synced date. |  |
| `qbo_id` | integer | No | The invoice's qbo class id. |  |
| `qbd_id` | string | No | The invoice's qbd class id. |  |
| `total` | number (float) | No | The invoice's total. |  |
| `is_paid` | boolean | No | The invoice's is paid flag. |  |
| `date` | datetime | No | The invoice's date. |  |
| `mail_send_date` | datetime | No | The invoice's mail send date. |  |
| `created_at` | datetime | No | The invoice's created date. |  |
| `updated_at` | datetime | No | The invoice's updated date. |  |
| `customer` | string | No | The `header` of attached customer to the invoice (Note: `header` - [string] the customer's fields concatenated by pattern `{customer_name}`). |  |
| `customer_contact` | string | No | The `header` of attached customer contact to the invoice (Note: `header` - [string] the customer contact's fields concatenated by pattern `{fname} {lname}` with space as separator). |  |
| `payment_terms` | string | No | The `header` of attached payment term to the invoice (Note: `header` - [string] the payment term's fields concatenated by pattern `{name}`). |  |
| `bill_to_customer_id` | integer | No | The `id` of attached bill to customer to the invoice (Note: `id` - [integer] the bill to customer's identifier). |  |
| `bill_to_customer_location_id` | integer | No | The `id` of attached bill to customer location to the invoice (Note: `id` - [integer] the bill to customer location's identifier). |  |
| `bill_to_customer_contact_id` | integer | No | The `id` of attached bill to customer contact to the invoice (Note: `id` - [integer] the bill to customer contact's identifier). |  |
| `bill_to_email_id` | integer | No | The `id` of attached bill to email to the invoice (Note: `id` - [integer] the bill to email's identifier). |  |
| `bill_to_phone_id` | integer | No | The `id` of attached bill to phone to the invoice (Note: `id` - [integer] the bill to phone's identifier). |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**Example:**
```json
{
  "id": 13,
  "number": 1001,
  "currency": "$",
  "po_number": null,
  "terms": "DUR",
  "customer_message": null,
  "notes": null,
  "pay_online_url": "https://app.servicefusion.com/invoiceOnline?id=WP7y6F6Ff48NqjQym4qX1maGXL_1oljugHAP0fNVaBg&key=0DtZ_Q5p4UZNqQHcx08U1k2dx8B3ZHKg3pBxavOtH61",
  "qbo_invoice_no": null,
  "qbo_sync_token": null,
  "qbo_synced_date": "2014-01-21T22:11:31+00:00",
  "qbo_id": null,
  "qbd_id": null,
  "total": 268.32,
  "is_paid": false,
  "date": "2014-01-21T00:00:00+00:00",
  "mail_send_date": null,
  "created_at": "2014-01-21T22:11:31+00:00",
  "updated_at": "2014-01-21T22:11:31+00:00",
  "customer": null,
  "customer_contact": null,
  "payment_terms": "Due Upon Receipt",
  "bill_to_customer_id": null,
  "bill_to_customer_location_id": null,
  "bill_to_customer_contact_id": null,
  "bill_to_email_id": null,
  "bill_to_phone_id": null,
  "_expandable": []
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/payment-types`

**Resource type:** `res.read-only`

#### GET `/payment-types`

List all PaymentTypes matching query criteria, if provided,
otherwise list all PaymentTypes.

**Traits applied:** `tra.paymentType-fieldable`, `tra.paymentType-sortable`, `tra.paymentType-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `code`, `short_name`, `type`, `is_custom` | `id,short_name` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `code`, `short_name`, `type`, `is_custom` | `type` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A payment-type's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The type's identifier. |  |
| `code` | string | No | The type's code. |  |
| `short_name` | string | No | The type's short name. |  |
| `type` | string | No | The type's type. |  |
| `is_custom` | boolean | No | The type's is custom flag. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 980190989,
      "code": "BILL",
      "short_name": "Direct Bill",
      "type": "BILL",
      "is_custom": false
    }
  ],
  "_expandable": [],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/payment-types/{payment-type-id}`

Get a PaymentType by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `payment-type-id` | integer | Yes | Used to send an identifier of the PaymentType to be used. |

##### GET `/payment-types/{payment-type-id}`

Get a PaymentType by identifier.

**Traits applied:** `tra.paymentType-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `code`, `short_name`, `type`, `is_custom` | `id,short_name` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `PaymentTypeView`

A payment type's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The type's identifier. |  |
| `code` | string | No | The type's code. |  |
| `short_name` | string | No | The type's short name. |  |
| `type` | string | No | The type's type. |  |
| `is_custom` | boolean | No | The type's is custom flag. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**Example:**
```json
{
  "id": 980190989,
  "code": "BILL",
  "short_name": "Direct Bill",
  "type": "BILL",
  "is_custom": false,
  "_expandable": []
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/sources`

**Resource type:** `res.read-only`

#### GET `/sources`

List all Sources matching query criteria, if provided,
otherwise list all Sources.

**Traits applied:** `tra.source-fieldable`, `tra.source-sortable`, `tra.source-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `short_name`, `long_name` | `id,short_name` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `short_name`, `long_name` | `id,-long_name` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A source's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The source's identifier. |  |
| `short_name` | string | No | The source's short name. |  |
| `long_name` | string | No | The source's long name. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 980192647,
      "short_name": "Source for Testing",
      "long_name": "Long Description of New Testing Source"
    }
  ],
  "_expandable": [],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/sources/{source-id}`

Get a Source by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `source-id` | integer | Yes | Used to send an identifier of the Source to be used. |

##### GET `/sources/{source-id}`

Get a Source by identifier.

**Traits applied:** `tra.source-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `short_name`, `long_name` | `id,short_name` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `SourceView`

A source's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The source's identifier. |  |
| `short_name` | string | No | The source's short name. |  |
| `long_name` | string | No | The source's long name. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**Example:**
```json
{
  "id": 980192647,
  "short_name": "Source for Testing",
  "long_name": "Long Description of New Testing Source",
  "_expandable": []
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

### `/techs`

**Resource type:** `res.read-only`

#### GET `/techs`

List all Techs matching query criteria, if provided,
otherwise list all Techs.

**Traits applied:** `tra.tech-fieldable`, `tra.tech-sortable`, `tra.tech-filtrable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `page` | integer | No | Used to send a page number to be displayed. | 1 |  | `2` |
| `per-page` | integer | No | Used to send a number of items displayed per page (min `1`, max `50`). | 10 |  | `20` |
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `first_name`, `last_name`, `nickname_on_workorder`, `nickname_on_dispatch`, `color_code`, `email`, `phone_1`, `phone_2`, `gender`, `department`, `title`, `bio`, `is_phone_1_mobile`, `is_phone_1_visible_to_client`, `is_phone_2_mobile`, `is_phone_2_visible_to_client`, `is_sales_rep`, `is_field_worker`, `created_at`, `updated_at` | `id,created_at,updated_at` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `sort` | string | No | Used to sort the results by given fields. Use minus `-` before field name to sort DESC. Accepted value is comma-separated string. | id | `id`, `first_name`, `last_name`, `nickname_on_workorder`, `nickname_on_dispatch`, `color_code`, `email`, `phone_1`, `phone_2`, `gender`, `department`, `title`, `bio`, `is_phone_1_mobile`, `is_phone_1_visible_to_client`, `is_phone_2_mobile`, `is_phone_2_visible_to_client`, `is_sales_rep`, `is_field_worker`, `created_at`, `updated_at` | `created_at,-first_name` |
| `filters[first_name]` | string | No | Used to filter results by given first name (partial match). |  |  | `Justin` |
| `filters[last_name]` | string | No | Used to filter results by given last name (partial match). |  |  | `Wormell` |
| `filters[email]` | string | No | Used to filter results by given email (partial match). |  |  | `@servicefusion.com` |
| `filters[nickname_on_workorder]` | string | No | Used to filter results by given nickname on workorder (partial match). |  |  | `Workorder Heating` |
| `filters[nickname_on_dispatch]` | string | No | Used to filter results by given nickname on dispatch (partial match). |  |  | `Dispatch Heating` |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `X-Pagination-Total-Count` | integer | Total number of data items. | `995` |
| `X-Pagination-Page-Count` | integer | Total number of pages of data. | `100` |
| `X-Pagination-Current-Page` | integer | Current page number (1-based). | `1` |
| `X-Pagination-Per-Page` | integer | Number of data items in each page. | `10` |

Body schema: `application/json`

A tech's list schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `items` | array[object] | Yes | Collection envelope. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |
| `_meta` | object | Yes | Meta information. |  |

**`items[]` (array item properties):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The tech's identifier. |  |
| `first_name` | string | No | The tech's first name. |  |
| `last_name` | string | No | The tech's last name. |  |
| `nickname_on_workorder` | string | No | The tech's nickname on workorder. |  |
| `nickname_on_dispatch` | string | No | The tech's nickname on dispatch. |  |
| `color_code` | string | No | The tech's color code. |  |
| `email` | string | No | The tech's email. |  |
| `phone_1` | string | No | The tech's phone 1. |  |
| `phone_2` | string | No | The tech's phone 2. |  |
| `gender` | string | No | The tech's gender. |  |
| `department` | string | No | The tech's department. |  |
| `title` | string | No | The tech's title. |  |
| `bio` | string | No | The tech's bio. |  |
| `is_phone_1_mobile` | boolean | No | The tech's is phone 1 mobile flag. |  |
| `is_phone_1_visible_to_client` | boolean | No | The tech's is phone 1 visible to client flag. |  |
| `is_phone_2_mobile` | boolean | No | The tech's is phone 2 mobile flag. |  |
| `is_phone_2_visible_to_client` | boolean | No | The tech's is phone 2 visible to client flag. |  |
| `is_sales_rep` | boolean | No | The tech's is sales rep flag. |  |
| `is_field_worker` | boolean | No | The tech's is field worker flag. |  |
| `created_at` | datetime | No | The tech's created date. |  |
| `updated_at` | datetime | No | The tech's updated date. |  |

**`_meta` (nested object):**

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `totalCount` | integer | No | Total number of data items. |  |
| `pageCount` | integer | No | Total number of pages of data. |  |
| `currentPage` | integer | No | The current page number (1-based). |  |
| `perPage` | integer | No | The number of data items in each page. |  |

**Example:**
```json
{
  "items": [
    {
      "id": 1472289,
      "first_name": "Justin",
      "last_name": "Wormell",
      "nickname_on_workorder": "Workorder Heating",
      "nickname_on_dispatch": "Dispatch Heating",
      "color_code": "#356a9f",
      "email": "justin@servicefusion.com",
      "phone_1": "232-323-123",
      "phone_2": "444-444-4444",
      "gender": "F",
      "department": "Plumbing",
      "title": "Service Tech",
      "bio": "Here is a short bio on the tech that you can include along with your confirmations",
      "is_phone_1_mobile": false,
      "is_phone_1_visible_to_client": false,
      "is_phone_2_mobile": true,
      "is_phone_2_visible_to_client": true,
      "is_sales_rep": false,
      "is_field_worker": true,
      "created_at": "2018-08-07T18:31:28+00:00",
      "updated_at": "2018-08-07T18:31:28+00:00"
    }
  ],
  "_expandable": [],
  "_meta": {
    "totalCount": 50,
    "pageCount": 5,
    "currentPage": 1,
    "perPage": 10
  }
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---

#### `/techs/{tech-id}`

Get a Tech by identifier.

**URI Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `tech-id` | integer | Yes | Used to send an identifier of the Tech to be used. |

##### GET `/techs/{tech-id}`

Get a Tech by identifier.

**Traits applied:** `tra.tech-fieldable`, `tra.formatable`

**Request Headers:**

| Header | Type | Required | Description | Default | Example |
|--------|------|----------|-------------|---------|---------|
| `Accept` | string | No | Used to send a format of data of the response. Do not use together with the `format` query parameter. Enum: `application/json`, `application/xml` | application/json |  |
| `Authorization` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `access_token` query parameter. |  | `Bearer eyJz93a...k4laUWw` |

**Query Parameters:**

| Parameter | Type | Required | Description | Default | Allowed Values | Example |
|-----------|------|----------|-------------|---------|----------------|---------|
| `fields` | string | No | Used to send a list of fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed all available. | `id`, `first_name`, `last_name`, `nickname_on_workorder`, `nickname_on_dispatch`, `color_code`, `email`, `phone_1`, `phone_2`, `gender`, `department`, `title`, `bio`, `is_phone_1_mobile`, `is_phone_1_visible_to_client`, `is_phone_2_mobile`, `is_phone_2_visible_to_client`, `is_sales_rep`, `is_field_worker`, `created_at`, `updated_at` | `id,created_at,updated_at` |
| `expand` | string | No | Used to send a list of extra-fields to be displayed. Accepted value is comma-separated string. | If not passed, will be displayed nothing. |  |  |
| `format` | string | No | Used to send a format of data of the response. Do not use together with the `Accept` header. | json | `json`, `xml` |  |
| `access_token` | string | No | Used to send a valid OAuth 2 access token. Do not use together with the `Authorization` header. |  |  | `eyJz93a...k4laUWw` |

**Responses:**

**`200`** ### 200 OK (Success)
Standard response for successful HTTP requests.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `TechView`

A tech's schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `id` | integer | No | The tech's identifier. |  |
| `first_name` | string | No | The tech's first name. |  |
| `last_name` | string | No | The tech's last name. |  |
| `nickname_on_workorder` | string | No | The tech's nickname on workorder. |  |
| `nickname_on_dispatch` | string | No | The tech's nickname on dispatch. |  |
| `color_code` | string | No | The tech's color code. |  |
| `email` | string | No | The tech's email. |  |
| `phone_1` | string | No | The tech's phone 1. |  |
| `phone_2` | string | No | The tech's phone 2. |  |
| `gender` | string | No | The tech's gender. |  |
| `department` | string | No | The tech's department. |  |
| `title` | string | No | The tech's title. |  |
| `bio` | string | No | The tech's bio. |  |
| `is_phone_1_mobile` | boolean | No | The tech's is phone 1 mobile flag. |  |
| `is_phone_1_visible_to_client` | boolean | No | The tech's is phone 1 visible to client flag. |  |
| `is_phone_2_mobile` | boolean | No | The tech's is phone 2 mobile flag. |  |
| `is_phone_2_visible_to_client` | boolean | No | The tech's is phone 2 visible to client flag. |  |
| `is_sales_rep` | boolean | No | The tech's is sales rep flag. |  |
| `is_field_worker` | boolean | No | The tech's is field worker flag. |  |
| `created_at` | datetime | No | The tech's created date. |  |
| `updated_at` | datetime | No | The tech's updated date. |  |
| `_expandable` | array[string] | Yes | The extra-field's list that are not expanded and can be expanded into objects. |  |

**Example:**
```json
{
  "id": 1472289,
  "first_name": "Justin",
  "last_name": "Wormell",
  "nickname_on_workorder": "Workorder Heating",
  "nickname_on_dispatch": "Dispatch Heating",
  "color_code": "#356a9f",
  "email": "justin@servicefusion.com",
  "phone_1": "232-323-123",
  "phone_2": "444-444-4444",
  "gender": "F",
  "department": "Plumbing",
  "title": "Service Tech",
  "bio": "Here is a short bio on the tech that you can include along with your confirmations",
  "is_phone_1_mobile": false,
  "is_phone_1_visible_to_client": false,
  "is_phone_2_mobile": true,
  "is_phone_2_visible_to_client": true,
  "is_sales_rep": false,
  "is_field_worker": true,
  "created_at": "2018-08-07T18:31:28+00:00",
  "updated_at": "2018-08-07T18:31:28+00:00",
  "_expandable": []
}
```

**`400`** ### 400 Bad Request (Client Error)
The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing).

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `400Error`

Bad request client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 400,
  "name": "Bad Request.",
  "message": "Your request is invalid."
}
```

**`401`** ### 401 Unauthorized (Client Error)
Authentication is required and has failed or has not yet been provided.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `Error`

Unauthorized client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 401,
  "name": "Unauthorized.",
  "message": "Your request was made with invalid credentials."
}
```

**`403`** ### 403 Forbidden (Client Error)
Access to the requested resource is forbidden. The server understood the request, but will not fulfill it.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `Error`

Forbidden client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 403,
  "name": "Forbidden.",
  "message": "Login Required."
}
```

**`404`** ### 404 Not Found (Client Error)
The requested resource could not be found but may be available in the future.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `404Error`

Not found client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 404,
  "name": "Not Found.",
  "message": "Item not found."
}
```

**`405`** ### 405 Method Not Allowed (Client Error)
A request method is not supported for the requested resource. For example, a GET request on a form that requires data to be presented via POST.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |
| `Allow` | string | List of HTTP request methods allowed for this resource separated by a comma. | `GET, POST, HEAD, OPTIONS` |

Body schema: `405Error`

Method not allowed client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 405,
  "name": "Method not allowed.",
  "message": "Method Not Allowed. This url can only handle the following request methods: GET.\n"
}
```

**`415`** ### 415 Unsupported Media Type (Client Error)
The request entity has a media type which the server or resource does not support. For example, the client set request data as `application/xml`, but the server requires that request data use a different format.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `415Error`

Unsupported media type client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 415,
  "name": "Unsupported Media Type.",
  "message": "None of your requested content types is supported."
}
```

**`429`** ### 429 Too Many Requests (Client Error)
The user has sent too many requests in a given amount of time.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |
| `X-Rate-Limit-Limit` | integer | The maximum number of requests that the consumer is permitted to make per minute. | `60` |
| `X-Rate-Limit-Remaining` | integer | The number of requests remaining in the current rate limit window. | `59` |
| `X-Rate-Limit-Reset` | integer | The time at which the current rate limit window resets in UTC epoch seconds. | `0` |

Body schema: `429Error`

Too many requests client's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 429,
  "name": "Too Many Requests.",
  "message": "Rate limit exceeded."
}
```

**`500`** ### 500 Internal Server Error (Server Error)
A generic error message, given when an unexpected condition was encountered and no more specific message is suitable.

Response Headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `Content-Type` | string | Data response format. | `application/json; charset=UTF-8` |

Body schema: `500Error`

Internal server's error schema.

| Field | Type | Required | Description | Constraints |
|-------|------|----------|-------------|-------------|
| `code` | integer | No | The error code associated with the error. |  |
| `name` | string | No | The error name associated with the error. |  |
| `message` | string | No | The error message associated with the error. |  |

**Example:**
```json
{
  "code": 500,
  "name": "Internal server error.",
  "message": "Failed to create the object for unknown reason."
}
```

---
