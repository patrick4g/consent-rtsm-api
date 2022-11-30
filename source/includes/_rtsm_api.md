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

Endpoints that return patient information from RTSM use the following schema:

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
rtsm_randomized_ts | yyyy-MM-ddThh:mm:ssZ | Timestamp of the status change (consent recorded in system) (Q: UTC? If Local site time?, will need to add TZ offset)
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

If no query parameters are provided, all study patient records are included in the result.  If no patients meet the
query parameters, an empty list is returned.

### HTTP Request

`GET https://sponsor-url.4gclinical.com/_/api/partner/econsent/patients/`

### Query Parameters

Parameter | Description
--------- | -----------
site_id | If set, the result only includes patients from the site.
rtsm_patient_id | If set, the site must also by included.  The result only includes the specific patient.

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
participant_id | No | | If provided, the participant_id must match.  Otherwise, a 404 error is returned.
participant_country | No
consent_document_id | No
