---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - 4G Clinical
  - THREAD Research

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the 4G-Hosted API
---

# Introduction

This document describes two separate APIs:

1. <b>`RTSM API`</b> (Thread -> 4G):  This is hosted by 4G Clinical.  It is used to:
   1. query patient information from RTSM
   2. notify RTSM of new eConsent patients events
   3. notify RTSM of certain participant status change events
2. <b>`ECONSENT API`</b> (4G -> Thread): This is hosted by Thread.  It provides endpoints to:
   1. query participant information from eConsent
   2. notify eConsent of certain RTSM status changes (incl. Screening, Randomization and others as specified)

Both APIs use REST with JSON payloads.
# Authentication
## RTSM
```shell
curl --request POST "https://{sponsor-url}.4gclinical.com/_/api/oauth/token" \
  --json '{
      "client_id": "{partner code}",
      "client_secret": "{partner secret}",
      "grant_type": "bearer"
  }'
```

> Make sure to replace `{partner code}` and `{partner secret}` with your credentials.

> The above command returns JSON structured like this:

```json
{
    "access_token": "fsVjdlCDAfwenfr43klfkl3...2er23",
    "token_type": "bearer",
    "expires_in": 1800,
    "scope": "econsent"
}
```

The RTSM API uses OAuth2's Client Credentials Flow to authenticate requests.

4G Clinical will assign unique `partner code` and `partner secret` for approved partners.  Separate credentials will be used on different environments (i.e. dev, testing, production, etc.)

### HTTP Request

`POST https://sponsor-url.4gclinical.com/_/api/oauth/token`

To authenticate, the partner should send `partner code` and `partner secret` to the <b>Oauth Token</b> endpoint.
The API will verify the credentials and return an `access token`, valid for a limited time (usually ~30 minutes).
A renewed `access token` can be retrieved at any time by calling the Oauth Token endpoint again.

The API expects a valid `access token` to be included in all subsequent API requests in a HTTP header:

`Authorization: Bearer {access token}`

<aside class="notice">
You must replace <code>{access token}</code> with the token returned by the oauth endpoint.
</aside>

## ECONSENT

TBD (Basic Authentication?)
# RTSM API

## Common Requirements

### Request Headers

All RTSM endpoints must include the following headers:

Header | Format | Required | Description
------ | ------ | -------- | -----------
Authorization | `Bearer: {access_token}` | Yes | The access token must valid (i.e. not expired)
study_code | | Yes |

<aside class="warning">The <code>study_code</code> must match across systems</aside>

### Patient Response Schema

Endpoints that return patient information from RTSM will use the following schema:

Parameter | Format | Description
--------- | ------ | -----------
thread_participant_id |
site_id |
participant_id |
consent_signed_on | yyyy-mm-dd | Consent date
thread_participant_status |
thread_participant_status_ts | yyyy-MM-ddThh:mm:ssZ | Timestamp of the status change (UTC)
participant_country |
consent_document_id |
rtsm_screening_status | Pass / Fail |
rtsm_screening_date | yyyy-MM-dd | | Local site date
rtsm_randomized | Yes / No |
rtsm_randomized_ts | yyyy-MM-ddThh:mm:ssZ | Timestamp of the status change (consent recorded in system) (Q: UTC? If Local site time?, will need offset)
rtsm_discontinued | Yes / No |
rtsm_discontinued_date | yyyy-MM-dd | | Local site date

<aside class="info">Note: Some fields may be null or missing.</aside>

## Post an eConsent notification for a new Patient
```shell
curl --request POST "https://{sponsor-url}.4gclinical.com/_/api/partner/econsent/patients/" \
  --header 'Authorization: Bearer {access_token}' \
  --header 'study_code: Study_123' \
  --json '{
      "thread_participant_id": "A123456789",
      "site_id": "101",
      "consent_signed_on": "2020-12-01",
      "thread_participant_status": "CONSENTED",
      "thread_participant_status_ts": "2020-12-02T01:25:00Z",
      "participant_country": "Canada",
      "consent_document_id": "120333"
  }'
```

### HTTP Request

`POST https://sponsor-url.4gclinical.com/_/api/partner/econsent/patients/`

### Request Body

Parameter | Required | Format | Description
--------- | -------- | ------ | -----------
thread_participant_id | Yes
site_id | Yes
consent_signed_on | Yes | yyyy-mm-dd | Consent date.  (Q: what timezone is that date in?)
thread_participant_status | Yes
thread_participant_status_ts | Yes | yyyy-MM-ddThh:mm:ssZ | Timestamp of the status change (consent recorded in system) (UTC)
participant_country | No
consent_document_id | No

<aside class="warning">Note: <code>site_id</code> must match across systems.</aside>

## Get a List of Patients
```shell
curl "https://sponsor-url.4gclinical.com/_/api/partner/econsent/patients/?site_id=101&rtsm_patient_id=2005" \
  --header 'Authorization: Bearer {access_token}' \
  --header 'study_code: Study_123'
```

> The above command returns JSON structured like this:

