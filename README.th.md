[English](README.md) | [ไทย](README.th.md)

# Project Store — ร้านเช่าชุดแต่งงานและระบบนัดหมาย

เว็บไซต์ร้านเช่าชุดแต่งงานพร้อมระบบจองเช่าและนัดหมายออนไลน์ สร้างด้วย [Next.js](https://nextjs.org) (App Router) และ [Supabase](https://supabase.com)

## ภาพรวม

Project Store คือเว็บแอปพลิเคชันแบบ full-stack สำหรับธุรกิจให้เช่าชุดแต่งงานและชุดล้านนา ลูกค้าสามารถเลือกชมแคตตาล็อกชุด ตรวจสอบวันว่างจากปฏิทิน จองเช่าตามช่วงวันที่ และนัดลองชุดหรือปรึกษา ส่วนเจ้าของร้านมีแดชบอร์ดสำหรับจัดการสินค้า หมวดหมู่ การนัดหมาย การจองเช่า การล็อควัน ช่องทางแจ้งเตือน และตั้งค่าร้านทั้งหมด

ระบบตรวจสอบวันว่างอิงตามช่วงวันที่ และถือว่า **สินค้าแต่ละชิ้นมีของจริงเพียง 1 ตัว** กล่าวคือชุดจะไม่ว่างในวันใดก็ตามที่ถูกครอบคลุมด้วยการล็อค การเช่าที่ยังไม่ถูกยกเลิก หรือ (ตามเงื่อนไข) การนัดลองชุด

## ฟีเจอร์

- **แคตตาล็อกสินค้า** — สินค้าพร้อมรูปภาพ ราคาเช่า และเงินมัดจำ รองรับการกรองตามหมวดหมู่ ค้นหา เรียงลำดับ และแบ่งหน้า
- **ปฏิทินวันว่าง** — ปฏิทินรายสินค้าที่แสดงวันว่าง/ไม่ว่าง (รวมการล็อค การเช่า และการนัดหมาย) คลิกที่วันเพื่อดูรายละเอียด
- **การจองเช่า (Rentals)** — ลูกค้าจองชุดตามช่วงวันที่แบบนับรวมวันเริ่มและวันสิ้นสุด ระบบตรวจสอบวันว่างตอนบันทึก และกันการจองทับซ้อนในระดับฐานข้อมูล
- **การนัดหมาย (Appointments)** — จองนัดลองชุด (`try_on`) หรือปรึกษา (`consultation`) ในช่วงเวลาทำการ เลือกผูกกับสินค้าได้
- **ล็อควันจอง (Date locks)** — แอดมินล็อคช่วงวันที่ของสินค้าได้ (เช่น ช่วงซ่อมบำรุง) เพื่อไม่ให้ถูกจอง
- **บัญชีผู้ใช้** — ยืนยันตัวตนด้วยอีเมล/รหัสผ่านผ่าน Supabase Auth จัดการโปรไฟล์ และการแจ้งเตือน
- **แดชบอร์ดแอดมิน** — สถิติและกราฟ (รายได้ย้อนหลัง 30 วัน, การเช่าแยกตามสถานะ, การจอง) พร้อมจัดการสินค้า หมวดหมู่ บริการนัดหมาย การนัดหมาย การเช่า และตั้งค่าร้านแบบครบวงจร
- **หน้าร้าน** — หน้าแรก เกี่ยวกับ ติดต่อ รายการ/รายละเอียดสินค้า โปรไฟล์ และการแจ้งเตือน พร้อมระบบธีม (สว่าง/มืด และสีแบรนด์ที่ตั้งค่าได้)
- **ตั้งค่าแอดมินคนแรก** — ผู้ใช้ที่ล็อกอินคนแรกสามารถรับสิทธิ์แอดมินได้แบบ atomic ผ่าน `/setup-admin`

## เทคโนโลยีที่ใช้

| ส่วน | เทคโนโลยี |
|------|-----------|
| เฟรมเวิร์ก | Next.js (App Router, Server Components, Server Actions) |
| ภาษา | TypeScript |
| ฐานข้อมูลและ Auth | Supabase (PostgreSQL, Auth, Row Level Security, Storage) |
| การเข้าถึงข้อมูล | `@supabase/supabase-js` / `@supabase/ssr` สำหรับคิวรีที่คำนึงถึง RLS และ Drizzle ORM สำหรับ typed query builder |
| Schema / migrations | Supabase SQL migrations (เป็นแหล่งความจริง) + `drizzle-kit` |
| UI | Tailwind CSS, shadcn/ui (สไตล์ New York), Radix UI, ไอคอน lucide-react |
| กราฟ | Recharts |
| แอนิเมชัน | GSAP และคอมโพเนนต์ reveal ที่เขียนเอง |
| ธีม | next-themes ร่วมกับค่าธีมใน `store_settings` |
| การทดสอบ | Vitest |

## โครงสร้างโปรเจกต์

```
app/                     Next.js App Router
  (home)/                หน้าแรก (route group)
  about/ contact/        หน้าเนื้อหาทั่วไป
  products/              รายการสินค้า + หน้ารายละเอียด [slug] พร้อมกรอง/เรียง/แบ่งหน้า
  rentals/               รายการเช่า, new/ (ฟอร์มจอง), [id]/ (รายละเอียด)
  appointments/          รายการนัดหมาย, book/, [id]/ (รายละเอียด)
  notifications/         การแจ้งเตือนของผู้ใช้
  profile/               โปรไฟล์ผู้ใช้
  admin/                 แดชบอร์ดแอดมิน + หน้าจัดการย่อย
    products/ categories/ appointments/ appointment-services/
    rentals/ settings/
  auth/                  login, sign-up, forgot/update password, callback, confirm, error
  setup-admin/           หน้าตั้งค่าแอดมินคนแรก
  api/setup-admin/       Endpoint POST สำหรับรับสิทธิ์แอดมินคนแรก
  api/store-asset/       Proxy ฝั่งเซิร์ฟเวอร์สำหรับเสิร์ฟไฟล์ใน bucket store-assets
  actions/               Server Actions (rentals.ts, appointments.ts)
  layout.tsx             Root layout (ฟอนต์, theme provider, header/footer)
components/              UI ที่ใช้ร่วมกัน (header, footer, การ์ด, ฟอร์ม, เมนู)
  ui/                    พื้นฐานจาก shadcn/ui
hooks/                   React hooks
lib/
  supabase/              client.ts, server.ts, admin.ts, proxy.ts, queries.ts
  db/                    ไคลเอนต์ Drizzle (index.ts) + schema (schema.ts)
  db.types.ts            อินเทอร์เฟซแถวข้อมูลที่เขียนมือ ใช้ใน queries
  date-utils.ts          ตัวช่วยเรื่องช่วงวันที่/การทับซ้อน/วันว่าง
  utils.ts, color.ts     ตัวช่วยอื่น ๆ
supabase/migrations/     SQL migrations (schema, storage, security, cleanup)
public/                  รูปภาพ static (gallery, review, services, wallpaper)
drizzle.config.ts        คอนฟิก drizzle-kit
proxy.ts                 จุดเข้า middleware ของ Next.js (รีเฟรช session + ป้องกันเส้นทาง)
vitest.config.mts        คอนฟิก Vitest
```

## ฐานข้อมูลและ Migrations

**SQL migrations ของ Supabase เป็นแหล่งความจริงของฐานข้อมูล** — ทั้ง RLS
policies, triggers, ฟังก์ชัน `SECURITY DEFINER` และ exclusion constraints
อยู่ใน `supabase/migrations/` ส่วน Drizzle ใช้เป็น typed query builder และ
สำหรับ introspection (`lib/db/schema.ts` สะท้อนโครงสร้าง SQL)

ไฟล์ migration:

| ไฟล์ | หน้าที่ |
|------|---------|
| `00001_initial_schema.sql` | ตาราง, RLS policies, triggers, indexes, ฟังก์ชันตรวจวันว่าง |
| `00002_storage_bucket.sql` | Storage buckets และ policy การเข้าถึง |
| `00003_security_hardening.sql` | ตรึง search_path, กันการยกระดับสิทธิ์, exclusion constraints กันจองซ้ำ, ตั้ง payment-slips เป็นส่วนตัว |
| `00004_drop_stock.sql` | ลบกลไกสต็อกที่ไม่ได้ใช้ (วันว่างอิงช่วงวันที่ล้วน) |

### ตาราง

`profiles`, `categories`, `products`, `product_images`, `store_settings`,
`product_date_locks`, `rentals`, `appointment_services`, `appointments`,
`notifications`

### ฟังก์ชัน SQL ที่สำคัญ (SECURITY DEFINER)

| ฟังก์ชัน | หน้าที่ |
|----------|---------|
| `is_admin()` | ตรวจสิทธิ์แอดมิน ใช้ใน RLS policies |
| `check_product_available(product, start, end, block_appointments)` | คืนค่าว่าสินค้าว่างในช่วงวันที่หรือไม่ |
| `get_product_unavailable_dates(product, start, end, block_from_appointments)` | ขยายช่วงการล็อค + การเช่า (+ การนัดหมาย) เป็นรายวันที่ไม่ว่าง |
| `get_appointments_for_date(date)` | ช่วงเวลาที่ถูกจองแล้วของวันนั้น |
| `get_rentals_for_calendar(product, start, end)` | ข้อมูลการเช่าสำหรับปฏิทินแอดมิน |
| `claim_first_admin()` | เลื่อนผู้เรียกเป็นแอดมินแบบ atomic ถ้ายังไม่มีแอดมิน |
| `guard_profile_role()` / `guard_appointment_update()` | Trigger กันการยกระดับสิทธิ์และการแก้คอลัมน์ที่สงวนไว้ |

### การกันจองทับซ้อน

มี exclusion constraint สองตัวเป็นด่านสุดท้ายในระดับฐานข้อมูลเพื่อกันปัญหา race
condition (ต้องมี extension `btree_gist`):

- `rentals_no_overlap` — การเช่าของสินค้าเดียวกันที่ยังไม่ถูกยกเลิก ห้ามทับซ้อนกัน
- `appointments_no_overlap` — การนัดหมายบริการเดียวกันที่ยังไม่ถูกยกเลิก ห้ามทับซ้อนกัน

### Storage buckets

`product-images`, `payment-slips` (ส่วนตัว), `store-assets`, `category-images`
policy จำกัดการอัปโหลด/แก้ไขไว้ที่แอดมิน และเปิดอ่านให้สาธารณะหรือเจ้าของตามความเหมาะสม

## เริ่มต้นใช้งาน

### สิ่งที่ต้องมี

- Node.js (เวอร์ชันที่ Next.js และ Drizzle รองรับ; `drizzle.config.ts` ใช้ `process.loadEnvFile` ซึ่งมีตั้งแต่ Node 20.12)
- npm
- โปรเจกต์ Supabase (หรือ Supabase CLI สำหรับรันในเครื่อง)

### 1. ติดตั้ง dependencies

```bash
npm install
```

### 2. ตั้งค่า Environment Variables

คัดลอกไฟล์ตัวอย่างแล้วใส่ค่าของ Supabase ของคุณเอง:

```bash
cp .env.example .env.local
```

ดูรายชื่อตัวแปรทั้งหมดได้ที่หัวข้อ [Environment variables](#environment-variables)

### 3. รัน Database Migration

> **สำคัญ:** ต้องรัน migration ก่อนใช้งาน เพราะการตรวจวันว่างพึ่งพาฟังก์ชัน SQL แบบ `SECURITY DEFINER` ที่สร้างในขั้นตอนนี้

รันผ่าน Supabase CLI:

```bash
supabase db push
```

หรือรันไฟล์ตามลำดับใน Supabase Dashboard → SQL Editor:

- `supabase/migrations/00001_initial_schema.sql`
- `supabase/migrations/00002_storage_bucket.sql`
- `supabase/migrations/00003_security_hardening.sql`
- `supabase/migrations/00004_drop_stock.sql`

### 4. รัน Development Server

```bash
npm run dev
```

เปิด [http://localhost:3000](http://localhost:3000)

### 5. ตั้งค่าแอดมินคนแรก

สมัครสมาชิก แล้วเข้า `/setup-admin` เพื่อรับสิทธิ์แอดมินให้บัญชีของคุณ

### Build และทดสอบ

```bash
npm run build   # สร้าง production build
npm run start   # รัน production build
npm run lint    # ตรวจ ESLint
npm test        # รัน unit tests (Vitest)
```

## Environment Variables

ระบุเฉพาะ **ชื่อตัวแปร** — ห้าม commit ค่าจริงลงในโปรเจกต์

| ชื่อ | ขอบเขต | หน้าที่ |
|------|---------|---------|
| `NEXT_PUBLIC_SUPABASE_URL` | Public (เบราว์เซอร์) | URL ของโปรเจกต์ Supabase |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | Public (เบราว์เซอร์) | Anon/publishable key ของ Supabase |
| `SUPABASE_SERVICE_ROLE_KEY` | ฝั่งเซิร์ฟเวอร์เท่านั้น | ไคลเอนต์ Supabase สิทธิ์พิเศษ (`lib/supabase/admin.ts`) |
| `DATABASE_URL` | ฝั่งเซิร์ฟเวอร์เท่านั้น | Connection string ของ Postgres สำหรับ Drizzle (`lib/db`); ใช้ transaction pooler (พอร์ต 6543) สำหรับ runtime ของแอป |
| `DIRECT_URL` | ฝั่งเซิร์ฟเวอร์เท่านั้น | URI สำรองของ Postgres สำหรับ `drizzle-kit` (session pooler / direct พอร์ต 5432) |
| `VERCEL_URL` | ฝั่งเซิร์ฟเวอร์ (อัตโนมัติ) | ตั้งอัตโนมัติบน Vercel ใช้สร้าง URL ของเว็บสำหรับ metadata |

ดูเทมเพลตพร้อมคำอธิบายได้ใน `.env.example`

## การใช้งาน

### ลูกค้า

- เลือกชมชุดที่ `/products` และเปิดรายละเอียดที่ `/products/[slug]`
- นัดลองชุดที่ `/appointments/book` หรือเริ่มจองเช่าที่ `/rentals/new` หลังตรวจปฏิทินวันว่าง
- ติดตามการเช่าที่ `/rentals` และการนัดหมายที่ `/appointments` จัดการบัญชีที่ `/profile`

### แอดมิน

- เปิด `/admin` เพื่อดูแดชบอร์ด (รายได้ กราฟการเช่า/การจอง สินค้ายอดนิยม)
- จัดการสินค้า หมวดหมู่ บริการนัดหมาย การนัดหมาย การเช่า และตั้งค่าร้านจากแถบเมนูแอดมิน
- ล็อควันไม่ว่างรายสินค้าได้จากหน้าจัดการสินค้า

### Scripts

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `npm run dev` | รัน dev server |
| `npm run build` | build production |
| `npm run start` | รัน production build |
| `npm run lint` | ตรวจ ESLint |
| `npm test` | รัน unit tests (Vitest) |
| `npm run db:pull` | ดึงโครงสร้างจาก DB จริงด้วย drizzle-kit |
| `npm run db:generate` | สร้าง Drizzle migrations |
| `npm run db:migrate` | รัน Drizzle migrations |
| `npm run db:push` | push Drizzle schema ขึ้น DB |
| `npm run db:studio` | เปิด Drizzle Studio |

## หมายเหตุด้านสถาปัตยกรรม

### การตรวจสอบวันว่าง

- **ฝั่ง UI** — `get_product_unavailable_dates` (RPC) เพื่อแสดงวันที่ไม่ว่างในปฏิทิน
- **ฝั่ง Server (ตอนบันทึก)** — `check_product_available` (RPC) ตรวจอีกครั้งก่อนบันทึก โดยมี exclusion constraints เป็นด่านสุดท้าย
- **Rental** ใช้ `block_appointments = true` (นัดลองชุดก็บล็อควันเช่าได้)
- **Appointment** ใช้ `block_appointments = false` (จองหลายช่วงในวันเดียวกันได้)

### Proxy `/api/store-asset`

รูปโลโก้ร้าน / QR พร้อมเพย์ถูกเก็บใน bucket `store-assets` และเสิร์ฟผ่าน API route:

```
/api/store-asset?path=<relative-path>
```

Route จะตรวจสอบ path ด้วยรูปแบบคีย์ที่เข้มงวด (ห้ามมี `..` และจำกัดความยาว) อ่านไฟล์ฝั่งเซิร์ฟเวอร์ แล้วคืน response พร้อม cache 1 ปีแบบ immutable

### Middleware `proxy.ts`

`proxy.ts` (ทำงานร่วมกับ `lib/supabase/proxy.ts`) จะรีเฟรช session ของ
Supabase ทุกคำขอ และ redirect ผู้ใช้ที่ยังไม่ล็อกอินออกจากเส้นทางที่ต้องป้องกัน
(`/appointments`, `/profile`, `/admin`, `/notifications`)

### การเสริมความปลอดภัย

Migration `00003` ตรึง `search_path` ให้ทุกฟังก์ชัน `SECURITY DEFINER`
กันการยกระดับสิทธิ์ผ่านโปรไฟล์ ห้ามลูกค้าแก้คอลัมน์สงวนของการนัดหมาย
และตั้ง bucket `payment-slips` เป็นแบบส่วนตัว

## การทดสอบ

Tests อยู่ที่ `lib/__tests__/` ครอบคลุมฟังก์ชัน pure ใน `lib/date-utils.ts`
(การสร้างช่วงวันที่ การตรวจ overlap การนับจำนวนวันเช่าแบบนับรวม และการจัดรูปแบบวันที่)

```bash
npm test
```

## หมายเหตุ

- ระบบวันว่างถือว่าสินค้าแต่ละชิ้นมีของจริงเพียง 1 ตัว; `stock_qty` และ RPC ด้านสต็อกถูกลบใน migration `00004`
- Drizzle ไม่ได้อ่าน `.env.local` เอง; `drizzle.config.ts` โหลดผ่าน `process.loadEnvFile`
- ตั้ง `DATABASE_URL` เป็น transaction pooler (พอร์ต 6543 พร้อม `prepare: false`) สำหรับ runtime ของแอป และใช้ session pooler / direct URI (พอร์ต 5432) สำหรับ DDL ของ `drizzle-kit`
- `next` ถูกปักหมุดเป็น `latest` ใน `package.json`; ก่อนพึ่งพา API ของเฟรมเวิร์กให้ตรวจ `node_modules/next/dist/docs/` เพราะเวอร์ชันนี้อาจต่างจาก Next.js แบบเดิม (ดู `AGENTS.md`)
- TODO: รายละเอียดการ deploy ขึ้น production (แพลตฟอร์มโฮสต์, CI) ยังไม่ได้ระบุไว้ในเอกสารนี้
