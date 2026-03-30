# BE02 — Product Data & API

## Scope
Setup products, variants, images, collections trong Medusa.

## Tasks

### BE02.1 — Product Collections
- [ ] Create collections via admin hoặc seed:
  - "T-shirt" (handle: `tshirt`)
  - "Polo" (handle: `polo`)
  - "Hoodie" (handle: `hoodie`)

### BE02.2 — Product Options
- [ ] Define options per product:
  - Color: Trắng, Đen, Xám nhạt, Xám đậm, Đỏ, Xanh dương, Xanh lá, Vàng, Navy, Kem
  - Size: S, M, L, XL, XXL

### BE02.3 — Products
- [ ] T-shirt Basic: 199.000₫, Cotton 100% 180gsm
- [ ] T-shirt Premium: 249.000₫, Cotton Compact 220gsm
- [ ] Polo: 299.000₫, Cotton Pique 200gsm
- [ ] Hoodie: 399.000₫, Cotton Fleece 320gsm
- [ ] Each product: description, thumbnail, images (front/back per color)

### BE02.4 — Variants Generation
- [ ] Per product: Color × Size variants
- [ ] Pricing per variant (same base price)
- [ ] Inventory: manage stock hoặc unlimited (initial)
- [ ] SKU format: `[TYPE]-[COLOR]-[SIZE]` (e.g., `TSB-WHT-L`)

### BE02.5 — Product Images
- [ ] Mockup images per product per color (front + back)
- [ ] Upload to Medusa file service hoặc S3
- [ ] Associate images with products/variants

### BE02.6 — Seed Script
- [ ] Create `data/seed-products.ts` hoặc JSON
- [ ] Automate product + variant + image creation
- [ ] Idempotent: can re-run without duplicates
- [ ] Run: `pnpm medusa exec ./data/seed-products.ts`

### BE02.7 — Verify Store API
- [ ] GET /store/products — returns all products
- [ ] GET /store/products/{id} — returns product with variants
- [ ] GET /store/products?collection_id[]=xxx — filter works
- [ ] Verify publishable key required

## Acceptance Criteria
- 4 products with full variants in database
- Store API returns correct data
- Images accessible
- Filter by collection works

## Dependencies
- BE01 (database, Medusa running)

## Estimated Subtasks: 7
