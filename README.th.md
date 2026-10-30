[EN](README.md) | [TH](README.th.md)

<h1 align="center">Project Store — ร้านเช่าชุดแต่งงาน</h1>

<p align="center">
  เว็บไซต์ร้านเช่าชุดแต่งงานและชุดล้านนา พร้อมระบบจองออนไลน์ ปฏิทินวันว่าง และแดชบอร์ดแอดมิน
</p>

<p align="center">
  <a href="#ภาพรวม">ภาพรวม</a> ·
  <a href="#ฟีเจอร์">ฟีเจอร์</a> ·
  <a href="#เทคโนโลยีที่ใช้">เทคโนโลยี</a> ·
  <a href="#โครงสร้างโปรเจกต์">โครงสร้าง</a> ·
  <a href="#การตั้งค่า-supabase">Supabase</a> ·
  <a href="#เริ่มต้นใช้งาน">เริ่มต้นใช้งาน</a> ·
  <a href="#ตัวแปรสภาพแวดล้อม">ตัวแปรสภาพแวดล้อม</a> ·
  <a href="#วิธีใช้งาน">วิธีใช้งาน</a> ·
  <a href="#หมายเหตุ">หมายเหตุ</a>
</p>

---

## ภาพรวม

Project Store คือเว็บแอปพลิเคชันแบบ full-stack สำหรับธุรกิจให้เช่าชุดแต่งงาน ลูกค้าสามารถเลือกชมแคตตาล็อก ตรวจสอบวันว่างจากปฏิทินรายสินค้า จองเช่าตามช่วงวันที่แบบนับรวมวันเริ่มและวันสิ้นสุด และนัดลองชุดหรือปรึกษา ส่วนเจ้าของร้านมีแดชบอร์ดสำหรับจัดการสินค้า หมวดหมู่ บริการนัดหมาย การนัดหมาย การเช่า การล็อควัน การแจ้งเตือน และตั้งค่าร้าน

ระบบวันว่างอิงตามช่วงวันที่และถือว่า **สินค้าแต่ละชิ้นมีของจริงเพียง 1 ตัว** กล่าวคือชุดจะไม่ว่างในวันใดก็ตามที่ถูกครอบคลุมด้วยการล็อค การเช่าที่ยังไม่ถูกยกเลิก หรือ (ตามเงื่อนไข) การนัดลองชุด และมี exclusion constraint ในฐานข้อมูลเป็นด่านสุดท้ายกันการจองทับซ้อน

## ฟีเจอร์

- **แคตตาล็อกสินค้า** — รูปภาพ ราคาเช่า เงินมัดจำ การกรองตามหมวดหมู่ ค้นหา เรียงลำดับ และแบ่งหน้า (หน้าละ 12 รายการ)
- **ปฏิทินวันว่าง** — ปฏิทินรายสินค้าที่รวมการล็อค การเช่า และการนัดหมายเป็นวันที่ไม่ว่าง คลิกที่วันเพื่อดูรายละเอียด
- **การจองเช่า (Rentals)** — จองตามช่วงวันที่แบบนับรวมวันเริ่มและวันสิ้นสุด เลือกรับเองหรือจัดส่ง ตรวจวันว่างตอนบันทึก และกันการจองทับซ้อนในระดับฐานข้อมูล
- **การนัดหมาย (Appointments)** — จองนัดลองชุด (`try_on`) หรือปรึกษา (`consultation`) ในช่วงเวลา 09:00–18:00 เลือกผูกกับสินค้าได้
- **ล็อควันจอง (Date locks)** — แอดมินล็อคช่วงวันที่ของสินค้าได้ (เช่น ช่วงซ่อมบำรุง) เพื่อไม่ให้ถูกจอง
- **บัญชีผู้ใช้** — ยืนยันตัวตนด้วยอีเมล/รหัสผ่านผ่าน Supabase Auth จัดการโปรไฟล์ และรับการแจ้งเตือนในแอป
- **แดชบอร์ดแอดมิน** — สรุปรายการเช่า รายได้ รายการรอดำเนินการ และสินค้า พร้อมกราฟรายได้ สถานะการเช่า การจอง สถานะการจอง และสินค้ายอดนิยม
- **CRUD สำหรับแอดมิน** — สินค้า หมวดหมู่ บริการนัดหมาย การนัดหมาย การเช่า และตั้งค่าร้าน
- **หน้าร้าน** — หน้าแรก เกี่ยวกับ ติดต่อ สินค้า การเช่า การนัดหมาย โปรไฟล์ และการแจ้งเตือน พร้อมธีมสว่าง/มืดและสีแบรนด์ที่ตั้งค่าได้
- **ตั้งค่าแอดมินคนแรก** — `/setup-admin` เลื่อนผู้ใช้ที่ล็อกอินคนแรกเป็นแอดมินแบบ atomic ผ่าน `claim_first_admin()`

