# Test Strategies — Per Task Type

## Test Categories

| Category | Tool | When to use |
|---|---|---|
| **DB Check** | `Bash` (psql/pg_isready) | DB connection, tables exist, migrations ran |
| **Data Verify** | `api-tester` (HTTP GET) | Seeded data exists via API |
| **API Test** | `api-tester` (HTTP *) | API endpoints return correct responses |
| **Build Check** | `Bash` (tsc, pnpm build) | TypeScript compiles, build passes |
| **Unit Test** | `Bash` (vitest/jest) | Function-level logic correctness |
| **E2E Test** | `claude-in-chrome` | UI renders, user flows work in browser |
| **Visual Check** | `claude-in-chrome` (screenshot) | Layout correct, responsive, no visual bugs |

---

## BE01 — Backend Setup

| Subtask | Test Strategy | How |
|---|---|---|
| BE01.1 DB Setup | DB Check | `pg_isready`, `psql -c "SELECT 1"` |
| BE01.2 Medusa Config | API Test | `GET http://localhost:9000/health` returns 200 |
| BE01.3 Seed Products | Data Verify | `GET /store/products` returns products |
| BE01.4 Region & Shipping | Data Verify | `GET /store/regions` returns VN region |
| BE01.5 Payment Provider | Data Verify | `GET /store/payment-providers` includes manual |
| BE01.6 Admin User | API Test | Admin login returns token |

### Unit Tests to Write
- None (infrastructure task)

### E2E Tests
- `GET /health` → 200
- Admin panel loads at `:9000/app`

---

## BE02 — Product Data

| Subtask | Test Strategy | How |
|---|---|---|
| BE02.1 Collections | Data Verify | `GET /store/collections` → 3 collections |
| BE02.2 Options | Data Verify | Products have color + size options |
| BE02.3 Products | Data Verify | `GET /store/products` → 4 products |
| BE02.4 Variants | Data Verify | Each product has color×size variants |
| BE02.5 Images | Data Verify | Products have thumbnail + images |
| BE02.6 Seed Script | Build Check | Script runs without errors |
| BE02.7 API Verify | API Test | Full API response validation |

### Unit Tests to Write
- Seed script idempotency test

### E2E Tests (API)
```
GET /store/products → 200, count >= 4
GET /store/products/{id} → 200, has variants
GET /store/products?collection_id[]=xxx → filtered results
```

---

## BE03 — Print Order Module

| Subtask | Test Strategy | How |
|---|---|---|
| BE03.1 Data Model | DB Check | Table `print_job` exists with correct columns |
| BE03.2 Service | Unit Test | CRUD operations, sendToPrintShop mock |
| BE03.3 Module Def | Build Check | Module resolves in container |
| BE03.4 Register | Build Check | medusa-config includes module |
| BE03.5 Migration | DB Check | Migration applied, table exists |
| BE03.6 Retry Logic | Unit Test | Retry behavior with mock failures |
| BE03.7 Tests | Unit Test | All unit tests pass |

### Unit Tests to Write
```
print-order.service.test.ts:
  - createPrintJob: creates record with correct fields
  - updatePrintJobStatus: updates status correctly
  - getByOrderId: returns jobs for order
  - callWithRetry: succeeds on first try
  - callWithRetry: retries then succeeds
  - callWithRetry: fails after max retries
  - sendToPrintShop: sends correct payload format
  - sendToPrintShop: creates PrintJob after send
```

---

## BE04 — Workflow & Subscriber

| Subtask | Test Strategy | How |
|---|---|---|
| BE04.1-4 Workflow | Unit Test | Each step, full workflow with mocks |
| BE04.5 Subscriber | Unit Test | Event triggers workflow |
| BE04.6 Integration | API Test | Create order → verify PrintJob created |

### Unit Tests to Write
```
fetch-order.step.test.ts:
  - fetches order with line items
  - throws on missing order

build-payload.step.test.ts:
  - transforms order to PrintShopPayload
  - validates design URLs in metadata
  - throws on missing design metadata

send-to-print-shop.step.test.ts:
  - sends payload, returns job id
  - compensation cancels job

send-to-print-shop.workflow.test.ts:
  - full workflow success path
  - workflow rolls back on step failure

order-placed.subscriber.test.ts:
  - triggers workflow on order.placed event
  - logs error without re-throwing on failure
```

---

## BE05 — Webhooks & API Routes

