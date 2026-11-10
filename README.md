# SafeCircle

SafeCircle is a portfolio-grade personal safety and emergency coordination platform. It helps a user alert trusted contacts, share a temporary live location link, coordinate an active safety session, record an incident timeline, and access an advisory AI safety assistant.

> SafeCircle does **not** guarantee rescue and does not replace local emergency services.

## Why this project exists

The goal is to demonstrate full-stack product thinking rather than a static UI: relational database design, secure authentication, REST APIs, real-time Socket.IO communication, expiring public share tokens, geolocation, provider abstractions, Docker, and documented trade-offs.

## Architecture

```text
React + TypeScript + TanStack Query
           |
           | REST + Socket.IO
           v
Node.js + Express + TypeScript
           |
           | Prisma ORM
           v
       PostgreSQL

External/provider layer:
- Leaflet + OpenStreetMap (map tiles)
- Notification provider abstraction (console/database fallback included)
- AI provider abstraction (safe deterministic fallback included)
```

See `docs/ARCHITECTURE.md` for the design decisions and `docs/API.md` for the API summary.

## Main features

- Register, login, refresh-token rotation, logout
- Trusted emergency contacts
- Safety sessions with initial GPS position
- SOS flow with event timeline and notifications
- Cryptographically random, hashed, expiring public tracking links
- Real-time location updates with Socket.IO
- Leaflet + OpenStreetMap live map and path
- Location persistence throttling to reduce database writes
- Safety check-ins and a demo missed-check-in worker
- Incident/session history
- AI safety assistant with an API-provider adapter and safe fallback
- Role-ready authorization
- Docker Compose for PostgreSQL + backend + frontend

## Tech stack

**Frontend:** React, TypeScript, Vite, React Router, TanStack Query, Axios, Socket.IO Client, React Leaflet

**Backend:** Node.js, TypeScript, Express, Socket.IO, Prisma, Zod, JWT, bcryptjs, Helmet, rate limiting

**Database:** PostgreSQL

**Infrastructure:** Docker, Docker Compose, nginx (frontend container)

## Quick start with Docker

Requirements: Docker Desktop / Docker Engine with Compose.

```bash
docker compose up --build
```

Then open:

- Frontend: http://localhost:5173
- Backend health: http://localhost:4000/health

The backend container runs `prisma db push` on startup so a fresh demo database is initialized automatically.

Seed demo data in another terminal:

```bash
docker compose exec backend npm run seed
```

Demo account:

```text
Email: demo@safecircle.local
Password: Demo12345!
```

## Local development without Docker for Node services

You still need a PostgreSQL database. Copy the example environment files first.

```bash
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
```

Backend:

```bash
cd backend
npm install
npx prisma generate
npx prisma db push
npm run seed
npm run dev
```

Frontend (new terminal):

```bash
cd frontend
npm install
npm run dev
```

## Core demo flow

1. Register or login.
2. Add a trusted contact.
3. Start a safety session or press SOS from the dashboard.
4. Allow browser location permission.
5. Open the emergency screen and generate/copy a secure tracking link.
6. Open that link in an incognito/second browser window.
7. Move with the device or use browser devtools geolocation overrides; the public map updates via Socket.IO.
8. Mark the session safe/resolved.
9. The public link is revoked and the event is recorded in history.

## Environment variables

Backend (`backend/.env`):

```text
NODE_ENV=development
PORT=4000
DATABASE_URL=postgresql://safecircle:safecircle@localhost:5432/safecircle?schema=public
FRONTEND_ORIGIN=http://localhost:5173
PUBLIC_BASE_URL=http://localhost:5173
JWT_ACCESS_SECRET=replace-with-a-long-random-secret
JWT_REFRESH_SECRET=replace-with-another-long-random-secret
ACCESS_TOKEN_TTL=15m
REFRESH_TOKEN_DAYS=7
SHARE_TOKEN_MINUTES=120
AI_API_URL=
AI_API_KEY=
AI_MODEL=
```

Frontend (`frontend/.env`):

```text
VITE_API_URL=http://localhost:4000/api
VITE_SOCKET_URL=http://localhost:4000
```

## External credentials

No paid credential is required for the core demo.

- **Maps:** OpenStreetMap tiles through Leaflet; no API key.
- **Notifications:** database + console fallback works immediately. `NotificationProvider` is designed for Twilio/SMTP/Resend adapters later.
- **AI:** deterministic safety-oriented fallback works immediately. If `AI_API_URL`, `AI_API_KEY`, and `AI_MODEL` are configured for an OpenAI-compatible provider, the backend will use it.

## Security notes

- Passwords are hashed with bcrypt.
- Refresh tokens are HttpOnly cookies and only their SHA-256 hashes are stored in PostgreSQL.
- Access tokens are short-lived and kept in frontend memory.
- Public tracking tokens are cryptographically random; only a hash is stored in the database.
- Public links expire and are revoked automatically when a session is resolved.
- User-owned resources enforce ownership checks.
- Auth/AI endpoints are rate-limited.
- External API keys stay server-side.
- Public tracking endpoints return a deliberately limited data shape.

## Browser location limitation

A web browser cannot guarantee continuous background location updates when the phone is locked or the browser is suspended. A production mobile version should use React Native/native background-location capabilities and can reuse this backend/API design.

## Production improvements

- Replace `prisma db push` with versioned migrations in CI/CD.
- Add Redis for Socket.IO scaling and distributed presence.
- Move missed-check-in processing to a durable queue/worker (BullMQ, SQS, etc.).
- Add real email/SMS/push provider adapters and delivery webhooks.
- Add Google OAuth and verified contact workflows.
- Add data-retention jobs for historical location points.
- Use React Native for dependable background tracking.
- Add integration/e2e tests and observability.
