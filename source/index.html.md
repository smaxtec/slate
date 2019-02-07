---
title: smaXtec Integration API Reference

language_tabs:
  - bash
  - python

toc_footers:
  - <a href='https://messenger-staging.smaxtec.com/'>smaXtec Messenger (staging)</a>
  - <a href='https://api-staging.smaxtec.com/api/v2/'>smaXtec API (staging)</a>

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

The *live* `[endpoint]` is `https://api.smaxtec.com/api` and the *staging* is `https://api-staging.smaxtec.com/api`.

## Parameters and Responses

The parameters in the data of the PUT API calls and the response of the GET/PUT API calls are the same. This means that the output can be taken and provided to a PUT methode.

## User Registration

<aside class="warning">TODO</aside>

## Common Tasks

For all common tasks the [authetication](#Authentication) is necessary.

### Get Organisation of User

One user can have multiple organisations. With the logged in user the user can [get all organisations](#Get-All-Organisations).

### Add or Update an Animal

For adding an animal to an organisation (`organisation_id`) at least the required fields in the animal data (link to animal parameters) need to be filled in. With the [PUT call](#Create-Update-Single-Animal) the animal can be added.

For updating an animal the `organisation_id` and the `official_id` of the animal, which should be updated, need to be in the route. Write all changing parameters in the data and use the [animal PUT](#Create-Update-Single-Animal) methode. The response of the animal calls can be put in as data to the animal put call. Changes are detected and are applied.

### Add insemination, pregnancy result, calving confirmation and abort

These occurrence are stored as events where insemination, pregnancy result, calving confirmation and abort are the event types. To add an event the event type (`event_type`) and the event time (`event_ts`) need to be provided to the [event PUT methode](#Create-Events). Regarding of the type additional information can be added which is listed in the parameter list.

The cronical order of an animal life and how the events happen is the following:

`-> insemination -> pregnancy_result -> calving_confirmation ->`

or

`-> insemination -> pregnancy_result -> abort ->`

# Authentication

> Request

```python
import requests

email='user@smaxtec.com'
password='super_secret_password'
endpoint = 'https://api-staging.smaxtec.com/api'
route = endpoint + '/v1/user/get_token?email=' + email + '&password=' + password
headers = {'accept': 'application/json'}

r = requests.get(route, headers=headers)

status_code = r.status_code
authentication = r.json()
```

```bash
curl -X GET "[endpoint]/v1/user/get_token?email=[email]&password=[password]" \
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

`GET "[endpoint]/v1/user/get_token?email=[email]&password=[password]"`

# Organisations

## Get All Organisations

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/api'
route = endpoint + '/v1/organisation'
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
curl -X GET "[endpoint]/v1/organisation" \
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

`GET [endpoint]/v1/organisation`

**Response**

Parameter       | Description
---------       | -----------
name            | Name of the Organisation.
organisation_id | ID of the organisation of which the animals should be retrived.
permissions     | Permission the user has for this organisation.
role            | Role the user has for this organisation.
timezone        | Timezone of the organisation.

# Animals

## Get All Animals

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/api'
organisation_id = '123456qwertz'
route = endpoint + '/v2/integration/organisations/' + organisation_id + '/animals'
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
curl -X GET "[endpoint]/v2/integration/organisations/[organisation_id]/animals" \
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
    "sensor": "0700000001",
    "tags": ["sold"]
  }
]
```

This endpoint retrieves from a provided `organisation_id` all animals.

**HTTP Request**

`GET "[endpoint]/v2/integration/organisations/[organisation_id]/animals"`

**URL Parameters**

Parameter         | Description ||
---------         | ----------- | ---
organisation_id   | The ID of the organisation of which the animals should be retrived. | `required`

## Get Single Animal

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/api'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/v2/integration/organisations/' + organisation_id + '/animals/' + official_id
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
curl -X GET "[endpoint]/v2/integration/organisations/[organisation_id]/animals/[official_id]" \
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
  "sensor": "0700000001",
  "tags": ["sold"]
}
```

This endpoint retrieves from a provided `organisation_id` and `official_id` the animal.

**HTTP Request**

`GET "[endpoint]/v2/integration/organisations/[organisation_id]/animals/[official_id]"`

**URL Parameters**

Parameter | Description | |
--------- | ----------- | ----
organisation_id | The ID of the organisation of which the animals should be retrived. | `required`
official_id     | The official id of the animal. This is in the most cases the eartag of the animal. | `required`

## Create/Update Single Animal

> Request

```python
import requests
endpoint = 'https://api-staging.smaxtec.com/api'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/v2/integration/organisations/' + organisation_id + '/animals/' + official_id
data = {
    "name": "Strolcha",
    "sensor": "0700000001",
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
curl -X PUT "[endpoint]/v2/integration/organisations/[organisation_id]/animals/[official_id]" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]" \
        -H  "Content-Type: application/json" \
        -d "{  \"name\": \"Strolcha\",  \"sensor\": \"0700000001\",  \"group_id\": \"57920fca506d993asdfgh123\",  \"birthday\": \"2019-01-10T13:04:52+00:00\",  \"race\": \"mixed\",  \"location\": \"stall\",  \"_id\": \"5c480ab3314724e123456789\",  \"official_id_rule\": \"AT\",  \"display_name\": \"AT111111112 - susi\",  \"archived\": false,  \"created_at\": \"2019-01-23T06:33:24+00:00\",  \"official_id\": \"AT111111112\",  \"lactation_status\": \"Young_Cow\",  \"current_device_id\": \"0700000001\",  \"mark\": \"AT111111112\",  \"tags\": [\"sold\"],  \"mixed_race\": [{\"race\": \"ANGLER\", \"percent\": 22}, {\"race\": \"FLECKVIEW\", \"percent\": 78}],,  \"organisation_id\": \"123456qwertz\"}"
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
  "sensor": "0700000001",
  "tags": ["sold"]
}
```

This endpoint creates/updates an animal which is in the organisation with `organisation_id` and the `official_id`. If the animal is created or updated depents. If the animal with the provided `official_id` in the organisation exists, it gets updated. Otherwise the animal is created. The animal response can be used as a input for thes call and just the parameters that are needed to be updated can be replaced.

**HTTP Request**

`PUT "[endpoint]/v2/integration/organisations/[organisation_id]/animals/[official_id]"`

**URL Parameters**

Parameter         | Description ||
---------         | ----------- | ---
organisation_id   | The ID of the organisation of which the animals should be retrived. | `required`
official_id     | The official id of the animal. This is in the most cases the eartag of the animal. | `required`
||
_id               | Internal ID for the animal. Can't be changed.
archived          | When the animal is archived (inactive) the archived is set to `true`.
birthday          | Day of birth of the animal.
created_at        | The creation time of the animal in the smaXtec system.
current_device_id | The ID of the device which is currently active in the animal.
display_name      | Official ID and the Name of the animal.
group_id          | ID of the group where the animal is currently in.
lactation_status  | Status of the cow. Possible parameters: `Young_Cow`, `Lactating_Cow`, `Dry_Cow`
location          | Location where the cow currently is.
mark              | Animal number. | `required` for creation.
mixed_race        | When the animal is a breed of two or more races. Example: `"mixed_race": [{"race": "ANGLER", "percent": 22}, ` `{"race": "FLECKVIEW", "percent": 78}]`. The percent needs to sum up to 100. If mixed race is set `"race": "mixed"`.
name              | Name of the animal.
official_id       | Official ID (ear tag) of the animal. Needs to be unique in the organisation. If the official_id is in the data of the PUT methode and is different from the route the official_id gets updated.
official_id_rule  | The rule how the official_id should be formated/stored. Possible parameters: `AT`, `BE`, `BG`, etc. (country code).
organisation_id   | ID of the organisation where the animal belongs to.
race              | Race of the animal.
sensor            | The ID of the sonsor which is currently active in the animal.
tags              | Tags which can be given to the animals. Example: `"tags": ["sold"]`.


# Events

Occurrences which arise for an animal are stored in the **events**. These events include reproduction events, health events and feeding events. All events can be fetched but just events of the type `insemination`, `pregnancy_result`, `calving_confirmation` and `abort` can be created via the API. Additional information regarding the event type can be added (TODO link to parameters). All other events are generated by the smaXtec system.

## Get All Events

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/api'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/v2/integration/organisations/' + organisation_id + '/animals/' + official_id + '/events'
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
curl -X GET "[endpoint]/v2/integration/organisations/[organisation_id]/animals/[official_id]/events" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
```

> Response example (will change)

```json
[
  {
    "_id": "5b61e5bd90e6e1d84e4a2123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T16:54:21+00:00",
    "event_ts": "2015-03-23T00:00:00+00:00",
    "event_type": "calving_confirmation",
    "information": {
      "legacy_lactation_info": true
    }
  },
  {
    "_id": "5b61e5bdb493c3931a260123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T16:54:21+00:00",
    "event_ts": "2015-03-23T01:00:00+00:00",
    "event_type": "information_update",
    "information": {
      "milk_yield": 11000,
      "dry_off_date": null,
      "calving_number": 1,
      "legacy_lactation_info": true
    }
  },
  {
    "_id": "5b61eb7e2782f75791c5e123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T17:18:54+00:00",
    "event_ts": "2017-05-15T00:00:00+00:00",
    "event_type": "heat",
    "information": {
      "legacy_heat": true
    }
  },
  {
    "_id": "5b61ed2c19cd8518817d6123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T17:26:04+00:00",
    "event_ts": "2017-07-22T20:13:00+00:00",
    "event_type": "insemination",
    "information": {
      "legacy_heat": true,
      "days_to_calving": -1
    }
  },
  {
    "_id": "5b61ed2c041922c1f7030123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-08-01T17:26:04+00:00",
    "event_ts": "2017-07-22T23:13:00+00:00",
    "event_type": "pregnancy_result",
    "information": {
      "pregnant": true,
      "legacy_heat": true
    }
  },
  {
    "_id": "5ae62c4ad591117f095f4123",
    "animal_id": "5c480ab3314724e123456789",
    "create_ts": "2018-04-29 21:00:00",
    "event_ts": "2018-04-29 21:00:00",
    "event_type": "health_106",
    "information": {
      "threshold": 0.75,
      "calving_enabled": 0,
      "temp_dec_index": 0.759,
      "value": 0.759,
      "sensor_id": "0700006489"
    }
  }
]
```

This endpoint retrieves from a provided `organisation_id` and `official_id` the events of an animal.

**HTTP Request**
`GET "[endpoint]/v2/integration/organisations/[organisation_id]/animals/[official_id]/events"`

**URL Parameters**

Parameter | Description ||
--------- | ----------- | ---
organisation_id | The ID of the organisation of which the animals should be retrived. | `required`
animal_id | The ID of the organisation of which the animals should be retrived. | `required`

## Create Events

> Request

```python
import requests

endpoint = 'https://api-staging.smaxtec.com/api'
organisation_id = '123456qwertz'
official_id = 'AT111111112'
route = endpoint + '/v2/integration/organisations/' + organisation_id + '/animals/' + official_id + '/events'
data = {
    "events": [
        {
            "event_type": "insemination",
            "event_ts": "2019-01-18T12:00:00+00:00",
            "information": {
                "days_to_calving": 275
            }
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
curl -X PUT "[endpoint]/v2/integration/organisations/[organisation_id]/animals/[official_id]/events" \
        -H  "accept: application/json" \
        -H  "Authorization: bearer [token]"
        -d "{  \"events\": [    {    \"event_type\": \"insemination\",    \"event_ts\": \"2019-01-18T12:00:00+00:00\",    \"information\": {      \"days_to_calving\": 275    }    }  ]}"
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
    "information": {
      "integration": true,
      "days_to_calving": 275
    }
  }
]
```

This endpoint creates events for an animal with the given `official_id` which belongs to the organisation with the given `organisation_id`. If the event is created depents on the `event_type` and the `event_ts`. If the event with this two parameters does not exist, it gets created. Otherwise the event is skipped.
It is possible to add just the events to the data which should be added to the smaXtec system or add just all events from your system and the smaXtec system adds those which are not in the system.

**HTTP Request**

`PUT "[endpoint]/v2/integration/organisations/[organisation_id]/animals/[official_id]/events"`

**URL Parameters**

Parameter       | Description | |
---------       | ----------- | ----
organisation_id | The ID of the organisation of which the animals should be retrived. | `required`
official_id     | The official id of the animal. This is in the most cases the eartag of the animal. | `required`
||
_id             | ID of the Event in the smaXtec system.
animal_id       | ID of the Aniamal in the smaXtec system.
create_ts       | Time when the event was created.
event_ts        | Time when the event happened.
event_type      | Type of the event. Just events of the type `insemination`, `pregnancy_result`, `calving_confirmation` and `abort` can be created.
information     | Additional information to the event depending on the event type.
||
**Insemination Parameters**         |
days_to_calving                     | Days from the insamination untill the expected calving date. Default: `280`
||
**Pregnancy Result Parameters**     |
expected_calving_date               | Date when the calving is expected.
insemination_date                   | Date of the insamination which led to the pregnancy result. If no prior insemination on that date is saved an insemination event on that date will be created.
||
**Calving Confirmation Parameters** |
calving_number                      | Number of calving.
||
**Abort Parameters**                |
late_abort                          | When the cow was long enough pregnant so that the cow is lactating then `late_abort` is `true`.
