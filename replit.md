# Sadaqah House Family Charity Tracker

A Muslim family donation tracker for 30 family members to record and verify monthly bKash contributions using a unique decimal-assignment system.

## Run & Operate

- `PORT=8080 pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/charity-tracker run dev` — run the frontend
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL`, `SESSION_SECRET`

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 + express-session
- Frontend: React + Vite + Wouter + TanStack Query
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)
- Real-time: Server-Sent Events (SSE)

## Where things live

- `lib/api-spec/openapi.yaml` — API contract (source of truth)
- `lib/db/src/schema/` — DB schema (users, invoices, donations, reminders)
- `artifacts/api-server/src/routes/` — Express route handlers
- `artifacts/charity-tracker/src/` — React frontend
- `artifacts/api-server/src/lib/sse.ts` — SSE broadcast helper
- `artifacts/api-server/src/lib/auth.ts` — SHA-256 password hashing

## Architecture decisions

- **Decimal-lock system**: Each pending bKash payment gets a unique decimal fraction (0.01–0.40) assigned by scanning all pending invoices. Only 40 slots available simultaneously — more than enough for 30 members.
- **Session auth**: Server-side express-session (cookie-based) for secure auth. Frontend stores user info in localStorage for UI state only.
- **SSE real-time**: `/api/events` endpoint streams `payment_confirmed` events to connected browser clients when the SMS webhook approves a payment.
- **Idempotency**: Past `approved` invoices are never touched again — the decimal lookup only scans `status = 'pending'` rows, so recycled decimals never cause confusion.
- **API server port**: Must run on port 8080 (matches artifact.toml proxy config routing `/api` → `localhost:8080`).

## Product

- **Login** — member usernames like `badsha`, `bithi`, `lova`, etc. Admins: `tahmina` (Tahmina Akhter), `tuli` (Tuli). All passwords: `12345678`
- **Home dashboard** — live progress bar, recent donations, pending invoice alert
- **Donate** — select amount (100–600 BDT = 1–6 months), get unique decimal-tagged amount to transfer
- **History** — personal donation history
- **Members** — family member grid with payment status indicators
- **Admin panel** — full member payment table, revoke invoices, statistics (admin1/admin2 only)
- **SMS webhook** — POST `/api/bkash-sms` with `{ smsText }` body to simulate bKash payment confirmation

## User preferences

- Members: S.M Badsha Miah (user1), Salma Begum Bithi (user2), ..., Asma Akhter (user28)
- Admins: Tahmina Akhter (admin1), Tuli (admin2)
- Donation range: 100–600 BDT only (100 = 1 month, 200 = 2 months, etc.)
- Islamic theme: emerald green (#0F5132), gold (#D4AF37)
- No emojis anywhere in the UI

## Gotchas

- API server MUST run on port 8080 (artifact.toml routes `/api` to 8080 via proxy)
- The dev script defaults `PORT=${PORT:-8080}` — don't set PORT=5000 manually
- After any `lib/db/src/schema/` change: run `pnpm --filter @workspace/db run push` then `pnpm run typecheck:libs`
- After any `lib/api-spec/openapi.yaml` change: run `pnpm --filter @workspace/api-spec run codegen`

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
