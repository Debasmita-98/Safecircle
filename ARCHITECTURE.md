# SafeCircle architecture

## 1. Application flow

The React client owns presentation, browser geolocation, and user interaction. It talks to an Express REST API for durable business operations and to Socket.IO for low-latency location events. Prisma persists relational data in PostgreSQL.

```text
Browser -> REST API -> services -> Prisma -> PostgreSQL
Browser <-> Socket.IO <-> socket authorization/persistence -> PostgreSQL
```

## 2. Why PostgreSQL

The domain has strong relationships: users own contacts, sessions own locations/events/tokens, and notification/audit records refer to sessions and contacts. PostgreSQL provides transactions, foreign keys, indexes, JSON metadata, and predictable consistency for emergency workflow state.

## 3. Why Socket.IO for live location

Live GPS positions are continuously changing. Polling a REST endpoint every few seconds would add request overhead and latency. Socket.IO provides a persistent bidirectional channel, rooms for per-session subscriptions, reconnection support, and a clean event model.

REST is still used for durable state changes such as creating sessions, generating share links, resolving incidents, and loading history.

## 4. Why UUIDs

UUIDs avoid exposing simple sequential identifiers and are a better fit if the application is later split across services or creates records in multiple regions. Authorization is still mandatory; UUIDs are not a replacement for access control.

## 5. Public share-link design

A public tracking link contains a cryptographically random token. The raw token is returned only when created. PostgreSQL stores only its SHA-256 hash. A link has an expiry time and optional revocation time.

When a session is resolved, all active share tokens are revoked. Public API responses intentionally contain only the minimum tracking data needed by the trusted viewer.

## 6. Realtime frequency vs database persistence

The client can emit positions frequently enough for a responsive map, while the server persists only when enough time has passed or the user has moved a meaningful distance. This prevents high-frequency GPS updates from becoming unnecessary PostgreSQL writes.

For a much larger deployment, the latest position should live in Redis or another ephemeral store and be periodically materialized into PostgreSQL.

## 7. Authentication/security decisions

- Access JWT: short lived; sent in Authorization header.
- Refresh JWT: HttpOnly cookie; token hash stored server-side and rotated.
- Password: bcrypt hash.
- Ownership checks: every user resource query includes the authenticated user ID.
- Share tokens: random, hashed, expiring, revocable.
- Validation: Zod on API inputs.
- HTTP hardening: Helmet, CORS, rate limits.
- Provider secrets: backend environment variables only.

## 8. Browser background-location limitation

`navigator.geolocation.watchPosition()` is useful for a web MVP but mobile operating systems can suspend a background browser tab or stop geolocation when a device is locked. The UI and documentation do not claim guaranteed background tracking.

## 9. React Native evolution

A React Native client can reuse the same authentication, contacts, sessions, SOS, share-link, history, notification, and AI APIs. Only the location-capture/push-notification layer needs native/mobile-specific implementation.

## 10. Notification provider abstraction

Business logic calls `NotificationService`, not Twilio or an email vendor directly. The included development provider writes notification rows and logs messages. Production adapters can be added without changing the SOS service.

## 11. Scaling path

1. Deploy multiple stateless API instances.
2. Add Redis adapter for Socket.IO rooms/events.
3. Store latest live positions in Redis and persist sampled history to PostgreSQL.
4. Replace demo check-in interval with a durable queue/worker.
5. Add object storage/log aggregation/metrics/tracing.
6. Partition/archive old location points according to a retention policy.
