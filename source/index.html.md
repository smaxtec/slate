---
title: smaXtec Integration API Reference

language_tabs:
  - bash
  - python

toc_footers:
  - <a href='https://messenger-staging.smaxtec.com/'>smaXtec Messenger (staging)</a>
  - <a href='https://api-staging.smaxtec.com/integration/v2/'>smaXtec API (staging)</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the smaXtec Integration API! 🐮 You can use our API to access intern API endpoints, which can get information on animals and animal events in our database.

The Integration API offer the following advantages:

- The animals are identified with the eratag/EID and not the smaXtec internal IDs.
- External partners do not need to choose to update or create. They can simply send the current state of all objects and smaXtec will do the synchronisation.

## Endpoints

The *live* `[endpoint]` is `https://api.smaxtec.com/integration/v2` and the *staging* is `https://api-staging.smaxtec.com/integration/v2`.

The *staging* endpoint can be used for **testing**.

## Parameters and Responses

The parameters in the data of the PUT API calls and the response of the GET/PUT API calls are the same. This means that the output can be taken and provided to a PUT methode.

## User Registration

The registration is done via the [smaXtec messenger](https://messenger.smaxtec.com).

If the user is registered over the [smaXtec staging messenger](https://messenger-staging.smaxtec.com) no registration e-mail is sent. You need to contact us so we can activate your user.

## Common Tasks

For all common tasks the [authetication](#authentication) is necessary.

### Get Organisation of User

One user can have multiple organisations. With the logged in user the user can [get all organisations](#get-all-organisations).

### Add or Update an Animal

For adding an animal to an organisation (`organisation_id`) at least the required fields in the animal data (link to animal parameters) need to be filled in. With the [PUT call](#create-update-single-animal) the animal can be added.

For updating an animal the `organisation_id` and the `official_id` of the animal, which should be updated, need to be in the route. Write all changing parameters in the data and use the [animal PUT](#create-update-single-animal) methode. The response of the animal calls can be put in as data to the animal put call. Changes are detected and are applied.

### Add heat, insemination, pregnancy result, dry off, calving confirmation, abort and diagnosis

These occurrences are stored as [events](#events) where insemination, pregnancy result, calving confirmation and abort are the event types. To add an event the event type (`event_type`) and the event time (`event_ts`) need to be provided to the [event PUT methode](#create-events). Regarding of the type additional information can be added which is listed in the parameter list.

The cronical order of an animal life and how the events happen is the following:

`-> heat -> insemination -> pregnancy_result -> dry_off -> calving_confirmation ->`

or

`-> heat -> insemination -> pregnancy_result -> abort ->`

A `diagnosis` can be made and added at any time in the life of the cow.

## Placeholder

Several placeholders are used in this document.

Placeholder        | Description                                                                              | Example
-----------        | -----------                                                                              | -------
[token]            | [Authentication](#authentication) token which can be get over the API                    | yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg
[endpoint]         | Endpoint to the integration API                                                          | https://api-staging.smaxtec.com/integration/v2
[email]            | E-mail adress from the registered user                                                   | user@smaxtec.com
[password]         | Password from the registered user                                                        | super_secret_password
[organisation_id]  | smaXtec intern ID of the organisation. Can be [get](#get-all-organisations) over the API | 123456qwertz
[official_id]      | The official id of the animal. This is in the most cases the eartag of the animal.       | AT111111112
[device_id]        | The id of a device.                                                                      | 0600000001
[integration_name] | The name of the integrating application.                                                 | HerdmanagerXYZ
[language]         | Language of the wanted translated text.                                                  | en
[event_type]       | [Event type](#get-all-events) of an animal event.                                        | HEALTH_102

# Authentication

> Request

```python
import requests

email='user@smaxtec.com'
password='super_secret_password'
integration_name='my-integration-app-name'
endpoint = 'https://api-staging.smaxtec.com/integration/v2'
route = endpoint + '/users/session_token'
headers = {'accept': 'application/json'}
data = {
   "user": email,
   "password": password,
   "integration_name": integration_name
}

r = requests.post(route, headers=headers, json=data)

status_code = r.status_code
authentication = r.json()
```

```bash
curl -X POST "[endpoint]/users/session_token?email=[email]&password=[password]&integration_name=[integration_name]" \
        -H  "accept: application/json"
```

> Response example

```json
{
  "token": "yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg"
}
```


An authorization token is requested over the API. The Response is a token which needs to be in all API requests to the server in a header that looks like the following:

`-H "authorization: Bearer [token]"`

For example:

`-H "authorization: Bearer yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg"`

<aside class="notice">The authorization header is needed in all following calls.</aside>

**HTTP Request**

`POST "[endpoint]/users/session_token?email=[email]&password=[password]&integration_name=[integration_name]"`

# Organisations

## Get All Organisations

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
route = endpoint + '/organisations'
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.get(route, headers=headers)

status_code = r.status_code
all_user_organisations = r.json()
```

```bash
curl -X GET "[endpoint]/organisations" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
[
  {
    "name": "Organisation Name",
    "organisation_id": "123456qwertz",
    "permissions": [
      "read",
      "groups.update",
      "animals.create",
      "animals.update",
      "animals.archive",
      "animals.changetags",
      "animals.changegroup",
      "devices.update",
      "lactations.update",
      "diagnosis.update",
      "heats.update",
      "organisations.update",
      "annotations.update"
    ],
    "role": "admin",
    "timezone": "Europe/Vienna"
  }
]
```

This endpoint retrieves all organisations of the user. For the Integration API calls the `organisation_id` is needed to make changes to the animals of the organsation.

**HTTP Request**

`GET [endpoint]/organisations`

**Response**

Parameter       | Description
---------       | -----------
name            | Name of the Organisation.
organisation_id | ID of the organisation where the animal belongs to.
permissions     | Permission the user has for this organisation.
role            | Role the user has for this organisation.
timezone        | Timezone of the organisation.

## Create An Organisation

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
route = endpoint + '/organisations'
data = {
    "name": "Farm XY",
    "timezone": "Europe/Vienna",
    "account": {
        "name": "Farm XY",
        "vat_number": "ATU99999999",
        "address": {
            "line1": "Farmer Joe",
            "line2": "Streetname 25",
            "line3": "Block 4",
            "country_code": "AT",
            "region": "Steiermark",
            "city": "Graz",
            "postal_code": "8020",
            "address_type": "default"
        }
    }
}
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.post(route, json=data, headers=headers)

status_code = r.status_code
organisation = r.json()
```

```bash
curl -X POST "[endpoint]/integration/v2/organisations" \
        -H   "accept: application/json" \
        -H   "Authorization: bearer [token]" \
        -H   "Content-Type: application/json" \
        -d  "{\"account\": {\"name\": \"Farm XY\", \"vat_number\": \"ATU99999999\", \"address\": {\"line1\": \"Farmer Joe\", \"line2\": \"Streetname 25\", \"line3\": \"Block 4\", \"country_code\": \"AT\", \"region\": \"Steiermark\", \"city\": \"Graz\", \"postal_code\": \"8020\", \"address_type\": \"default\" } }, \"name\": \"Farm XY\", \"timezone\": \"Europe/Vienna\"}"
```

> Response example

```json
{
    "account_id": "5dasdfasdfasdf1234567890",
    "_id": "5dasdfasdfasdf1234567890",
    "devices": [],
    "timezone": "Europe/Vienna",
    "name": "Farm XY",
    "organisation_settings": {
        "algorithms": {
            "105_threshold": 0.5,
            "105_sigmoid_threshold": 0.3,
            "106_threshold": 0.75,
            "104_svm_sensitivity": "normal",
            "704_threshold": 50.0,
            "703_threshold": 60.0,
            "702_threshold": 50.0,
            "101_threshold": 2.0,
            "104_svm_threshold": 0.5,
            "104_threshold": 0.75,
            "estrus_algo": "HeatIndexStable50"
        }
    },
    "metadata": {},
    "features": [],
    "animal_groups": [],
    "created_by": "5aasdfasdfasdf1234567890"
}
```

This endpoint creates an organisation and its billing address. The smaXtec intern organisation_id is returned as `_id`.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/animals"`

**URL Parameters**

Parameter       | Description                                   ||
---------       | -----------                                   | ---
name            | Name of the farm                              | `required`
timezone        | Timezone of the farm                          | `required`
**account**||
name            | Name of the account                           | `required`
vat_number      | Value added tax identification number         |
**address**||
line1           | First line of the address                     | `required`
line2           | Second line of the address                    | `required`
line3           | Third line of the address                     |
country_code    | Country code                                  | `required`
region          | Region                                        |
city            | City                                          |
postal_code     | Postal code / ZIP                             |
address_type    | Type of the adress. (`default` or `primary`)  |


# Animals

## Get All Animals

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
optional_parameters = '?include_archived=false'
route = endpoint + '/organisations/' + organisation_id + '/animals' + optional_parameters
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.get(route, headers=headers)

status_code = r.status_code
all_animals = r.json()
```

```bash
curl -X GET "[endpoint]/organisations/[organisation_id]/animals" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
[
  {
    "_id": "5c480ab3314724e123456789",
    "archived": false,
    "birthday": "2019-01-10T13:04:52+00:00",
    "created_at": "2019-01-23T06:33:24+00:00",
    "current_device_id": "0700000001",
    "display_name": "AT111111112 - susi",
    "group_id": "57920fca506d993asdfghjkl",
    "lactation_status": "Young_Cow",
    "location": "stall",
    "mark": "AT111111112",
    "mixed_race": [{"race": "ANGLER", "percent": 22}, {"race": "FLECKVIEW", "percent": 78}],
    "name": "susi",
    "official_id": "AT111111112",
    "official_id_rule": "AT",
    "organisation_id": "123456qwertz",
    "race": "mixed",
    "tags": ["sold"]
  }
]
```

This endpoint retrieves from a provided `organisation_id` all animals.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/animals"`

**URL Parameters**

Parameter         | Description ||
---------         | ----------- | ---
organisation_id   | ID of the organisation where the animal belongs to. | `required`
include_archived  | Include archived animals. | default: `false`

## Get Single Animal

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/organisations/' + organisation_id + '/animals/' + official_id
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.get(route, headers=headers)

status_code = r.status_code
animal = r.json()
```

```bash
curl -X GET "[endpoint]/organisations/[organisation_id]/animals/[official_id]" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
{
  "_id": "5c480ab3314724e123456789",
  "archived": false,
  "birthday": "2019-01-10T13:04:52+00:00",
  "created_at": "2019-01-23T06:33:24+00:00",
  "current_device_id": "0700000001",
  "display_name": "AT111111112 - susi",
  "group_id": "57920fca506d993asdfghjkl",
  "lactation_status": "Young_Cow",
  "location": "stall",
  "mark": "AT111111112",
  "mixed_race": [{"race": "ANGLER", "percent": 22}, {"race": "FLECKVIEW", "percent": 78}],
  "name": "susi",
  "official_id": "AT111111112",
  "official_id_rule": "AT",
  "organisation_id": "123456qwertz",
  "race": "mixed",
  "tags": ["sold"]
}
```

This endpoint retrieves from a provided `organisation_id` and `official_id` the animal.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/animals/[official_id]"`

**URL Parameters**

Parameter | Description | |
--------- | ----------- | ----
organisation_id | ID of the organisation where the animal belongs to. | `required`
official_id     | The official id of the animal. This is in the most cases the eartag of the animal. | `required`

## Create/Update Single Animal

> Request

```python
import requests
endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/organisations/' + organisation_id + '/animals/' + official_id
data = {
    "name": "Strolcha",
    "group_id": "57920fca506d993asdfgh123",
    "birthday": "2019-01-10T13:04:52+00:00",
    "race": "mixed",
    "location": "stall",
    "_id": "5c480ab3314724e123456789",
    "official_id_rule": "AT",
    "display_name": "AT111111112 - susi",
    "archived": False,
    "created_at": "2019-01-23T06:33:24+00:00",
    "official_id": "AT111111112",
    "lactation_status": "Young_Cow",
    "current_device_id": "0700000001",
    "mark": "AT111111112",
    "tags": ["sold"],
    "mixed_race": [{"race": "ANGLER", "percent": 22}, {"race": "FLECKVIEW", "percent": 78}],
    "organisation_id": "123456qwertz"
}
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.put(route, json=data, headers=headers)

status_code = r.status_code
animal = r.json()
```

```bash
curl -X PUT "[endpoint]/organisations/[organisation_id]/animals/[official_id]" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]" \
        -H  "Content-Type: application/json" \
        -d "{  \"name\": \"Strolcha\",  \"group_id\": \"57920fca506d993asdfgh123\",  \"birthday\": \"2019-01-10T13:04:52+00:00\",  \"race\": \"mixed\",  \"location\": \"stall\",  \"_id\": \"5c480ab3314724e123456789\",  \"official_id_rule\": \"AT\",  \"display_name\": \"AT111111112 - susi\",  \"archived\": false,  \"created_at\": \"2019-01-23T06:33:24+00:00\",  \"official_id\": \"AT111111112\",  \"lactation_status\": \"Young_Cow\",  \"current_device_id\": \"0700000001\",  \"mark\": \"AT111111112\",  \"tags\": [\"sold\"],  \"mixed_race\": [{\"race\": \"ANGLER\", \"percent\": 22}, {\"race\": \"FLECKVIEW\", \"percent\": 78}],,  \"organisation_id\": \"123456qwertz\"}"
```

> Response example

```json
{
  "_id": "5c480ab3314724e123456789",
  "archived": false,
  "birthday": "2019-01-10T13:04:52+00:00",
  "created_at": "2019-01-23T06:33:24+00:00",
  "current_device_id": "0700000001",
  "display_name": "AT111111112 - Strolcha",
  "group_id": "57920fca506d993asdfghjkl",
  "lactation_status": "Young_Cow",
  "location": "stall",
  "mark": "AT111111112",
  "mixed_race": [{"race": "ANGLER", "percent": 22}, {"race": "FLECKVIEW", "percent": 78}],
  "name": "Strolcha",
  "official_id": "AT111111112",
  "official_id_rule": "AT",
  "organisation_id": "123456qwertz",
  "race": "mixed",
  "tags": ["sold"]
}
```

This endpoint creates/updates an animal which is in the organisation with `organisation_id` and the `official_id`. If the animal is created or updated depends. If the animal with the provided `official_id` exists in the organisation, it gets updated. Otherwise the animal gets created. The animal response can be used as a input for thes call and just the parameters that are needed to be updated can be replaced.

**HTTP Request**

`PUT "[endpoint]/organisations/[organisation_id]/animals/[official_id]"`

**URL Parameters**

Parameter         | Description ||
---------         | ----------- | ---
organisation_id   | ID of the organisation where the animal belongs to. | `required`
official_id       | The official id of the animal. This is in the most cases the eartag of the animal. | `required`
||
_id               | Internal ID for the animal. Can't be changed.
archived          | When the animal is archived (inactive) the archived is set to `true`.
birthday          | Day of birth of the animal.
created_at        | The creation time of the animal in the smaXtec system. Can't be changed.
current_device_id | The ID of the device which is currently active in the animal.
display_name      | Composition of the official ID and the name of the animal.
group_id          | ID of the group where the animal is currently in.
lactation_status  | Status of the cow. Possible parameters: `Young_Cow`, `Lactating_Cow`, `Dry_Cow`. Changes according to events.
location          | Location where the cow currently is.
mark              | Animal number. | `required` for creation.
mixed_race        | When the animal is a breed of two or more races. Example: `"mixed_race": [{"race": "ANGLER", "percent": 22}, ` `{"race": "FLECKVIEW", "percent": 78}]`. The percent needs to sum up to 100. If mixed race is set `"race": "mixed"`.
name              | Name of the animal.
official_id       | Official ID (ear tag) of the animal. Needs to be unique in the organisation. If the official_id is in the data of the PUT methode and is different from the route the official_id gets updated.
official_id_rule  | The rule how the official_id should be formated/stored. Possible parameters: `AT`, `BE`, `BG`, etc. (country code).
organisation_id   | ID of the organisation where the animal belongs to.
race              | Race of the animal.
tags              | Tags which can be given to the animals. Example: `"tags": ["sold"]`.


## Get Animal Metrics

> Request

```python

import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/organisations/' + organisation_id + '/animals/' + official_id + '/metrics'

token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.get(route, headers=headers)

status_code = r.status_code
animal = r.json()

```

```bash
curl -X GET "[endpoint]/organisations/[organisation_id]/animals/[official_id]/metrics" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]" \
        -H  "Content-Type: application/json"
```


> Response example

```json
[
    {
        "metric": "temp",
        "available_units": [
            "degree_celsius",
            "degree_fahrenheit"
        ]
    },
    {
        "metric": "act",
        "available_units": [
            "act"
        ]
    },
    {
        "metric": "act_index",
        "available_units": [
            "percent"
        ]
    }
]
```

This Endpoint retrieves all metrics and the corresponding availlable units from an animal which is in the organisation with `organisation_id` and the `official_id`.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/animals/[official_id]/metrics"`

**URL Parameters**

Parameter | Description ||
--------- | ----------- | ---
organisation_id | ID of the organisation where the animal belongs to. | `required`
official_id | The official id of the animal. This is in the most cases the eartag of the animal. | `required`


## Get Animal Data

> Request

```python

import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/organisations/' + organisation_id + '/animals/' + official_id + '/data.json'

token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

data = {
    "metrics": ["temp", "act"],
    "from_date": "2019-02-01T08:21:33+0000",
    "to_date": "2019-02-01T12:22:16+0000"
}

r = requests.get(route, data=data, headers=headers)

status_code = r.status_code
animal = r.json()

```

```bash
curl -X GET "[endpoint]/organisations/[organisation_id]/animals/[official_id]/data.json" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]" \
        -H  "Content-Type: application/json" \
        -d "{  \"metrics\": [\"temp\", \"act\"],  \"from_date\": \"2019-02-01T08:21:33+0000\",  \"to_date\": \"2019-02-01T12:22:16+0000\"}"
```

> Response example

```json
[
    {"metric": "temp", "data": [
            ["2019-02-01T09: 00: 00+01: 00",
                38.13
            ],
            ["2019-02-01T10: 00: 00+01: 00",
                38.52
            ],
            ["2019-02-01T11: 00: 00+01: 00",
                38.85
            ],
            ["2019-02-01T12: 00: 00+01: 00",
                36.89
            ],
            ["2019-02-01T13: 00: 00+01: 00",
                37.6
            ]
        ], "unit": "degree_celsius"
    },
    {"metric": "act", "data": [
            ["2019-02-01T09: 00: 00+01: 00",
                3.35
            ],
            ["2019-02-01T10: 00: 00+01: 00",
                3.31
            ],
            ["2019-02-01T11: 00: 00+01: 00",
                3.46
            ],
            ["2019-02-01T12: 00: 00+01: 00",
                3.99
            ],
            ["2019-02-01T13: 00: 00+01: 00",
                4.3
            ]
        ], "unit": "act"
    }
]
```

This endpoint retrieves from a provided `organisation_id` and `official_id` data from its sensor. The response depends on the provided list of `metrics` and the period.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/animals/[official_id]/data.json"`

**URL Parameters**

Parameter | Description ||
--------- | ----------- | ---
organisation_id | ID of the organisation where the animal belongs to. | `required`
official_id | The official id of the animal. This is in the most cases the eartag of the animal. | `required`
||
metrics | The list of metrics where data is wanted. | `required`
from_date | The start date from the period where data is wanted. | `required`
to_date | The end date from the period where data is wanted. | `required`
aggregation_period | The period of data points. Either 10 minutes or 1 hour.
preferred_units | The units which are prefered. For example: "act", "degree_celsius", "degree_fahrenheit".


# Events

Occurrences which arise for an animal are stored in the **events**. These events include reproduction events, health events, feeding events etc.. All events can be fetched but just events of the type `heat`, `insemination`, `pregnancy_result`, `dry_off`, `calving_confirmation`, `abort` and `diagnosis` can be created via the API. Additional information regarding the event type can be added. All other events are generated by the smaXtec system.

## Get All Events

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/organisations/' + organisation_id + '/animals/' + official_id + '/events'
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.get(route, headers=headers)

status_code = r.status_code
all_animal_events = r.json()
```

```bash
curl -X GET "[endpoint]/organisations/[organisation_id]/animals/[official_id]/events" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
[
  {
    "_id": "5b61e5bd90e6e1d84e4a2123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T16:54:21+00:00",
    "event_ts": "2015-03-23T00:00:00+00:00",
    "event_type": "calving_confirmation",
    "calving_number": 3
  },
  {
    "_id": "5b61e5bdb493c3931a260123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T16:54:21+00:00",
    "event_ts": "2015-03-23T01:00:00+00:00",
    "event_type": "information_update"
  },
  {
    "_id": "5b61eb7e2782f75791c5e123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T17:18:54+00:00",
    "event_ts": "2017-05-15T00:00:00+00:00",
    "event_type": "heat",
    "cycle_length": 300
  },
  {
    "_id": "5b61ed2c19cd8518817d6123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T17:26:04+00:00",
    "event_ts": "2017-07-22T20:13:00+00:00",
    "event_type": "insemination",
    "days_to_calving": 275
  },
  {
    "_id": "5b61ed2c041922c1f7030123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T17:26:04+00:00",
    "event_ts": "2017-07-22T23:13:00+00:00",
    "event_type": "pregnancy_result",
    "pregnant": true,
    "expected_calving_date": "2018-04-23T12:00:00+00:00",
    "insemination_date": "2017-07-22T20:13:00+00:00"
  },
  {
    "_id": "5ae62c4ad591117f095f4123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-04-29 21:00:00",
    "event_ts": "2018-04-29 21:00:00",
    "event_type": "health_106"
  }
]
```

This endpoint retrieves from a provided `organisation_id` and `official_id` the events of an animal.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/animals/[official_id]/events"`

**URL Parameters**

Parameter | Description ||
--------- | ----------- | ---
organisation_id | ID of the organisation where the animal belongs to. | `required`
official_id | The official id of the animal. This is in the most cases the eartag of the animal. | `required`

**Event Types**

The smaXtec system has the following event types.

Event Type           | Description
----------           | -----------
heat                 | Detection of a heat
insemination         | Inseminated of an animal
pregnancy_result     | A positive or negative pregnancy result
calving_detection    | Detection of a calving
dry_off              | Dry off
calving_confirmation | Confirmation of a calving
waiting_for_calving  | Few days before calving
abort                | Abortion of a pregnancy which can be a late abort
information_update   | Updating information
diagnosis            | Diagnose
health_101           | Insufficient water intake
health_102           | Increase in drinking cycles
health_103           | Reduced drinking cycles
health_104           | Temperature increase
health_106           | Temperature drop
health_601           | THI ≥ 72 Heat stress
health_602           | THI ≥ 78 Heat stress
health_603           | THI ≥ 82 Heat stress
health_703           | Drop in activity
feeding_201          | Feed efficiency
feeding_202          | Risk of acidosis
feeding_203          | Increase in average pH
feeding_204          | Drop in average pH
feeding_205          | Increase in 12h pH
feeding_206          | Drop in 12h pH
fertility_105        | Indication of imminent calving
fertility_702        | Oestrus
actincrease_701      | Increase in activity
actincrease_704      | Increase in activity

## Create Events

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/organisations/' + organisation_id + '/animals/' + official_id + '/events'
data = {
    "events": [
        {
            "event_type": "insemination",
            "event_ts": "2019-01-18T12:00:00+00:00",
            "days_to_calving": 275
        }
    ]
}
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.put(route, json=data, headers=headers)

status_code = r.status_code
all_animal_events = r.json()
```

```bash
curl -X PUT "[endpoint]/organisations/[organisation_id]/animals/[official_id]/events" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
        -d "{\"events\": [{\"event_type\": \"insemination\", \"event_ts\": \"2019-01-18T12:00:00+00:00\", \"days_to_calving\": 275}]}"
```

> Response example (will change)

```json
[
  {
    "_id": "5c584ba651945498be341123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2019-02-04T14:26:46+00:00",
    "event_ts": "2019-01-18T12:00:00+00:00",
    "event_type": "insemination",
    "days_to_calving": 275
  }
]
```

This endpoint creates events for an animal with the given `official_id` which belongs to the organisation with the given `organisation_id`. If the event is created depents on the `event_type` and the `event_ts`. If the event with this two parameters does not exist, it gets created. Otherwise the event is skipped.
It is possible to add just the events to the data which should be added to the smaXtec system or add just all events from your system and the smaXtec system adds those which are not in the system.

<aside class="notice">Just events of the type <b>heat</b>, <b>insemination</b>, <b>pregnancy_result</b>, <b>dry_off</b>, <b>calving_confirmation</b>, <b>abort</b> and <b>diagnosis</b> can be created via the API.</aside>

**HTTP Request**

`PUT "[endpoint]/organisations/[organisation_id]/animals/[official_id]/events"`

**URL Parameters**

Parameter       | Description | |
---------       | ----------- | ----
organisation_id | The ID of the organisation which the animals belong to. | `required`
official_id     | The official id of the animal. This is in the most cases the eartag of the animal. | `required`
||
_id             | ID of the Event in the smaXtec system.
animal_id       | ID of the Aniamal in the smaXtec system.
create_ts       | Time when the event was created.
event_ts        | Time when the event happened.
event_type      | Type of the event.
||
**Heat Parameters**                 |
cycle_length                        | Length of the current cycle.
reason                              | The reason why the heat was added.
||
**Insemination Parameters**         |
days_to_calving                     | Days from the insamination untill the expected calving date. Default: `280`
||
**Pregnancy Result Parameters**     |
expected_calving_date               | Date when the calving is expected.
insemination_date                   | Date of the insamination which led to the pregnancy result. If no prior insemination on that date is saved an insemination event on that date will be created.
pregnant                            | If the pregnancy resutl was positive (true) or negative (false).
||
**Calving Confirmation Parameters** |
calving_number                      | Number of calvings.
||
**Abort Parameters**                |
late_abort                          | When the cow was long enough pregnant so that the cow is lactating then `late_abort` is `true`.
||
**Diagnosis**                       |
icar_key                            | ICAR key for the diagnosis.
diagnosis_key                       | If an own key for diagnosis is used, it should be set here.
diagnosis_key_type                  | If an own key for diagnosis is used, a type of the diagnosis_key should be defined. eg.: `MY_OWN_KEY_DEFENITION_SYSTEM`


# Devices

Devices are all sensors which measure data.

## Get All Devices

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
route = endpoint + '/organisations/' + organisation_id + '/devices'
data = 'all'
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.put(route, json=data, headers=headers)

status_code = r.status_code
all_devices = r.json()
```

```bash
curl -X GET "[endpoint]/organisations/[organisation_id]/devices" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
[
  "0600000001",
  "0600000002",
  "0200000001",
  "0700000001"
]
```

This call delivers all devices of the given organisation if the parameter 'all' is used, Otherwise this call returns just the climate sensors (06) for the given organisation.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/devices"`

**URL Parameters**

Parameter         | Description ||
---------         | ----------- | ---
organisation_id   | ID of the organisation where the animal belongs to. | `required`
device_type   | at the moment 'all' represents all devices.


## Get Device stats

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
device_id = '0200000001'
route = endpoint + '/organisations/' + organisation_id + '/devices/' + device_id
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.get(route, headers=headers)

status_code = r.status_code
all_devices = r.json()
```

```bash
curl -X GET "[endpoint]/organisations/[organisation_id]/devices[device_id]" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
[
  "organisation_id": "569e0bcea80a5f1c07b5430f",
  "_id": "0700007154",
  "last_readout": [
    "timestamp": "2019-11-14T12:22:16",
    "readout_id": "0200000041",
    "basestation_id": "0200000041"
  ]
]
```

This call delivers data about the last readout.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/devices/[device_id]"`

**URL Parameters**

Parameter         | Description ||
---------         | ----------- | ---
organisation_id   | ID of the organisation where the animal belongs to. | `required`
device_id         | ID of the device | `required`


## Get Device Data

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
organisation_id = '123456qwertz'
device_id = '0600000001'
route = endpoint + '/organisations/' + organisation_id + '/devices/' + device_id + '/data.json'

token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

data = {
    "metrics": ["temp"],
    "preferred_units": ["degree_celsius"],
    "from_date": "2019-06-01T12:22:16",
    "to_date": "2019-06-02T12:22:16",
    "timestamp_format": "iso",
    "aggregation_period": "hourly"
}

r = requests.get(route, data=data, headers=headers)

status_code = r.status_code
device_data = r.json()
```

```bash
curl -X GET "[endpoint]/organisations/[organisation_id]/devices/[device_id]/data.json?to_date=2019-06-20T13%3A04%3A52&aggregation_period=hourly&metrics=temp&timestamp_format=iso&from_date=2019-04-20T13%3A04%3A52" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
[
    {
        "metric": "temp",
        "unit": "degree_celsius",
        "data": [
            [
                "2019-06-01T12:23:00+00:00",
                20.77
            ],
            [
                "2019-06-01T12:33:00+00:00",
                21.34
            ],
            [
                "2019-06-01T12:43:00+00:00",
                21.66
            ]
        ]
    }
]
```

This endpoint retrives from a specific `organisation_id` and `device_id` the corresponding data. The response depends furthermore on the list of provided `metrics` and the period of time.

**HTTP Request**

`GET "[endpoint]/organisations/[organisation_id]/devices/[device_id]/data.json"`

**URL Parameters**

Parameter | Description ||
--------- | ----------- | ---
organisation_id     | ID of the organisation where the animal belongs to.   | `required`
device_id           | The ID of the device.                                 | `required`
||
metrics             | The list of metrics which are wanted.                 | `required`
from_date           | The start date from the period of data.               | `required`
to_date             | The end date from the period of data.                 | `required`
timestamp_format    | The format of the provided time stamps. (`iso`, `utc`, `local` or `tuple`)
aggregation_period  | The period of data points. Either 10 minutes or 1 hour. (`10minutes` or `hourly`)
preferred_units     | The units which are prefered. For example: `act`, `degree_celsius`, `degree_fahrenheit`.


# Translations

The translated text for events or specific events can be accessed over the integration API.

## Get all Event Translations

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
language = 'en'
route = endpoint + '/translations/' + language + '/events'
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.get(route, headers=headers)

status_code = r.status_code
event_text = r.json()
```

```bash
curl -X GET "[endpoint]/translations/[language]/events" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
{
    "EVENT": {
        "HEALTH_101": {
            "TITLE": "Indication of too few daily drinking cycles ({{value | number:0}})",
            "DESCRIPTION": "On {{event_ts | timestamp:'dddd, ll'}} the number of drinking cycles was {{value | number:0}}. <br \/>Please check provision of water and state of health (e.g. heat stress).",
            "KEYWORD": "Insufficient water intake"
        },
        "HEALTH_102": {
            "TITLE": "Indication of an increase in number of drinking cycles compared with the previous day by {{value | number: 0}}",
            "DESCRIPTION": "On {{event_ts | timestamp:'dddd, ll'}} the animal drank {{value | number:0}} times more than the previous day. <br \/>Please check animal's state of health and possible heat stress.",
            "KEYWORD": "Increase in drinking cycles"
        }
    }
}
```

With this call translations for the animal events can be accessed for various languages. The translations contain for each event a `KEYWORD`, a `TITLE` and a `DESCRIPTION`.

<aside class="notice">Current available languages: <b>de, en-au, en, es, fr, hr, hu, it, ko, lv, nl, pl, pt, ro, ru, sl, tr, uk, zh-cn</b></aside>

**HTTP Request**

`GET "[endpoint]/translations/[language]/events"`

**Response**

Key           | Descrition
---           | ----------
`KEYWORD`     | Keywords for the event translation.
`TITLE`       | Short description.
`DESCRIPTION` | Detailed information about the event.

## Get single Event Translations

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/integration/v2'
language = 'en'
event_type = 'health_102'
route = endpoint + '/translations/' + language + '/events/' + event_type
token = 'yx2zvuB8JD8ppwGti84OT8Muq5eiB2b2EZqsqC-HOXUvLSg'
headers = {
    'accept': 'application/json',
    'Authorization': 'bearer ' + token
}

r = requests.get(route, headers=headers)

status_code = r.status_code
health_102_text = r.json()
```

```bash
curl -X GET "[endpoint]/translations/[language]/events/[event_type]" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example

```json
{
    "EVENT": {
        "health_102": {
            "DESCRIPTION": "On {{event_ts | timestamp:'dddd, ll'}} the animal drank {{value | number:0}} times more than the previous day. <br />Please check animal's state of health and possible heat stress.",
            "KEYWORD": "Increase in drinking cycles",
            "TITLE": "Indication of an increase in number of drinking cycles compared with the previous day by {{value | number: 0}}"
        }
    }
}
```

It is also possible to access the translation for just a single event.

**HTTP Request**

`GET "[endpoint]/translations/[language]/events/[event_type]"`