```json
[
    {
        "thread_participant_id": "A123456789",
        "site_id": "101",
        "participant_id": "2005",
        "consent_signed_on": "2020-12-01",
        "thread_participant_status": "CONSENTED",
        "thread_participant_status_ts": "2020-12-02T01:25:00Z",
        "participant_country": "Canada",
        "consent_document_id": "120333",
        "rtsm_screening_status	": "Pass",
        "rtsm_screening_date": "2020-12-04",
        "rtsm_randomized": "No",
        "rtsm_randomized_ts": null,
        "rtsm_discontinued": "No",
        "rtsm_discontinued_date": null
    }
]
```

Returns a list of patient records matching the specified query parameters.

If no query parameters are provided, all study patient records will be included in the result.  If no patients meet the
query parameters, an empty list will be returned.

### HTTP Request

`GET https://sponsor-url.4gclinical.com/_/api/partner/econsent/patients/`

### Query Parameters

Parameter | Description
--------- | -----------
site_id | If set, the result will only include patients from the site.
rtsm_patient_id | If set, the site must also by included.  The result will only include the specific patient.

## Get a Specific Patient
```shell
curl "https://sponsor-url.4gclinical.com/_/api/partner/econsent/patients/A123456789" \
  --header 'Authorization: Bearer {access_token}' \
  --header 'study_code: Study_123'
```

> The above command returns JSON structured like this:

```json
{
    "thread_participant_id": "A123456789",
    "site_id": "101",
    "participant_id": "2005",
    "consent_signed_on": "2020-12-01",
    "thread_participant_status": "CONSENTED",
    "thread_participant_status_ts": "2020-12-02T01:25:00Z",
    "participant_country": "Canada",
    "consent_document_id": "120333",
    "rtsm_screening_status": "Pass",
    "rtsm_screening_date": "2020-12-04",
    "rtsm_randomized": "No",
    "rtsm_randomized_ts": null,
    "rtsm_discontinued": "No",
    "rtsm_discontinued_date": null
}
```
### HTTP Request

`GET https://sponsor-url.4gclinical.com/_/api/partner/econsent/patients/{thread_participant_id}`

### URL Parameters

Parameter | Description
--------- | -----------
thread_participant_id | The thread participant ID.

## Update a Patient (status)
```shell
curl --request PUT "https://sponsor-url.4gclinical.com/_/api/partner/econsent/patients/A123456789" \
  --header 'Authorization: Bearer {access_token}' \
  --header 'study_code: Study_123' \
  --json '{
      "thread_participant_id": "A123456789",
      "site_id": "101",
      "participant_id": "2005",
      "consent_signed_on": "2020-12-01",
      "thread_participant_status": "WITHDREW CONSENT",
      "thread_participant_status_ts": "2020-12-22T05:54:00Z",
      "participant_country": "Canada",
      "consent_document_id": "120333"
}
```

### HTTP Request

`PUT https://sponsor-url.4gclinical.com/_/api/partner/econsent/patients/{thread_participant_id}`

### Request Body

Parameter | Required | Format | Description
--------- | -------- | ------ | -----------
thread_participant_id | Yes
site_id | Yes
consent_signed_on | Yes | yyyy-mm-dd | Consent date (Q: local timezone?)
thread_participant_status | Yes
thread_participant_status_ts | Yes | yyyy-MM-ddThh:mm:ss | Timestamp of the status change (UTC)
participant_id | No | | If provided, the participant_id must match.  Otherwise, a 404 error will be returned.
participant_country | No
consent_document_id | No

# ECONSENT API

## Get a list of Participants

### HTTP Request

`GET https://example.thread.com/api/v1/participants/`

### Query Parameters

Parameter | Description
--------- | -----------
study_id | Required (??).
site_id | If set, filter results by the site.
participant_id | If set, site_id is also required; filter results by participant_id.

### Response Body

Parameter | Required | Format | Description
--------- | -------- | ------ | -----------
thread_participant_id | Yes
site_id | Yes
consent_signed_on | Yes | yyyy-mm-dd | Consent date (Q: local timezone?)
thread_participant_status | Yes
thread_participant_status_ts | Yes | yyyy-MM-ddThh:mm:ss | Timestamp of the status change (UTC)
participant_id | No | | If provided, the participant_id must match.  Otherwise, a 404 error will be returned.
participant_country | No
consent_document_id | No

## Update Participants

This endpoint will match a participant based on the `thread_participant_id`, `study_id` and `site_id`.
If a match is found, all other participant properties will be updated in eConsent system.  Otherwise a 404 error wwill
be returned.

### HTTP Request

`PUT https://example.thread.com/api/v1/participants/{thread_participant_id}`

### Request Body

Parameter | Required | Format | Description
--------- | -------- | ------ | -----------
thread_participant_id | Yes |
study_id | Yes
site_id | Yes
participant_id | No |
rtsm_screening_date | No | yyyy-MM-dd | | Local site date
rtsm_screening_status | No | Pass / Fail |
rtsm_randomized | No | Yes / No |
rtsm_randomized_ts | No | yyyy-MM-ddThh:mm:ss | Timestamp of the status change (consent recorded in system) (Local site time?)
rtsm_discontinued | No | Yes / No |
rtsm_discontinued_date | No | yyyy-MM-dd | | Local site date
