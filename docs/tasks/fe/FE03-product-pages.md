# FE03 — Product Listing & Detail Pages

## Scope
Trang danh sách sản phẩm và trang chi tiết sản phẩm.

## Reference
- `docs/user-flow.md` → Step 2
- `docs/system-flow.md` → Flow 1

## Tasks

### FE03.1 — Product Listing Page (`/products`)
- [ ] Tạo `app/(store)/products/page.tsx`
- [ ] SSR: Fetch products từ Medusa API
- [ ] Product grid: 3 cột desktop, 2 tablet, 1 mobile
- [ ] Loading skeleton khi fetch

### FE03.2 — Product Card Component
- [ ] Ảnh mockup (hover → đổi sang back image)
- [ ] Product name + giá (format VND)
- [ ] Color swatches (hiển thị 3-5 màu đầu tiên)
- [ ] Badge: "Mới", "Bán chạy" từ product tags
- [ ] Click → navigate to `/products/[id]`

### FE03.3 — Product Filter
- [ ] Filter bar: chip buttons (Tất cả / T-shirt / Polo / Hoodie)
- [ ] Filter by collection hoặc product type
- [ ] URL sync: `/products?type=tshirt`
- [ ] Re-fetch products khi filter change

### FE03.4 — Product Detail Page (`/products/[id]`)
- [ ] Tạo `app/(store)/products/[id]/page.tsx`
- [ ] SSR: Fetch product by ID từ Medusa API
- [ ] Layout 2 cột: image gallery (left) + info panel (right)

### FE03.5 — Image Gallery
- [ ] Main image (large)
- [ ] Thumbnail row (click để đổi main image)
- [ ] Front / Back views
- [ ] Next.js Image component với optimization
- [ ] Lightbox on click (optional)

### FE03.6 — Product Info Panel
- [ ] Tên sản phẩm + giá
- [ ] Color picker: grid buttons, highlight selected
- [ ] Size selector: button group (S/M/L/XL/XXL)
- [ ] Quantity input: number input, min 1, max 100
- [ ] "Bảng size" link → open Size Chart Sheet
- [ ] "Bắt đầu thiết kế" CTA button

### FE03.7 — Size Chart
- [ ] shadcn Sheet (drawer from right)
- [ ] Table: Size × Measurements (Vai, Ngực, Dài, Tay)
- [ ] Data from `docs/business.md`

### FE03.8 — Variant Selection Logic
- [ ] Map color + size → variant_id từ product.variants
- [ ] Update mockup image khi đổi color
- [ ] Sync selections to URL params
- [ ] Validate before navigate to editor
- [ ] Error state: "Vui lòng chọn màu và size"

### FE03.9 — Navigate to Editor
- [ ] On click "Bắt đầu thiết kế":
  1. Validate color + size + quantity selected
  2. Navigate to `/design/[productId]?variantId=[id]&color=[color]&size=[size]&qty=[qty]`

## Acceptance Criteria
- Products fetch từ Medusa API và render đúng
- Filter hoạt động, URL sync
- Color/size selection map đúng variant
- Navigate to editor với đúng params
- Responsive trên tất cả breakpoints

## Dependencies
- FE01 (project setup, medusa.ts)
- BE02 (products seeded in Medusa)

## Estimated Subtasks: 9
