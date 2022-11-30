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
participant_id | No | | If provided, the participant_id must match.  Otherwise, a 404 error is returned.
participant_country | No
consent_document_id | No

## Update Participants

This endpoint matches a participant based on the `thread_participant_id`, `study_id` and `site_id`.
If a match is found, all other participant properties are updated in eConsent system.  Otherwise a 404 error is returned.

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