| Subtask | Test Strategy | How |
|---|---|---|
| BE05.1 Webhook Route | API Test | POST with valid/invalid API key |
| BE05.2 Middleware | API Test | Auth skipped for /webhooks/* |
| BE05.3-5 Admin Routes | API Test | CRUD with admin auth |
| BE05.6 Schemas | Unit Test | Zod validation pass/fail |
| BE05.7 Links | Build Check | Link resolves correctly |

### Unit Tests to Write
```
webhook-payload.schema.test.ts:
  - valid payload passes
  - missing order_id fails
  - invalid status fails

admin-routes.test.ts:
  - GET /admin/print-orders returns list
  - GET /admin/print-orders/:id returns detail
  - POST /admin/print-orders/:id/retry retries failed job
  - POST /admin/print-orders/:id/retry rejects non-failed job
```

### E2E Tests (API)
```
POST /webhooks/print-shop (no API key) → 401
POST /webhooks/print-shop (valid key, valid body) → 200
POST /webhooks/print-shop (valid key, invalid body) → 400
GET /admin/print-orders (no auth) → 401
GET /admin/print-orders (admin auth) → 200
```

---

## FE01 — Frontend Setup

| Subtask | Test Strategy | How |
|---|---|---|
| FE01.1 Folders | Build Check | Directories exist |
| FE01.2 env.ts | Unit Test | Throws on missing vars, returns typed |
| FE01.3 medusa.ts | Build Check | Exports medusa instance |
| FE01.4 s3.ts | Unit Test | Generates correct presigned URL |
| FE01.5 Utils | Build Check | cn() works, fonts load |
| FE01.6 Layout | E2E (Browser) | Page renders at localhost:3000 |
| FE01.7 Types | Build Check | @tshirt/shared imports work |

### Unit Tests to Write
```
env.test.ts:
  - returns typed env when all vars present
  - throws on missing required var

s3.test.ts:
  - generatePresignedUrl returns url + fileUrl
  - getDesignKey formats correctly
```

### E2E Tests (Browser)
- Open `http://localhost:3000` → page loads without error
- Check console for no critical errors

---

## FE02 — Landing Page

| Subtask | Test Strategy | How |
|---|---|---|
| FE02.1-9 Sections | E2E (Browser) | All sections visible, CTAs work |

### Unit Tests to Write
```
landing/HeroSection.test.tsx:
  - renders headline and CTA
  - CTA links to /products

landing/HowItWorks.test.tsx:
  - renders 3 steps

landing/PricingSection.test.tsx:
  - renders 3 pricing cards
  - highlights popular card
```

### E2E Tests (Browser)
- Navigate to `/` → Hero section visible
- Screenshot: full page layout check
- Click "Thiết kế ngay" → navigates to `/products`
- Scroll to FAQ → accordion opens/closes
- Mobile viewport (375px): layout stacks correctly

---

## FE03 — Product Pages

### Unit Tests to Write
```
product/ProductCard.test.tsx:
  - renders product name, price, image
  - shows color swatches
  - links to product detail

product/ProductFilter.test.tsx:
  - renders filter chips
  - calls onFilter when clicked
  - highlights active filter

product/SizeChart.test.tsx:
  - renders size table with correct data

product/VariantSelector.test.tsx:
  - selects color updates state
  - selects size updates state
  - maps color+size to variant_id
  - shows error when incomplete selection
```

### E2E Tests (Browser)
- `/products` → product grid renders with cards
- Click filter "Polo" → only polo products shown
- `/products/:id` → product detail with image, options
- Select color + size → "Bắt đầu thiết kế" enabled
- Click CTA → navigates to `/design/:id` with correct params

---

## FE04 — Design Editor

### Unit Tests to Write
```
store/design.store.test.ts:
  - initial state correct
  - setActiveTool updates tool
  - setSide switches front/back
  - saveSnapshot adds to history
  - undo moves historyIndex back
  - redo moves historyIndex forward
  - history max 30 items

canvas/constraints.test.ts:
  - isWithinPrintArea: inside → true
  - isWithinPrintArea: outside → false
  - isWithinPrintArea: partially outside → false
  - validateAllObjects: all inside → valid
  - validateAllObjects: some outside → returns list

canvas/export.test.ts:
  - exportToJson: includes custom properties
  - loadFromJson: round-trip preserves objects
```

### E2E Tests (Browser)
- `/design/:id` → editor loads, canvas visible
- Add text → text appears on canvas
- Add image → image appears in print area
- Undo → element removed
- Redo → element restored
- Layer panel shows elements
- Side toggle: front ↔ back preserves state
- Element outside print area → warning shown

---

## FE05 — Preview & Export

### Unit Tests to Write
```
canvas/export.test.ts:
  - exportToPng: returns Blob with image/png type
  - exportToPng: correct dimensions (multiplier)

api/upload-design.test.ts:
  - valid request → returns presignedUrl + fileUrl
  - invalid contentType → 400
  - missing fileName → 400
```

### E2E Tests (Browser)
- Click "Xem trước" → preview modal opens
- Mockup shows design overlaid on T-shirt
- Click "Thêm vào giỏ hàng" → loading states shown → redirect to /cart

---

## FE06 — Cart

### E2E Tests (Browser)
- `/cart` with items → items list renders
- Change quantity → total updates
- Delete item → item removed
- Click "Chỉnh sửa" → navigates to editor with saved design
- Empty cart → empty state shown
- Click "Tiến hành thanh toán" → navigates to `/checkout`

---

## FE07 — Checkout

### Unit Tests to Write
```
checkout/schemas.test.ts:
  - valid form data passes
  - invalid phone fails
  - missing required fields fail
  - cascading address validation
```

### E2E Tests (Browser)
- `/checkout` → form renders
- Fill valid data → submit → order success
- Invalid phone → inline error shown
- Province → District → Ward cascading works
- COD selected → no redirect
- Success page → order ID shown, links work

---

## FE08 — Order Tracking

### E2E Tests (Browser)
- `/orders/:id` → order details render
- Progress tracker shows correct status
- Design thumbnails clickable
- Invalid order ID → error state
