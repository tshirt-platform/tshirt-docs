# BE01 — Backend Project Setup

## Scope
Cấu hình Medusa backend, database, cơ bản.

## Tasks

### BE01.1 — Database Setup
- [ ] Tạo PostgreSQL database `tshirt_db`
- [ ] Verify Redis running (localhost:6379)
- [ ] Update DATABASE_URL trong `.env`
- [ ] Run `pnpm medusa db:migrate`

### BE01.2 — Medusa Config (`medusa-config.ts`)
- [ ] Verify projectConfig: database, redis, http
- [ ] Setup CORS: STORE_CORS, ADMIN_CORS, AUTH_CORS
- [ ] Register custom modules (placeholder)
- [ ] Verify admin panel accessible at :9000/app

### BE01.3 — Seed Product Data
- [ ] Tạo seed script hoặc manual via admin
- [ ] Products: T-shirt Basic, T-shirt Premium, Polo, Hoodie
- [ ] Variants per product: 10 colors × 5 sizes = 50 variants
- [ ] Product images (mockup placeholders)
- [ ] Pricing: theo `docs/business.md`
- [ ] Collections/Tags cho filter (tshirt, polo, hoodie)

### BE01.4 — Region & Shipping Setup
- [ ] Create region: Vietnam (VND currency)
- [ ] Shipping options:
  - Nội thành: 25.000₫
  - Ngoại thành: 30.000₫
  - Liên tỉnh: 35.000₫
- [ ] Free shipping thresholds

### BE01.5 — Payment Provider Setup
- [ ] Manual payment provider (COD) — built-in
- [ ] VNPay provider (future — placeholder)

### BE01.6 — Admin User
- [ ] Create admin user: `pnpm medusa user -e admin@tshirt.vn -p [password]`
- [ ] Verify login at admin panel

## Acceptance Criteria
- `pnpm dev` starts without errors
- Admin panel accessible and functional
- Products visible via Store API: GET /store/products
- Regions and shipping options configured
- Database migrations applied

## Dependencies
- None (first BE task)

## Estimated Subtasks: 6
