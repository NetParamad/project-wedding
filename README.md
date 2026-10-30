[EN](README.md) | [TH](README.th.md)

<h1 align="center">Project Store — Wedding Dress Rental</h1>

<p align="center">
  A wedding-dress and Lanna-costume rental site with online booking, availability calendars, and an admin dashboard.
</p>

<p align="center">
  <a href="#overview">Overview</a> ·
  <a href="#features">Features</a> ·
  <a href="#tech-stack">Tech stack</a> ·
  <a href="#project-structure">Structure</a> ·
  <a href="#supabase-setup">Supabase</a> ·
  <a href="#getting-started">Getting started</a> ·
  <a href="#environment-variables">Environment</a> ·
  <a href="#usage">Usage</a> ·
  <a href="#notes">Notes</a>
</p>

---

## Overview

Project Store is a full-stack web application for a wedding-dress rental business. Customers browse a catalog, check availability on a per-product calendar, book a rental for an inclusive date range, and schedule fitting or consultation appointments. Shop owners get an admin dashboard for products, categories, appointment services, appointments, rentals, date locks, notifications, and store settings.

Availability is date-range based and assumes **one physical unit per product**: a product is unavailable on any date covered by a date lock, a non-cancelled rental, or (optionally) a fitting appointment. Database exclusion constraints back up the application-level checks against double booking.

## Features

- **Product catalog** — images, rental price, deposit, category filtering, search, sorting, and pagination (12 products per page).
- **Availability calendar** — per-product free/unavailable dates merging date locks, rentals, and appointments; clicking a date reveals details.
- **Rentals** — book an inclusive start/end range with pickup or delivery, validate availability on submit, and block overlaps at the database level.
- **Appointments** — book a fitting (`try_on`) or consultation (`consultation`) in 09:00–18:00 slots, optionally linked to a product.
- **Date locks** — admins lock a date range for a product (for example maintenance) so it cannot be booked.
- **User accounts** — email/password authentication through Supabase Auth, profile management, and in-app notifications.
- **Admin dashboard** — totals for rentals, revenue, pending rentals, and products, plus revenue, rental-status, booking, and booking-status charts and top products.
- **Admin CRUD** — products, categories, appointment services, appointments, rentals, and store settings.
- **Storefront** — home, about, contact, products, rentals, appointments, profile, and notifications, with light/dark themes and a configurable brand color.
- **First-admin bootstrap** — `/setup-admin` promotes the first authenticated user atomically through `claim_first_admin()`.

## Tech stack