## เทคโนโลยีที่ใช้

| ด้าน | ที่เลือกใช้ |
| --- | --- |
| เฟรมเวิร์ก | [Next.js](https://nextjs.org) App Router, Server Components, Server Actions (`next: "latest"`) |
| ภาษา | TypeScript |
| ฐานข้อมูลและ Auth | [Supabase](https://supabase.com) (PostgreSQL, Auth, Row Level Security, Storage) |
| การเข้าถึงข้อมูล | `@supabase/supabase-js` / `@supabase/ssr` สำหรับคิวรีที่คำนึงถึง RLS + [Drizzle ORM](https://orm.drizzle.team) สำหรับ typed query |
| Schema / migrations | Supabase SQL migrations (แหล่งความจริง) + `drizzle-kit` |
| UI | Tailwind CSS v3, shadcn/ui (New York), Radix UI, lucide-react |
| กราฟ | Recharts |
| แอนิเมชัน | GSAP + คอมโพเนนต์ reveal ที่เขียนเอง |
| ธีม | next-themes + ค่าธีมใน `store_settings` |
| การทดสอบ | Vitest |
| ตัวจัดการแพ็กเกจ | npm (`package-lock.json`) |

## โครงสร้างโปรเจกต์

```
app/
  (home)/                   หน้าแรก (route group)
  about/ contact/           หน้าเนื้อหาทั่วไป
  products/                 รายการสินค้า + หน้ารายละเอียด [slug] พร้อมกรอง/เรียง/แบ่งหน้า
  rentals/                  รายการ, new/ ฟอร์มจอง, [id] รายละเอียด
  appointments/             รายการ, book/, [id] รายละเอียด
  notifications/            การแจ้งเตือนของผู้ใช้
  profile/                  โปรไฟล์ผู้ใช้
  admin/                    แดชบอร์ด + products, categories, appointments,
                            appointment-services, rentals, settings
  auth/                     login, sign-up, forgot/update password, callback, confirm, error
  setup-admin/              หน้าตั้งค่าแอดมินคนแรก
  api/setup-admin/          Endpoint POST สำหรับรับสิทธิ์แอดมินคนแรก
  api/store-asset/          Proxy ฝั่งเซิร์ฟเวอร์สำหรับ bucket store-assets
  actions/                  Server Actions (rentals.ts, appointments.ts)
  layout.tsx                root layout: ฟอนต์, ธีม, header/footer
components/
  ui/                       พื้นฐานจาก shadcn/ui
  header.tsx footer.tsx     โครงหลักของหน้า
  product-card.tsx          การ์ดสินค้า
  date-picker.tsx           ตัวเลือกวันที่ที่คำนึงถึงวันว่าง
  realtime-chat.tsx         คอมโพเนนต์แชตเรียลไทม์
hooks/                      React hooks
lib/
  supabase/                 client.ts, server.ts, admin.ts, proxy.ts, queries.ts
  db/                       ไคลเอนต์ Drizzle (index.ts) + schema (schema.ts)
  db.types.ts               อินเทอร์เฟซแถวข้อมูลที่เขียนมือ
  date-utils.ts             ตัวช่วยช่วงวันที่/การทับซ้อน/จำนวนวันเช่า
  __tests__/                เทสต์ Vitest ของ date-utils
supabase/migrations/        SQL schema, storage, security, cleanup
public/                     รูป gallery, review, services, wallpaper
drizzle.config.ts           คอนฟิก drizzle-kit
proxy.ts                    รีเฟรช session + ป้องกันเส้นทาง
vitest.config.mts           คอนฟิก Vitest
```

## การตั้งค่า Supabase

**SQL migrations ของ Supabase เป็นแหล่งความจริงของฐานข้อมูล** — RLS policies, triggers, ฟังก์ชัน `SECURITY DEFINER` และ exclusion constraints อยู่ใน `supabase/migrations/` ส่วน Drizzle สะท้อนสคีมาใน `lib/db/schema.ts` สำหรับ typed query และ introspection

| ไฟล์ | หน้าที่ |
| --- | --- |
| `00001_initial_schema.sql` | ตาราง, RLS policies, triggers, indexes, ฟังก์ชันตรวจวันว่าง |
| `00002_storage_bucket.sql` | Storage buckets และ policy การเข้าถึง |
| `00003_security_hardening.sql` | ตรึง `search_path`, กันการยกระดับสิทธิ์, constraint กันจองซ้ำ, ตั้ง payment slips เป็นส่วนตัว |
| `00004_drop_stock.sql` | ลบกลไกสต็อกที่ไม่ได้ใช้ |

ตาราง: `profiles`, `categories`, `products`, `product_images`, `store_settings`, `product_date_locks`, `rentals`, `appointment_services`, `appointments`, `notifications`

ฟังก์ชัน SQL ที่สำคัญ: `is_admin()`, `check_product_available(product, start, end, block_appointments)`, `get_product_unavailable_dates(product, start, end, block_appointments)`, `get_appointments_for_date(date)`, `get_rentals_for_calendar(product, start, end)`, `claim_first_admin()` รวมถึง trigger `guard_profile_role()` และ `guard_appointment_update()`

exclusion constraint สองตัว (ต้องมี `btree_gist`) กันการทับซ้อน: `rentals_no_overlap` สำหรับการเช่าสินค้าเดียวกันที่ยังไม่ถูกยกเลิก และ `appointments_no_overlap` สำหรับการนัดหมายบริการเดียวกันที่ยังไม่ถูกยกเลิก Storage buckets ได้แก่ `product-images`, `payment-slips` (ส่วนตัว), `store-assets` และ `category-images`

## เริ่มต้นใช้งาน

ความต้องการ: **Node.js >= 22.23.4** (`.nvmrc` ปักหมุด `24.22.1`), **npm** และโปรเจกต์ Supabase (หรือ Supabase CLI สำหรับรันในเครื่อง)

```bash
git clone https://github.com/NetParamad/project-wedding.git
cd project-wedding
npm install
cp .env.example .env.local   # fill in your Supabase values
```

ต้องรัน migration **ก่อนใช้งานแอป** เพราะการตรวจวันว่างพึ่งพาฟังก์ชัน `SECURITY DEFINER` รันผ่าน CLI:

```bash
supabase db push
```

หรือรันไฟล์ทั้งสี่ตามลำดับใน Supabase Dashboard → SQL Editor:

- `supabase/migrations/00001_initial_schema.sql`
- `supabase/migrations/00002_storage_bucket.sql`
- `supabase/migrations/00003_security_hardening.sql`
- `supabase/migrations/00004_drop_stock.sql`

จากนั้นเปิด dev server และสร้างแอดมินคนแรก:

```bash
npm run dev                  # http://localhost:3000
```

สมัครสมาชิก แล้วเข้า `/setup-admin` เพื่อรับสิทธิ์แอดมินให้บัญชีของคุณ

### คำสั่งสคริปต์

| คำสั่ง | คำอธิบาย |
| --- | --- |
| `npm run dev` | เปิด dev server ที่ `http://localhost:3000` |
| `npm run build` | บิลด์สำหรับโปรดักชัน |
| `npm run start` | เสิร์ฟบิลด์โปรดักชัน |
| `npm run lint` | รัน ESLint |
| `npm test` | รันยูนิตเทสต์ด้วย Vitest |
| `npm run db:pull` | ดึงโครงสร้างจากฐานข้อมูลจริงด้วย drizzle-kit |
| `npm run db:generate` | สร้าง Drizzle migrations |
| `npm run db:migrate` | รัน Drizzle migrations |
| `npm run db:push` | push Drizzle schema ขึ้นฐานข้อมูล |
| `npm run db:studio` | เปิด Drizzle Studio |

## ตัวแปรสภาพแวดล้อม

คัดลอก `.env.example` เป็น `.env.local` แล้วตั้งค่า (ระบุเฉพาะชื่อ — ห้ามคอมมิตค่าจริง):

| ตัวแปร | จำเป็น | หมายเหตุ |
| --- | --- | --- |
| `NEXT_PUBLIC_SUPABASE_URL` | ✅ | URL ของโปรเจกต์ Supabase |
| `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | ✅ | คีย์ publishable (หรือ anon แบบเดิม) ของ Supabase |
| `SUPABASE_SERVICE_ROLE_KEY` | ✅ สำหรับแอดมิน | ไคลเอนต์สิทธิ์พิเศษฝั่งเซิร์ฟเวอร์ใน `lib/supabase/admin.ts`; ห้ามเปิดเผยต่อเบราว์เซอร์ |
| `DATABASE_URL` | ✅ สำหรับ Drizzle | Connection string ของ Postgres สำหรับ `lib/db`; ใช้ transaction pooler (พอร์ต 6543) ตอน runtime |

โค้ดยังอ่าน `DIRECT_URL` เป็น fallback ของ drizzle-kit และ `VERCEL_URL` (Vercel ตั้งให้อัตโนมัติ) สำหรับ `metadataBase` โดยทั้งสองตัวไม่ได้อยู่ใน `.env.example` ส่วน `drizzle.config.ts` โหลด `.env.local` ผ่าน `process.loadEnvFile`

## วิธีใช้งาน

1. **เลือกชมสินค้า** — `/products` รองรับค้นหา กรองตามหมวดหมู่ เรียงลำดับ และแบ่งหน้า; `/products/[slug]` แสดงรายละเอียดและปฏิทินวันว่าง
2. **ตรวจวันว่าง** — ปฏิทินรวมการล็อค การเช่า และ (ตามเงื่อนไข) การนัดหมายเป็นวันที่ไม่ว่าง
3. **จองเช่า** — `/rentals/new` เลือกสินค้าและช่วงวันที่แบบนับรวม ตรวจวันว่าง และบันทึกรายละเอียดรับเองหรือจัดส่ง
4. **นัดหมาย** — `/appointments/book` เลือกบริการ วันที่ และช่วงเวลาว่าง 09:00–18:00 โดยเลือกผูกกับสินค้าได้
5. **ติดตามรายการ** — `/rentals`, `/appointments` และ `/notifications` แสดงข้อมูลของผู้ใช้ที่ล็อกอิน; `/profile` จัดการบัญชี
6. **จัดการร้าน** — `/admin` แสดงสถิติและกราฟ แถบเมนูจัดการสินค้า หมวดหมู่ บริการนัดหมาย การนัดหมาย การเช่า และตั้งค่า; ล็อควันได้จากหน้าจัดการสินค้า
7. **ตั้งค่าแอดมินคนแรก** — ผู้ใช้ที่ล็อกอินคนแรกรับสิทธิ์ได้ที่ `/setup-admin`

## หมายเหตุ

- **โมเดลวันว่าง** — สินค้าแต่ละชิ้นมีของจริงเพียง 1 ตัว `stock_qty` และ RPC ด้านสต็อกถูกลบใน migration `00004`; วันว่างอิงช่วงวันที่ล้วน
- **การตรวจสอบการเช่า** — UI เรียก `get_product_unavailable_dates` ฝั่งเซิร์ฟเวอร์ตรวจซ้ำด้วย `check_product_available` ตอนบันทึก และ exclusion constraints เป็นด่านสุดท้าย Rental ใช้ `block_appointments = true` ส่วน appointment ใช้ `false`
- **`/api/store-asset`** — เสิร์ฟรูปโลโก้และ QR พร้อมเพย์จาก bucket `store-assets` ผ่าน `/api/store-asset?path=<relative-path>` ตรวจ path ด้วยรูปแบบที่เข้มงวด และคืน cache แบบ immutable หนึ่งปี
- **`proxy.ts`** — รีเฟรช session ของ Supabase ทุกคำขอ และ redirect ผู้ใช้ที่ยังไม่ล็อกอินออกจาก `/appointments`, `/profile`, `/admin` และ `/notifications`
- **พอร์ตของ pooler** — ตั้ง `DATABASE_URL` เป็น transaction pooler (6543) สำหรับ runtime และใช้ session pooler หรือ direct URI (5432) สำหรับ DDL ของ drizzle-kit
- **การทดสอบ** — `lib/__tests__/date-utils.test.ts` ครอบคลุมการสร้างช่วงวันที่ การตรวจ overlap การนับจำนวนวันเช่าแบบนับรวม และการจัดรูปแบบวันที่
- **เวอร์ชัน Next.js** — `next` ปักหมุดเป็น `latest`; ก่อนพึ่งพา API ของเฟรมเวิร์กให้ตรวจ `node_modules/next/dist/docs/` (ดู [`AGENTS.md`](AGENTS.md))
- **การดีพลอย** — ยังไม่มีเอกสารระบุแพลตฟอร์มโฮสต์หรือ workflow ของ CI ใน repository
- **สัญญาอนุญาต** — ในโปรเจกต์ยังไม่มีไฟล์ `LICENSE`
