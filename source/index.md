---
title: API Reference

language_tabs:
  - Javascript

toc_footers:
  - <a href='#'></a>

includes:
  - errors

search: true
---

# Introduction

The WaterReporter API is a RESTful data programming interface. You can send OPTIONS, GET, POST, PATCH, PUT, and DELETE requests. Remember that some of these methods are protected. In order to access them you'll need to be properly authenticated. Your user account will also need to have the appropriate permissions.

## Your First Request

In the example below we will simple send a request to `https://api.waterreporter.org/v1/`. If you've sent a `GET` request to this endpoint, you should receive a `200` status code response with a Welcome message.

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/ | None

If you are unfamiliar with making HTTP requests please see our [Making Requests](#making-requests) section below.

#### Response

###### Fields

Field   | Description
------- | -----------
code    | The status code response sent with the response
message | Feedback telling you what to do next
status  | A human friendly version of the Status Code

###### Body

````json
{
  "code": 200, 
  "message": "Welcome to the WaterReporterAPI, to learn more about the api visit http://api.waterreporter.org/v1/docs", 
  "status": "200 OK"
}
````

## Making Requests

We like to use a handy little desktop application called [PAW](https://luckymarmot.com/paw?utm_medium=website&utm_campaign=referral_marketing&utm_source=http://github.com/WaterReporter/WaterReporter-API) to make our API development simpler. However, here are a few other ways to access the API.

#### cURL

````json
curl -X "GET" "https://api.waterreporter.org/v1/"
````

#### Python

````python
# Install the Python Requests library:
# `pip install requests`

import requests

def send_request():
    try:
        r = requests.get(
            url="https://api.waterreporter.org/v1/",
        )
        print('Response HTTP Status Code : {status_code}'.format(status_code=r.status_code))
        print('Response HTTP Response Body : {content}'.format(content=r.content))
    except requests.exceptions.RequestException as e:
        print('HTTP Request failed')
````

#### jQuery

````javascript
$.ajax({
    url: "https://api.waterreporter.org/v1/",
    type: "GET",
})
.done(function(data, textStatus, jqXHR) {
    console.log("HTTP Request Succeeded: " + jqXHR.status);
    console.log(data);
})
.fail(function(jqXHR, textStatus, errorThrown) {
    console.log("HTTP Request Failed");
})
.always(function() {
    /* ... */
});
````

# Authentication

Authenticating with the WaterReporting API requires a User account. We will cover User registration below.

## New User Registration

#### Request

Method | URL | Authentication
------ | --- | ---
POST    | https://api.waterreporter.org/v1/user/register | None

#### Request Data (JSON)

Name | Description
---- | -----------
email | The email address the user wishes to register with
password | The password selected by the user

#### Example Request

````javascript
$.ajax({
    url: "https://api.waterreporter.org/v1/user/register",
    type: "POST",
    contentType: "application/json",
    data: JSON.stringify({
        "email": "example@waterreporter.org",
        "password": "mypassword"
    })
})
.done(function(data, textStatus, jqXHR) {
    console.log("HTTP Request Succeeded: " + jqXHR.status);
    console.log(data);
})
.fail(function(jqXHR, textStatus, errorThrown) {
    console.log("HTTP Request Failed");
})
.always(function() {
    /* ... */
});
````

## Remote Login

#### Request URL

Method | URL | Authentication
------ | --- | ---
POST    | https://api.waterreporter.org/v1/auth/remote | None

#### Request Parameters

Name | Expected Value | Description
---- | ------- | ---
response_type | "token" | The type of response you would like to get. `token` should be your parameter value
client_id | <string:client_id> | The generated Client ID within the system
redirect_uri | <string:redirect_uri> | The Redirect URI entered when the Client ID was created
scope | "user" | The scope of permissions we are attempting to access
state | "json" | The state we would like our response to be return in

#### Request Data (JSON)

Name | Description
---- | -----------
email | The email address the user registered with
password | The password the user registered with

````javascript
$.ajax({
    url: "https://api.waterreporter.org/v1/oauth/remote",
    type: "POST",
    data: {
        "response_type": "token",
        "client_id": "QG92Aa2ejWqiYW43I08r6lhSyKVnK1gDN2xrrykU",
        "redirect_uri": "http://127.0.0.1:9000/authorize",
        "scope": "user",
        "state": "json",
    },
    contentType: "application/json",
    data: JSON.stringify({
        "email": "example@waterreporter.org",
        "password": "mypassword"
    })
})
.done(function(data, textStatus, jqXHR) {
    console.log("HTTP Request Succeeded: " + jqXHR.status);
    console.log(data);
})
.fail(function(jqXHR, textStatus, errorThrown) {
    console.log("HTTP Request Failed");
})
.always(function() {
    /* ... */
});

````


# Users

## User Feature Structure

Field | Type  | Description
----- | ----- | ------
id | `integer` | A system field containing unique identifier for interacting with this User
confirmed_at | `timestamp` | A system field containing the timestamp on which this User was added to the WaterReporter API
is_active | `boolean` | A system field containing a Boolean value to signify if the User `is` or `is not` active or "allowed to login to the system"

# Reports

Reports are submitted by WaterKeepers, non-profit groups, and citizens. Individual reports hold user, image, comment, and geospatial information.

## Report Feature Structure

Field | Type  | Description
----- | ----- | ------
id | `integer` | A system field containing unique identifier for interacting with this Report
created | `timestamp` | A system field containing the timestamp on which this Report was added to the WaterReporter API 
updated | `timestamp` | A system field containing the last timestamp this Report feature was modified in some way
status | `varchar` | A system field containing the status of the Feature in the system. If not Authentication is provided, only `public` Report objects will be returned in the repsonse. To expand the returned objects to include `private`, `draft`, and `crowdsourced` Report objects you must submit a valid `access_token` or `Authorization` header with your request. The associated User account must also have appropriate permissions.
owner | `integer` | A system field containing the unique identifier of the User, if any that originally submitted the Report
geometry | `geography` | A valid GeoJSON GEOMETRY COLLECTION containing user submitted POINT data
description | `text` | A field containing user submitted comments and hashtags
pollution | `boolean` | A boolean indentifying whether or not the Report should be considered a Pollution report or not

## Report Feature Collection [GET MANY]

Anyone can request the GET MANY endpoint for Report features. The Report features return in the response vary depending on authentication and individual user permissions.

#### User Permissions

User | Response
---- | --------
Anonymous | Access Report features with a `status` of `public`
Citizen | Access Report features with a `report.status` of `public` and any Report feature where `report.owner` matches their `user.id`
WaterKeeeper | Access all Report features regardless of `report.status`

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/report | Optional

#### Response

###### Fields

Field   | Type | Description
------- | ---- | ------
num_results | `integer` | The total number of Report objects returned from the original request
objects | `array` | The individual Report features returned
page | `integer` | The current page
total_pages | `integer` | The total number of available pages containing Report features

###### Body

````json
{
  "num_results": 0, 
  "objects": [
    {...},
    {...}
  ], 
  "page": 1, 
  "total_pages": 0
}
````

## Report Feature [GET SINGLE]

Anyone can request the GET endpoint for a single Report feature. The Report feature return in the response operates in the same way the GET MANY request operates.

#### User Permissions

User | Response
---- | --------
Anonymous | Access Report features with a `status` of `public`
Citizen | Access Report features with a `report.status` of `public` or any Report feature where `report.owner` matches their `user.id`
WaterKeeeper | Access all Report features regardless of `report.status` or `report.owner` field

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/report/<integer:id> | Optional

#### Response

###### Body

````json
{
  "comments": [], 
  "created": null, 
  "description": null, 
  "geometry": null, 
  "id": 0, 
  "owner": null, 
  "pollution": null, 
  "status": null, 
  "updated": null
}
````

## Report Feature [POST]

#### User Permissions

Anyone may POST a report to this endpoint

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/report | Optional

#### Response

###### Body

````json
{
  "comments": [], 
  "created": null, 
  "description": null, 
  "geometry": null, 
  "id": 0, 
  "owner": null, 
  "pollution": null, 
  "status": null, 
  "updated": null
}
````

## Report Feature [PATCH]

Disabled under the current API version.

## Report Feature [PUT]

Disabled under the current API version.

# Organizations

Organizations can be submitted by WaterKeepers, non-profit groups, and other admins.

## Organization Feature Structure

Field | Type  | Description
----- | ----- | ------
id | `integer` | A system field containing unique identifier for interacting with this Organization
created | `timestamp` | A system field containing the timestamp on which this Organization was added to the WaterReporter API 
updated | `timestamp` | A system field containing the last timestamp this Organization feature was modified in some way
status | `varchar` | A system field containing the status of the Feature in the system.
geometry | `geography` | A valid GeoJSON GEOMETRY COLLECTION containing user submitted POINT data
name | `text` | A field containing the name of the Organization
phone | `text` | A field containing the phone number of the Organization
email | `text` | A field containing the email address of the Organization
website | `text` | A field containing the website of the Organization

## Organization Feature Collection [GET MANY]

Anyone can request the GET MANY endpoint for Organization features. The Organization features return in the response vary depending on authentication and individual user permissions.

#### User Permissions

User | Response
---- | --------
Anonymous | Access Organization features with a `status` of `public`
Citizen | Access Organization features with an `organization.status` of `public` and any Organization feature where `organization.users` contains the current Users `user` object
WaterKeeeper | Access all Organization features regardless of `organization.status`

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/organization | Optional

#### Response

###### Fields

Field   | Type | Description
------- | ---- | ------
num_results | `integer` | The total number of Organization objects returned from the original request
objects | `array` | The individual Organization features returned
page | `integer` | The current page
total_pages | `integer` | The total number of available pages containing Organization features

###### Body

````json
{
  "num_results": 0, 
  "objects": [
    {...},
    {...}
  ], 
  "page": 1, 
  "total_pages": 0
}
````


## Organization Feature [GET SINGLE]

Anyone can request the GET endpoint for a single Organization feature. The Organization feature return in the response operates in the same way the GET MANY request operates.

#### User Permissions

User | Response
---- | --------
Anonymous | Access Organization features with a `status` of `public`
Citizen | Access Organization features with an `organization.status` of `public` and any Organization feature where `organization.users` contains the current Users `user` object
WaterKeeeper | Access all Organization features regardless of `organization.status`

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/organization/<integer:id> | Optional

#### Response

###### Body

````json
{
  "created": null, 
  "email": null, 
  "geometry": null, 
  "id": 1, 
  "name": null, 
  "phone": null, 
  "status": null, 
  "updated": null, 
  "users": [
    {
      "active": true, 
      "confirmed_at": null, 
      "email": "email@waterreporter.org", 
      "id": 0, 
      "name": "", 
    }
  ], 
  "website": null
}
````

## Organization Feature [POST]

#### User Permissions

Only Users with the `admin` role may create new Organizations within WaterReporter.

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/organization | Required

#### Response

###### Body

````json
{
  "created": null, 
  "email": null, 
  "geometry": null, 
  "id": 1, 
  "name": null, 
  "phone": null, 
  "status": null, 
  "updated": null, 
  "users": [
    {
      "active": true, 
      "confirmed_at": null, 
      "email": "email@waterreporter.org", 
      "id": 0, 
      "name": "", 
    }
  ], 
  "website": null
}
````

## Report Feature [PATCH]

#### User Permissions

Only Users with the `admin` role may create new Organizations within WaterReporter and are listed in the `organization.users` array may request the PATCH endpoint.

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/organization/<integer:id> | Required

#### Response

###### Body

````json
{
  "created": null, 
  "email": null, 
  "geometry": null, 
  "id": 1, 
  "name": null, 
  "phone": null, 
  "status": null, 
  "updated": null, 
  "users": [
    {
      "active": true, 
      "confirmed_at": null, 
      "email": "email@waterreporter.org", 
      "id": 0, 
      "name": "", 
    }
  ], 
  "website": null
}
````

## Report Feature [PUT]

#### User Permissions

Only Users with the `admin` role may create new Organizations within WaterReporter and are listed in the `organization.users` array may request the PATCH endpoint.

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/organization/<integer:id> | Required

#### Response

###### Body

````json
{
  "created": null, 
  "email": null, 
  "geometry": null, 
  "id": 1, 
  "name": null, 
  "phone": null, 
  "status": null, 
  "updated": null, 
  "users": [
    {
      "active": true, 
      "confirmed_at": null, 
      "email": "email@waterreporter.org", 
      "id": 0, 
      "name": "", 
    }
  ], 
  "website": null
}
````

# Comments

Comments can be submitted by WaterKeepers, non-profit groups, and citizens.

## Comment Feature Structure

Field | Type  | Description
----- | ----- | ------
id | `integer` | A system field containing unique identifier for interacting with this Comment
created | `timestamp` | A system field containing the timestamp on which this Comment was added to the WaterReporter API 
updated | `timestamp` | A system field containing the last timestamp this Comment feature was modified in some way
status | `varchar` | A system field containing the status of the Feature in the system.
geometry | `geography` | A valid GeoJSON GEOMETRY COLLECTION containing user submitted POINT data
owner | `integer` | A system field containing the unique identifier of the User, if any that originally submitted the Report
body | `text` | A field containing the actual comment

## Comment Feature Collection [GET MANY]

Anyone can request the GET MANY endpoint for Comment features. The Comment features return in the response vary depending on authentication and individual user permissions.

#### User Permissions

User | Response
---- | --------
Anonymous | Access Comment features with a `status` of `public`
Citizen | Access Comment features with an `Comment.status` of `public` and any Comment feature where `Comment.owner` contains the current Users `user.id`
WaterKeeeper | Access all Comment features regardless of `Comment.status`

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/comment | Optional

#### Response

###### Fields

Field   | Type | Description
------- | ---- | ------
num_results | `integer` | The total number of Comment objects returned from the original request
objects | `array` | The individual Comment features returned
page | `integer` | The current page
total_pages | `integer` | The total number of available pages containing Comment features

###### Body

````json
{
  "num_results": 0, 
  "objects": [
    {...},
    {...}
  ], 
  "page": 1, 
  "total_pages": 0
}
````

## Comment Feature [GET SINGLE]

Anyone can request the GET endpoint for a single Comment feature. The Comment feature return in the response operates in the same way the GET MANY request operates.

#### User Permissions

User | Response
---- | --------
Anonymous | Access Comment features with a `status` of `public`
Citizen | Access Comment features with an `Comment.status` of `public` and any Comment feature where `Comment.owner` contains the current Users `user.id`
WaterKeeeper | Access all Comment features regardless of `Comment.status`

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/comment/<integer:id> | Optional

#### Response

###### Body

````json
{
  "created": null, 
  "email": null, 
  "geometry": null, 
  "id": 1, 
  "status": null, 
  "body": null, 
  "owner": null
}
````

## Comment Feature [POST]

#### User Permissions

User | Response
---- | --------
Anonymous | 403 Forbidden
Citizen | Citizen's may POST to the Comment endpoint and their `user.id` will be added as the `comment.owner`
WaterKeeeper | Admin's may POST to the Comment endpoint and their `user.id` will be added as the `comment.owner`

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/comment/<integer:id> | Required

#### Response

###### Body

````json
{
  "created": null, 
  "email": null, 
  "geometry": null, 
  "id": 1, 
  "status": null, 
  "body": null, 
  "owner": null
}
````

## Report Feature [PATCH]

#### User Permissions

User | Response
---- | --------
Anonymous | 403 Forbidden
Citizen | Update the Comment feature if current User's `user.id` field is equal to the `comment.owner` field
WaterKeeeper | Update any Comment

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/comment/<integer:id> | Required

#### Response

###### Body

````json
{
  "created": null, 
  "email": null, 
  "geometry": null, 
  "id": 1, 
  "status": null, 
  "body": null, 
  "owner": null
}
````

## Report Feature [PUT]

#### User Permissions

User | Response
---- | --------
Anonymous | 403 Forbidden
Citizen | Update the Comment feature if current User's `user.id` field is equal to the `comment.owner` field
WaterKeeeper | Update any Comment

#### Request

Method | URL | Authentication
------ | --- | ---
GET    | https://api.waterreporter.org/v1/comment/<integer:id> | Required

#### Response

###### Body

````json
{
  "created": null, 
  "email": null, 
  "geometry": null, 
  "id": 1, 
  "status": null, 
  "body": null, 
  "owner": null
}
````