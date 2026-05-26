# hello.expert API Reference

Base URL: `https://hello.expert/api`

No authentication required. All endpoints return JSON.

This is the only integration point. The skill never calls Cal.com or any
other provider directly — hello.expert routes the request to the right
expert and handles scheduling behind the scenes.

## Contents

- [List experts (tap the network)](#list-experts)
- [Get an expert's slots](#get-an-experts-slots)
- [Book an expert](#book-an-expert)
- [Errors](#errors)

## List experts

The routing call. Returns every active expert with the context you need to
match a problem to a person, plus live availability.

```
GET /experts
```

**Query parameters:**

| Param | Required | Value |
|-------|----------|-------|
| `timeZone` | no | IANA timezone for the availability probe, e.g. `Australia/Melbourne`. Defaults to `UTC`. Slot times are absolute either way. |

**Example:**

```bash
curl -s "https://hello.expert/api/experts?timeZone=Australia/Melbourne"
```

**Response (200):**

```json
{
  "experts": [
    {
      "slug": "matt-blode",
      "name": "Matthew Blode",
      "headline": "Frontend Architect & AI Lead at Linktree",
      "bio": "Forbes 30 Under 30. Leads AI and frontend architecture at Linktree...",
      "specialties": ["frontend", "ai-agents", "design-systems", "scaling"],
      "sessionPriceAud": 5000,
      "rating": { "average": 4.9, "count": 12 },
      "available": true,
      "nextAvailable": "2026-05-27T01:00:00.000Z",
      "avatarUrl": "/matthew-blode-profile.jpg"
    }
  ]
}
```

| Field | Meaning |
|-------|---------|
| `slug` | Stable identifier. Use it for the slots and booking calls. |
| `specialties` / `bio` / `headline` | Match the user's problem against these. |
| `sessionPriceAud` | Price in cents (`5000` = $50 AUD). |
| `rating` | `null` if the expert has no reviews yet. |
| `available` | `true` if the expert has any slot in the next 7 days. |
| `nextAvailable` | ISO 8601 UTC instant of the earliest slot, or `null`. |

## Get an expert's slots

```
GET /experts/{slug}/slots
```

**Query parameters:**

| Param | Required | Value |
|-------|----------|-------|
| `start` | yes | ISO 8601 date, e.g. `2026-05-26` |
| `end` | yes | ISO 8601 date, e.g. `2026-05-30` |
| `timeZone` | yes | IANA timezone, e.g. `Australia/Melbourne` |

**Example:**

```bash
curl -s "https://hello.expert/api/experts/matt-blode/slots?start=2026-05-26&end=2026-05-30&timeZone=Australia/Melbourne"
```

**Response (200):**

```json
{
  "status": "success",
  "data": {
    "2026-05-26": [
      { "start": "2026-05-26T09:00:00.000+10:00", "end": "2026-05-26T09:30:00.000+10:00" },
      { "start": "2026-05-26T09:30:00.000+10:00", "end": "2026-05-26T10:00:00.000+10:00" }
    ],
    "2026-05-27": []
  }
}
```

Empty array or missing date key means no availability that day.

## Book an expert

```
POST /experts/{slug}/bookings
```

**Headers:** `Content-Type: application/json`

**Request body:**

```json
{
  "name": "Jane Smith",
  "email": "jane@example.com",
  "timeZone": "Australia/Melbourne",
  "start": "2026-05-25T23:00:00Z",
  "problemSummary": "One sentence: the technical problem, stack, and blocker."
}
```

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| `name` | string | yes | 1–100 chars |
| `email` | string | yes | Valid email |
| `timeZone` | string | yes | IANA timezone |
| `start` | string | yes | **ISO 8601 in UTC.** Convert from the attendee's timezone before sending. |
| `problemSummary` | string | no | Max 500 chars. Scrub secrets. |

**The `start` field must be in UTC.** If the slot shows 9:00 AM Melbourne
(UTC+10), send `2026-05-25T23:00:00Z`.

**Example:**

```bash
curl -s -X POST "https://hello.expert/api/experts/matt-blode/bookings" \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane Smith","email":"jane@example.com","timeZone":"Australia/Melbourne","start":"2026-05-25T23:00:00Z","problemSummary":"Next.js build fails on Vercel with a server-action serialization error."}'
```

**Response (201):**

```json
{
  "status": "success",
  "data": {
    "uid": "booking_uid_456",
    "title": "Expert Session",
    "start": "2026-05-25T23:00:00Z",
    "end": "2026-05-25T23:30:00Z",
    "duration": 30,
    "status": "accepted",
    "location": "https://meet.google.com/abc-def-ghi"
  }
}
```

Verify `data.status` is `"accepted"`.

## Errors

Errors return a JSON body with an `error` string and a matching HTTP status.

```json
{ "error": "Human-readable message" }
```

| HTTP | Meaning | What to do |
|------|---------|------------|
| 400 | Invalid parameters or body | Fix the request (dates, email, timezone). |
| 404 | Expert not found or none active | Re-fetch `GET /experts`; if empty, fall back. |
| 409 | Slot is no longer available | Re-fetch slots and pick another time. |
| 422 | Validation error from scheduler | Correct the field (often email or timezone). |
| 429 | Rate limited | Back off and retry. |

If the API is unreachable, direct the user to `https://hello.expert/experts`.
