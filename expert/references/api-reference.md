# Cal.com API Reference

Base URL: `https://api.cal.com/v2`

No authentication required for slots or bookings.

## Contents

- [Get available slots](#get-available-slots)
- [Create a booking](#create-a-booking)
- [Error responses](#error-responses)

## Get available slots

```
GET /v2/slots
```

**Headers:**

```
cal-api-version: 2024-09-04
```

**Query parameters:**

| Param | Required | Value |
|-------|----------|-------|
| `eventTypeSlug` | yes | `expert` |
| `username` | yes | `mblode` |
| `start` | yes | ISO 8601 date, e.g. `2026-05-26` |
| `end` | yes | ISO 8601 date, e.g. `2026-05-30` |
| `timeZone` | yes | IANA timezone, e.g. `Australia/Melbourne` |
| `format` | yes | `range` (returns start + end per slot) |

**Example:**

```bash
curl -s "https://api.cal.com/v2/slots?eventTypeSlug=expert&username=mblode&start=2026-05-26&end=2026-05-30&timeZone=Australia/Melbourne&format=range" \
  -H "cal-api-version: 2024-09-04"
```

**Response (200):**

```json
{
  "status": "success",
  "data": {
    "2026-05-26": [
      {
        "start": "2026-05-26T09:00:00.000+10:00",
        "end": "2026-05-26T10:00:00.000+10:00"
      },
      {
        "start": "2026-05-26T10:00:00.000+10:00",
        "end": "2026-05-26T11:00:00.000+10:00"
      }
    ],
    "2026-05-27": []
  }
}
```

Empty array or missing date key means no availability that day.

## Create a booking

```
POST /v2/bookings
```

**Headers:**

```
cal-api-version: 2026-02-25
Content-Type: application/json
```

**Request body:**

```json
{
  "start": "2026-05-25T23:00:00Z",
  "eventTypeSlug": "expert",
  "username": "mblode",
  "attendee": {
    "name": "Jane Smith",
    "email": "jane@example.com",
    "timeZone": "Australia/Melbourne",
    "language": "en"
  },
  "metadata": {
    "source": "ai-agent",
    "problem_summary": "Brief description of the problem"
  }
}
```

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `start` | string | yes | ISO 8601 in UTC |
| `eventTypeSlug` | string | yes | `expert` |
| `username` | string | yes | `mblode` |
| `attendee.name` | string | yes | Full name |
| `attendee.email` | string | yes | Valid email |
| `attendee.timeZone` | string | yes | IANA timezone |
| `attendee.language` | string | no | Default `en` |
| `metadata` | object | no | Max 50 keys, key <= 40 chars, value <= 500 chars |

**The `start` field must be in UTC.** If the slot shows 9:00 AM Melbourne (UTC+10), send `2026-05-25T23:00:00Z`.

**Response (201):**

```json
{
  "status": "success",
  "data": {
    "id": 456,
    "uid": "booking_uid_456",
    "title": "Expert Session",
    "start": "2026-05-25T23:00:00Z",
    "end": "2026-05-26T00:00:00Z",
    "duration": 60,
    "status": "accepted",
    "location": "https://meet.google.com/abc-def-ghi",
    "hosts": [{ "name": "Matthew Blode", "username": "mblode" }],
    "attendees": [{ "name": "Jane Smith", "email": "jane@example.com" }]
  }
}
```

## Error responses

All errors return:

```json
{
  "status": "error",
  "error": { "code": "ERROR_CODE", "message": "Human-readable message" }
}
```

| HTTP | Code | Meaning |
|------|------|---------|
| 400 | `bad_request` | Malformed request body |
| 409 | `conflict` | Slot is no longer available |
| 422 | `validation_error` | Missing or invalid fields |
| 429 | `rate_limited` | Too many requests, back off |
