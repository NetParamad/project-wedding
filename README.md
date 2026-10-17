[English](README.md) | [ไทย](README.th.md)

# Project Store — Wedding Dress Rental & Appointments

A wedding-dress rental shop website with an online booking and appointment system, built with [Next.js](https://nextjs.org) (App Router) and [Supabase](https://supabase.com).

## Overview

Project Store is a full-stack web application for a wedding-dress / Lanna-costume rental business. Customers browse a catalog of dresses, check real-time availability on a calendar, book rentals for a date range, and schedule fitting or consultation appointments. Shop owners get an admin dashboard to manage products, categories, appointments, rentals, date locks, notification flows, and store-wide settings.

The availability model is date-range based and assumes **one physical unit per product**: a product is unavailable on any date that is covered by a date lock, an active rental, or (optionally) a fitting appointment.

## Features

- **Product catalog** — products with images, rental price, and deposit; category filtering, search, sorting, and pagination.
- **Availability calendar** — per-product calendar showing free/unavailable dates (merges date locks, rentals, and appointments); clicking a date reveals the details.
- **Rentals** — customers book a dress for an inclusive start/end date range; the system validates availability on submit and blocks overlapping bookings at the database level.
- **Appointments** — book a fitting (`try_on`) or consultation (`consultation`) in a business-hours time slot, optionally linked to a product.
- **Date locks** — admins can lock a date range for a product (e.g. maintenance) so it cannot be booked.
- **User accounts** — email/password authentication via Supabase Auth, profile management, and notifications.
- **Admin dashboard** — metrics and charts (revenue over the last 30 days, rentals by status, bookings), plus full CRUD for products, categories, appointment services, appointments, rentals, and store settings.
- **Storefront** — home, about, contact, product listing/detail, profile, and notifications pages, with a theme system (light/dark plus configurable brand color/theme).
- **First-admin bootstrap** — the first authenticated user can claim the admin role atomically via `/setup-admin`.

## Tech Stack

| Area | Choice |
|------|--------|
| Framework | Next.js (App Router, Server Components, Server Actions) |
| Language | TypeScript |
| Database & Auth | Supabase (PostgreSQL, Auth, Row Level Security, Storage) |
| Data access | `@supabase/supabase-js` / `@supabase/ssr` for RLS-aware queries, plus Drizzle ORM for a typed query builder |
| Schema / migrations | Supabase SQL migrations (source of truth) + `drizzle-kit` |
| UI | Tailwind CSS, shadcn/ui (New York style), Radix UI primitives, lucide-react icons |
| Charts | Recharts |
| Animation | GSAP, custom reveal components |
| Theming | next-themes + `store_settings` theme config |
| Testing | Vitest |

## Project Structure

```
app/                     Next.js App Router
  (home)/                Landing page (route group)
  about/ contact/        Static marketing pages
  products/              Catalog list + [slug] detail, filter/sort/pagination
  rentals/               Rental list, new/ (booking form), [id]/ (detail)
  appointments/          Appointment list, book/, [id]/ (detail)
  notifications/         User notifications
  profile/               User profile
  admin/                 Admin dashboard + nested management pages
    products/ categories/ appointments/ appointment-services/
    rentals/ settings/
  auth/                  login, sign-up, forgot/update password, callback, confirm, error
  setup-admin/           First-admin bootstrap page
  api/setup-admin/       POST endpoint that claims the first admin
  api/store-asset/       Server-side proxy that serves the store-assets bucket
  actions/               Server Actions (rentals.ts, appointments.ts)
  layout.tsx             Root layout (fonts, theme providers, header/footer)
components/              Shared UI (header, footer, cards, forms, nav)
  ui/                    shadcn/ui primitives
hooks/                   React hooks
lib/
  supabase/              client.ts, server.ts, admin.ts, proxy.ts, queries.ts
  db/                    Drizzle client (index.ts) + schema (schema.ts)
  db.types.ts            Hand-written row interfaces used by queries
  date-utils.ts          Date range / overlap / availability helpers
  utils.ts, color.ts     Misc helpers
supabase/migrations/     SQL migrations (schema, storage, security, cleanup)
public/                  Static images (gallery, review, services, wallpaper)
drizzle.config.ts        drizzle-kit config
proxy.ts                 Next.js middleware entry (session refresh + route guard)
vitest.config.mts        Vitest config
```

## Database & Migrations

The **Supabase SQL migrations are the source of truth** for the database — RLS
policies, triggers, `SECURITY DEFINER` functions, and exclusion constraints all
live in `supabase/migrations/`. Drizzle is used for a typed query builder and
introspection (`lib/db/schema.ts` mirrors the SQL).

Migration files:

| File | Purpose |
|------|---------|
| `00001_initial_schema.sql` | Tables, RLS policies, triggers, indexes, availability functions |
| `00002_storage_bucket.sql` | Storage buckets and their access policies |
| `00003_security_hardening.sql` | search_path pinning, privilege-escalation guards, double-booking exclusion constraints, private payment-slips |
| `00004_drop_stock.sql` | Removes the unused stock mechanism (availability is purely date-range based) |

### Tables

`profiles`, `categories`, `products`, `product_images`, `store_settings`,
`product_date_locks`, `rentals`, `appointment_services`, `appointments`,
`notifications`.

### Key SQL functions (SECURITY DEFINER)

| Function | Purpose |
|----------|---------|
| `is_admin()` | Admin check used by RLS policies |
| `check_product_available(product, start, end, block_appointments)` | Returns whether a product is free in a range |
| `get_product_unavailable_dates(product, start, end, block_from_appointments)` | Expands locks + rentals (+ appointments) into individual unavailable dates |
| `get_appointments_for_date(date)` | Booked time slots for a day |
| `get_rentals_for_calendar(product, start, end)` | Rental rows for the admin calendar |
| `claim_first_admin()` | Atomically promotes the caller to admin if none exists yet |
| `guard_profile_role()` / `guard_appointment_update()` | Triggers preventing privilege escalation and protected-column edits |

### Double-booking protection

Two exclusion constraints provide a database-level backstop against race
conditions (requires the `btree_gist` extension):

- `rentals_no_overlap` — no two non-cancelled rentals of the same product may overlap.
- `appointments_no_overlap` — no two non-cancelled appointments for the same service may overlap.

### Storage buckets

`product-images`, `payment-slips` (private), `store-assets`, `category-images`.
Policies scope uploads/edits to admins and reads to public or owner as appropriate.

## Getting Started

### Prerequisites

- Node.js (a version supported by Next.js and Drizzle; `drizzle.config.ts` relies on `process.loadEnvFile`, available from Node 20.12)
- npm
- A Supabase project (or the Supabase CLI for local development)

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Copy the example file and fill in your own Supabase values:

```bash
cp .env.example .env.local
```

See [Environment variables](#environment-variables) for the full list of names.

### 3. Run the database migrations

> **Important:** you must run the migrations before using the app — availability checks rely on `SECURITY DEFINER` SQL functions created there.

Either push with the Supabase CLI:

```bash
supabase db push
```

or run the files in order in the Supabase Dashboard → SQL Editor:

- `supabase/migrations/00001_initial_schema.sql`
- `supabase/migrations/00002_storage_bucket.sql`
- `supabase/migrations/00003_security_hardening.sql`
- `supabase/migrations/00004_drop_stock.sql`

### 4. Run the development server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### 5. Create the first admin

Sign up, then visit `/setup-admin` to claim the admin role for your account.

### Build & test

```bash
npm run build   # production build
npm run start   # serve the production build
npm run lint    # ESLint
npm test        # Vitest unit tests
```

## Environment Variables

Variable **names** only — never commit real values.

| Name | Scope | Purpose |
|------|-------|---------|
| `NEXT_PUBLIC_SUPABASE_URL` | Public (browser) | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Public (browser) | Supabase anon/publishable key |
| `SUPABASE_SERVICE_ROLE_KEY` | Server only | Privileged Supabase client (`lib/supabase/admin.ts`) |
| `DATABASE_URL` | Server only | Postgres connection string for Drizzle (`lib/db`); use the transaction pooler (port 6543) for the app runtime |
| `DIRECT_URL` | Server only | Fallback Postgres URI for `drizzle-kit` (session pooler / direct, port 5432) |
| `VERCEL_URL` | Server (auto) | Set automatically on Vercel; used to derive the site URL for metadata |

See `.env.example` for the annotated template.

## Usage

### Customers

- Browse dresses at `/products` and open a product at `/products/[slug]`.
- Book a fitting at `/appointments/book`, or start a rental at `/rentals/new` after checking the calendar.
- Track rentals at `/rentals` and appointments at `/appointments`; manage your account at `/profile`.

### Admins

- Open `/admin` for the dashboard (revenue, rental/booking charts, top products).
- Manage products, categories, appointment services, appointments, rentals, and store settings from the admin sidebar.
- Lock unavailable dates per product from the product edit page.

### Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start the dev server |
| `npm run build` | Production build |
| `npm run start` | Serve the production build |
| `npm run lint` | Run ESLint |
| `npm test` | Run Vitest unit tests |
| `npm run db:pull` | Introspect the live DB with drizzle-kit |
| `npm run db:generate` | Generate Drizzle migrations |
| `npm run db:migrate` | Apply Drizzle migrations |
| `npm run db:push` | Push Drizzle schema to the DB |
| `npm run db:studio` | Open Drizzle Studio |

## Architecture Notes

### Availability checks

- **UI** — `get_product_unavailable_dates` (RPC) fills the calendar with unavailable dates.
- **Server (on submit)** — `check_product_available` (RPC) re-validates before saving; the exclusion constraints are the final backstop.
- **Rental** uses `block_appointments = true` (a fitting appointment also blocks the rental date).
- **Appointment** uses `block_appointments = false` (multiple appointments can share a day).

### `/api/store-asset` proxy

Store logo / PromptPay QR images live in the `store-assets` bucket and are served through an API route:

```
/api/store-asset?path=<relative-path>
```

The route validates the path against a strict key pattern (no `..`, length-capped), reads the object server-side, and returns it with a one-year immutable cache header.

### `proxy.ts` middleware

`proxy.ts` (backed by `lib/supabase/proxy.ts`) refreshes the Supabase session on
every request and redirects unauthenticated users away from protected paths
(`/appointments`, `/profile`, `/admin`, `/notifications`).

### Security hardening

Migration `00003` pins `search_path` on every `SECURITY DEFINER` function,
guards against profile role escalation, blocks customers from editing
privileged appointment columns, and makes the `payment-slips` bucket private.

## Testing

Tests live in `lib/__tests__/` and cover the pure helpers in
`lib/date-utils.ts` (date-range generation, overlap detection, inclusive rental
day counting, date formatting).

```bash
npm test
```

## Notes

- The availability model assumes exactly one physical unit per product; `stock_qty` and the stock RPCs were removed in migration `00004`.
- Drizzle does not read `.env.local` on its own; `drizzle.config.ts` loads it via `process.loadEnvFile`.
- Point `DATABASE_URL` at the transaction pooler (port 6543, with `prepare: false`) for the app runtime; use the session pooler / direct URI (port 5432) for `drizzle-kit` DDL.
- `next` is pinned to `latest` in `package.json`; check `node_modules/next/dist/docs/` before relying on framework APIs, as this version may differ from older Next.js conventions (see `AGENTS.md`).
- TODO: production deployment details (hosting target, CI) are not documented here.
