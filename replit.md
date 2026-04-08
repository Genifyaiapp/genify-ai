# Genify AI — Full-Stack AI SaaS

## Overview

A premium, dark-themed AI SaaS platform built on a pnpm monorepo. Users can sign up, generate AI content (OpenAI via Replit AI Integrations proxy), and upgrade to Pro via Stripe.

## Stack

- **Monorepo**: pnpm workspaces, TypeScript 5.9, Node.js 24
- **Frontend**: React + Vite + Tailwind CSS + shadcn/ui + Framer Motion (artifact: `genify-ai`)
- **API**: Express 5, JWT auth (HTTP-only cookies), SSE streaming (artifact: `api-server`)
- **Database**: PostgreSQL + Drizzle ORM (package: `@workspace/db`)
- **AI**: OpenAI GPT via Replit AI Integrations proxy (no user API key needed)
- **Payments**: Stripe (checkout sessions + webhook)
- **Auth**: bcryptjs passwords, JWT tokens in HTTP-only cookies (`auth_token`, 7-day)

## Artifacts

| Artifact | Path | Port |
|---|---|---|
| `genify-ai` | `/` | `$PORT` (22863 dev) |
| `api-server` | `/api` | 8080 |
| `mockup-sandbox` | `/__mockup` | 8081 |

## Key API Routes

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/api/auth/signup` | No | Create account |
| POST | `/api/auth/login` | No | Login |
| POST | `/api/auth/logout` | No | Logout |
| GET | `/api/user` | Yes | Get current user |
| POST | `/api/generate` | Yes | SSE streaming AI content (deducts 1 credit) |
| POST | `/api/upgrade` | Yes | Create Stripe checkout session |
| POST | `/api/upgrade/webhook` | No | Stripe webhook (upgrades plan) |
| POST | `/api/demo` | No | Public SSE demo (no auth, no credits) |

## Database Schema

`users` table (`@workspace/db/lib/db/src/schema/index.ts`):
- `id` — UUID primary key
- `name` — text
- `email` — text unique
- `hashedPassword` — text
- `plan` — text default "free" (`"free"` | `"pro"`)
- `credits` — integer default 10 (free=10, pro=-1=unlimited)
- `stripeCustomerId` — text nullable
- `stripeSessionId` — text nullable
- `createdAt` — timestamp

## Credit System

- Free plan: 10 credits, deducted by 1 on each `/api/generate` call
- Pro plan: credits=-1 sentinel (bypass check = unlimited)
- Credits displayed in sidebar with progress bar and warning when ≤2

## Environment Variables / Secrets

- `SESSION_SECRET` — JWT signing secret
- `AI_INTEGRATIONS_OPENAI_BASE_URL` — Replit AI proxy base URL
- `AI_INTEGRATIONS_OPENAI_API_KEY` — Replit AI proxy key
- `STRIPE_SECRET_KEY` — Stripe secret (optional; upgrade returns 503 if missing)
- `STRIPE_WEBHOOK_SECRET` — Stripe webhook secret (optional)
- `STRIPE_PRO_PRICE_ID` — Stripe price ID for Pro plan

## Frontend Routes

- `/` — Landing page with live AI demo (public)
- `/login` — Login page
- `/signup` — Signup page
- `/dashboard` — Dashboard (protected, redirects to /login if not authed)

## Key Commands

- `pnpm run typecheck` — full typecheck
- `pnpm --filter @workspace/db run push` — push DB schema (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server
- `pnpm --filter @workspace/genify-ai run dev` — run frontend

## Design System

- Background: `#0F172A`, Cards: `#1E293B`
- Primary: `#6366F1` (indigo), Accent: `#8B5CF6` (purple)
- Text: `#E2E8F0`, Muted: `#94A3B8`, Border: `white/5`
- Font: Inter, No emojis in UI