| Area | Choice |
| --- | --- |
| Framework | [Next.js](https://nextjs.org) App Router, Server Components, Server Actions (`next: "latest"`) |
| Language | TypeScript |
| Database & Auth | [Supabase](https://supabase.com) (PostgreSQL, Auth, Row Level Security, Storage) |
| Data access | `@supabase/supabase-js` / `@supabase/ssr` for RLS-aware queries + [Drizzle ORM](https://orm.drizzle.team) for typed queries |
| Schema / migrations | Supabase SQL migrations (source of truth) + `drizzle-kit` |
| UI | Tailwind CSS v3, shadcn/ui (New York), Radix UI, lucide-react |
| Charts | Recharts |
| Animation | GSAP + custom reveal components |
| Theming | next-themes + theme values in `store_settings` |
| Testing | Vitest |
| Package manager | npm (`package-lock.json`) |

## Project structure

```
app/
  (home)/                   landing page (route group)
  about/ contact/           marketing pages
  products/                 catalog list + [slug] detail, filter/sort/pagination
  rentals/                  list, new/ booking form, [id] detail
  appointments/             list, book/, [id] detail
  notifications/            user notifications
  profile/                  user profile
  admin/                    dashboard + products, categories, appointments,
                            appointment-services, rentals, settings
  auth/                     login, sign-up, forgot/update password, callback, confirm, error
  setup-admin/              first-admin bootstrap page
  api/setup-admin/          POST endpoint that claims the first admin
  api/store-asset/          server proxy for the store-assets bucket
  actions/                  Server Actions (rentals.ts, appointments.ts)
  layout.tsx                root layout: fonts, theme, header/footer
components/
  ui/                       shadcn/ui primitives
  header.tsx footer.tsx     shell
  product-card.tsx          catalog card
  date-picker.tsx           availability-aware date picker
  realtime-chat.tsx         realtime chat component
hooks/                      React hooks
lib/
  supabase/                 client.ts, server.ts, admin.ts, proxy.ts, queries.ts
  db/                       Drizzle client (index.ts) + schema (schema.ts)
  db.types.ts               hand-written row interfaces
  date-utils.ts             date range, overlap, rental-day helpers
  __tests__/                Vitest tests for date-utils
supabase/migrations/        SQL schema, storage, security, cleanup
public/                     gallery, review, services, wallpaper images
drizzle.config.ts           drizzle-kit config
proxy.ts                    session refresh + route guard
vitest.config.mts           Vitest config
```

## Supabase setup

The **Supabase SQL migrations are the source of truth** for the database — RLS policies, triggers, `SECURITY DEFINER` functions, and exclusion constraints live in `supabase/migrations/`. Drizzle mirrors the schema in `lib/db/schema.ts` for typed queries and introspection.

| File | Purpose |
| --- | --- |
| `00001_initial_schema.sql` | Tables, RLS policies, triggers, indexes, availability functions |
| `00002_storage_bucket.sql` | Storage buckets and access policies |
| `00003_security_hardening.sql` | `search_path` pinning, privilege-escalation guards, double-booking constraints, private payment slips |
| `00004_drop_stock.sql` | Removes the unused stock mechanism |

Tables: `profiles`, `categories`, `products`, `product_images`, `store_settings`, `product_date_locks`, `rentals`, `appointment_services`, `appointments`, `notifications`.

Key SQL functions: `is_admin()`, `check_product_available(product, start, end, block_appointments)`, `get_product_unavailable_dates(product, start, end, block_appointments)`, `get_appointments_for_date(date)`, `get_rentals_for_calendar(product, start, end)`, `claim_first_admin()`, plus the `guard_profile_role()` and `guard_appointment_update()` triggers.

Two exclusion constraints (requiring `btree_gist`) prevent overlaps: `rentals_no_overlap` for non-cancelled rentals of the same product and `appointments_no_overlap` for non-cancelled appointments of the same service. Storage buckets are `product-images`, `payment-slips` (private), `store-assets`, and `category-images`.

## Getting started

Requirements: **Node.js >= 22.23.4** (`.nvmrc` pins `24.22.1`), **npm**, and a Supabase project (or the Supabase CLI for local development).

```bash
git clone https://github.com/NetParamad/project-wedding.git
cd project-wedding
npm install
cp .env.example .env.local   # fill in your Supabase values
```

Run the migrations **before using the app** — availability checks rely on the `SECURITY DEFINER` functions. Either push with the CLI:

```bash
supabase db push
```

or run the four files in order in the Supabase Dashboard → SQL Editor:

- `supabase/migrations/00001_initial_schema.sql`
- `supabase/migrations/00002_storage_bucket.sql`
- `supabase/migrations/00003_security_hardening.sql`
- `supabase/migrations/00004_drop_stock.sql`

Then start the dev server and create the first admin:

```bash
npm run dev                  # http://localhost:3000
```

Sign up, then visit `/setup-admin` to claim the admin role for your account.

### Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Dev server at `http://localhost:3000` |
| `npm run build` | Production build |
| `npm run start` | Serve the production build |
| `npm run lint` | Run ESLint |
| `npm test` | Vitest unit tests |
| `npm run db:pull` | Introspect the live database with drizzle-kit |
| `npm run db:generate` | Generate Drizzle migrations |
| `npm run db:migrate` | Apply Drizzle migrations |
| `npm run db:push` | Push the Drizzle schema to the database |
| `npm run db:studio` | Open Drizzle Studio |

## Environment variables

Copy `.env.example` to `.env.local` and set (names only — never commit real values):

| Variable | Required | Notes |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | ✅ | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | ✅ | Supabase publishable (or legacy anon) key |
| `SUPABASE_SERVICE_ROLE_KEY` | ✅ for admin | Server-only privileged client in `lib/supabase/admin.ts`; never expose to the browser |
| `DATABASE_URL` | ✅ for Drizzle | Postgres connection string for `lib/db`; use the transaction pooler (port 6543) at runtime |

The code also reads `DIRECT_URL` as a drizzle-kit fallback and `VERCEL_URL` (set automatically on Vercel) for `metadataBase`; neither is listed in `.env.example`. `drizzle.config.ts` loads `.env.local` through `process.loadEnvFile`.

## Usage

1. **Browse the catalog** — `/products` supports search, category filters, sorting, and pagination; `/products/[slug]` shows the detail and availability calendar.
2. **Check availability** — the calendar merges date locks, rentals, and (optionally) appointments into unavailable dates.
3. **Book a rental** — `/rentals/new` selects a product and inclusive date range, validates availability, and records pickup or delivery details.
4. **Book an appointment** — `/appointments/book` picks a service, date, and available 09:00–18:00 slot, optionally linked to a product.
5. **Track activity** — `/rentals`, `/appointments`, and `/notifications` show the signed-in user's records; `/profile` manages the account.
6. **Administer** — `/admin` shows dashboard metrics and charts; the sidebar manages products, categories, appointment services, appointments, rentals, and settings. Lock dates from the product edit page.
7. **Bootstrap the first admin** — the first authenticated user can claim the role at `/setup-admin`.

## Notes

- **Availability model** — exactly one physical unit per product. `stock_qty` and the stock RPCs were removed in migration `00004`; availability is purely date-range based.
- **Rental validation** — the UI calls `get_product_unavailable_dates`, the server re-checks with `check_product_available` on submit, and the exclusion constraints are the final backstop. Rentals use `block_appointments = true`; appointments use `false`.
- **`/api/store-asset`** — serves logo and PromptPay QR images from the `store-assets` bucket via `/api/store-asset?path=<relative-path>`, validating the key against a strict pattern and returning a one-year immutable cache header.
- **`proxy.ts`** — refreshes the Supabase session on every request and redirects unauthenticated users away from `/appointments`, `/profile`, `/admin`, and `/notifications`.
- **Pooler ports** — point `DATABASE_URL` at the transaction pooler (6543) for the app runtime; use the session pooler or direct URI (5432) for drizzle-kit DDL.
- **Tests** — `lib/__tests__/date-utils.test.ts` covers date-range generation, overlap detection, inclusive rental-day counting, and date formatting.
- **Next.js version** — `next` is pinned to `latest`; check `node_modules/next/dist/docs/` before relying on framework APIs (see [`AGENTS.md`](AGENTS.md)).
- **Deployment** — no hosting target or CI workflow is documented in the repository.
- **License** — no `LICENSE` file is present in the repository.
