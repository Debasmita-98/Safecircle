# API summary

Base URL: `/api`

Authenticated endpoints expect:

```http
Authorization: Bearer <access-token>
```

Refresh token is an HttpOnly cookie.

## Auth

- `POST /auth/register` `{ name, email, password, phone? }`
- `POST /auth/login` `{ email, password }`
- `POST /auth/refresh`
- `POST /auth/logout`
- `GET /users/me`

## Contacts

- `GET /contacts`
- `POST /contacts`
- `PATCH /contacts/:id`
- `DELETE /contacts/:id`

## Safety sessions

- `POST /safety-sessions` `{ reason?, latitude, longitude }`
- `GET /safety-sessions`
- `GET /safety-sessions/:id`
- `PATCH /safety-sessions/:id/resolve`

## SOS

- `POST /sos` `{ reason?, latitude, longitude }`

Creates/activates an emergency session, timeline event, public share token, and notification records.

## Share links

- `POST /share-links` `{ sessionId, expiresInMinutes? }`
- `GET /public/share/:token` (public)

## Check-ins

- `POST /checkins` `{ destination, expectedAt }`
- `GET /checkins`
- `PATCH /checkins/:id/complete`
- `PATCH /checkins/:id/cancel`

## AI assistant

- `POST /ai/chat` `{ conversationId?, message }`

## Socket events

Client -> server:

- `session:join` `{ sessionId, shareToken? }`
- `location:update` `{ sessionId, latitude, longitude, accuracy? }`

Server -> client:

- `location:broadcast`
- `session:resolved`

Authenticated location publishers connect with `auth: { token: <access JWT> }`. Public viewers connect without a JWT and prove access using the share token in `session:join`.
