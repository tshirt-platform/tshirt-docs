# FE05 — Preview & Design Export

## Scope
Preview modal, export PNG/JSON, upload to S3, add design to cart.

## Reference
- `docs/user-flow.md` → Step 4
- `docs/system-flow.md` → Flow 3

## Tasks

### FE05.1 — Export Utilities (`lib/canvas/export.ts`)
- [ ] `exportToPng(canvas)` → Blob (3000×3000, 300 DPI)
  - Scale multiplier = 3000 / canvas.getWidth()
  - Filter: exclude non-printable objects
  - Convert dataURL → Blob
- [ ] `exportToJson(canvas)` → string
  - Include custom properties: excludeFromExport, name, layerName
- [ ] `loadFromJson(canvas, json)` → Promise (for re-editing)

### FE05.2 — Upload Design API Route (`app/api/upload-design/route.ts`)
- [ ] POST handler
- [ ] Zod validation: { fileName, contentType, side }
- [ ] Generate unique S3 key: `designs/{uuid}/{side}.{ext}`
- [ ] Create PutObjectCommand
- [ ] Generate presigned URL (expires 15min)
- [ ] Return: { presignedUrl, fileUrl }
- [ ] Error handling: 400 (validation), 500 (S3 error)

### FE05.3 — Upload Flow (Client)
- [ ] `uploadDesign(blob, contentType, side)` helper function
- [ ] Step 1: POST /api/upload-design → get presigned URL
- [ ] Step 2: PUT blob to presigned URL
- [ ] Step 3: Return permanent fileUrl
- [ ] Retry logic: 3 attempts for S3 upload
- [ ] Progress tracking (optional: XHR for upload progress)

### FE05.4 — Preview Modal (`PreviewModal.tsx`)
- [ ] Full-screen Dialog (shadcn Dialog)
- [ ] Mockup: T-shirt image với design overlaid
- [ ] Show front AND back nếu cả 2 đã design
- [ ] Product info: name, color, size, quantity, price
- [ ] Actions: "Quay lại chỉnh sửa", "Thêm vào giỏ hàng"

### FE05.5 — Add to Cart Flow
- [ ] On "Thêm vào giỏ hàng":
  1. Validate print area (all elements within bounds)
  2. Show loading: "Đang xuất thiết kế..."
  3. Export PNG + JSON
  4. Show loading: "Đang tải lên..."
  5. Upload both to S3
  6. Show loading: "Đang thêm vào giỏ..."
  7. POST /store/carts/{id}/line-items with metadata
  8. Success → navigate to `/cart`
  9. Error → toast + keep modal open
- [ ] Loading states trong CTA button
- [ ] Error recovery: retry button

### FE05.6 — Validation Before Export
- [ ] Check all elements within print area
- [ ] Show warning modal nếu có elements ngoài:
  - List out-of-bounds elements
  - "Tự động điều chỉnh" button (move elements inside)
  - "Xuất anyway" button (proceed)
  - "Quay lại chỉnh sửa" button

## Acceptance Criteria
- Export PNG đúng 3000×3000
- Export JSON loadable (round-trip: export → load → identical canvas)
- S3 upload succeeds
- Cart line item has correct metadata
- Loading states hiển thị đúng
- Validation catches out-of-bounds elements

## Dependencies
- FE01 (s3.ts, medusa.ts)
- FE04 (design editor, canvas)

## Estimated Subtasks: 6
