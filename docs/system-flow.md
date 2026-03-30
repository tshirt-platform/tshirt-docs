# System Flow — Chi tiết kiến trúc & luồng dữ liệu

## Architecture Overview

```
                    ┌──────────────────────────────────────────────┐
                    │              Client (Browser)                 │
                    │         Next.js 16 — tshirt-store            │
                    │            localhost:3000                     │
                    └──────────┬──────────┬──────────┬─────────────┘
                               │          │          │
                    ┌──────────▼──┐  ┌────▼────┐  ┌─▼──────────────┐
                    │ Medusa API  │  │ Next.js  │  │   AWS S3       │
                    │ :9000       │  │ API Route│  │ (Design files) │
                    │ tshirt-     │  │ /api/*   │  │                │
                    │ backend     │  └────┬─────┘  └────────────────┘
                    └──────┬──────┘       │
                           │              │ Presigned URL generation
                    ┌──────▼──────┐       │
                    │ PostgreSQL  │  ┌────▼─────────────┐
                    │ + Redis     │  │ @aws-sdk/client-s3│
                    └─────────────┘  └──────────────────┘
                           │
                    ┌──────▼──────────────┐
                    │ Print Shop API       │
                    │ (External service)   │
                    └──────────────────────┘
```

---

## Flow 1: Product Browsing

### Sequence: Landing → Product List → Product Detail

```
Browser                          Next.js Server              Medusa API
  │                                   │                          │
  │  GET /                            │                          │
  │──────────────────────────────────▶│                          │
  │  SSR: render landing page         │                          │
  │  (static sections + dynamic       │                          │
  │   template gallery)               │                          │
  │◀──────────────────────────────────│                          │
  │                                   │                          │
  │  GET /products                    │                          │
  │──────────────────────────────────▶│                          │
  │                                   │  GET /store/products     │
  │                                   │  ?fields=+variants       │
  │                                   │  &limit=20               │
  │                                   │──────��──────────────────▶│
  │                                   │◀─────────────────────────│
  │  SSR: render product grid         │  { products, count }     │
  │◀──────────────────────────────────│                          │
  │                                   │                          │
  │  GET /products/[id]               │                          │
  │──────────────────────────────────▶│                          │
  │                                   │  GET /store/products/[id]│
  │                                   │  ?fields=+variants       │
  │                                   │  +images,+options        │
  │                                   │─────────────────────────▶│
  │                                   │◀─────────────────────────│
  │  SSR: render product detail       │                          │
  │◀──────────────────────────────────│                          │
```

### API Calls

| Step | Method | Endpoint | Purpose |
|---|---|---|---|
| Product List | GET | `/store/products?limit=20&offset=0` | Fetch products with pagination |
| Product List (filter) | GET | `/store/products?collection_id[]=xxx` | Filter by shirt type |
| Product Detail | GET | `/store/products/{id}?fields=+variants,+images,+options` | Full product with variants |

### Medusa SDK Usage

```ts
// Product listing
const { products, count } = await medusa.store.product.list({
  limit: 20,
  offset: 0,
  fields: "+variants,+images",
})

// Product detail
const { product } = await medusa.store.product.retrieve(productId, {
  fields: "+variants,+images,+options",
})

// Variant selection (color + size → variant)
const variant = product.variants.find(
  (v) => v.options.color === selectedColor && v.options.size === selectedSize
)
```

---

## Flow 2: Cart Management

### Sequence: Create Cart → Add Item → Update → Get Cart

```
Browser                          Medusa API
  │                                   │
  │  POST /store/carts                │
  │  (first visit, no cart exists)    │
  │──────────────────────────────────▶│
  │◀──────────────────────────────────│
  │  { cart: { id: "cart_xxx" } }     │
  │  → Store cart_id in cookie/       │
  │    localStorage                   │
  │                                   │
  │  POST /store/carts/{id}/          │
  │       line-items                  │
  │  {                                │
  │    variant_id: "variant_xxx",     │
  │    quantity: 2,                   │
  │    metadata: {                    │
  │      design_png_url: "...",       │
  │      design_json_url: "...",      │
  │      design_side: "front"         │
  │    }                              │
  │  }                                │
  │──────────────────────────────────▶│
  │◀──────────────────────────────────│
  │  { cart: { ... updated } }        │
  │                                   │
  │  GET /store/carts/{id}            │
  │──────────────────────────────────▶│
  │◀──────────────────────────────────│
  │  { cart: { items, totals } }      │
```

### Cart Storage Strategy
```
1. Check localStorage for cart_id
2. If exists → GET /store/carts/{id} to validate
3. If not exists or invalid → POST /store/carts to create new
4. Store cart_id in localStorage
5. On each page load, hydrate cart state from API
```

### API Calls

| Step | Method | Endpoint | Body |
|---|---|---|---|
| Create cart | POST | `/store/carts` | `{ region_id }` |
| Get cart | GET | `/store/carts/{id}` | — |
| Add item | POST | `/store/carts/{id}/line-items` | `{ variant_id, quantity, metadata }` |
| Update item | POST | `/store/carts/{id}/line-items/{itemId}` | `{ quantity }` |
| Remove item | DELETE | `/store/carts/{id}/line-items/{itemId}` | — |

---

## Flow 3: Design Upload (Core)

### Sequence: Export → Presign → Upload → Attach to Cart

```
Browser (Canvas)          Next.js API Route         AWS S3
  │                            │                       │
  │ 1. Export PNG from canvas  │                       │
  │    (3000x3000, 300 DPI)   │                       │
  │ 2. Export JSON state       │                       │
  │                            │                       │
  │ POST /api/upload-design    │                       │
  │ { fileName, contentType,   │                       │
  │   side: "front" }          │                       │
  │───────────────────────────▶│                       │
  │                            │                       │
  │                            │ PutObjectCommand      │
  │                            │ getSignedUrl()        │
  │                            │──────────────────────▶│
  │                            │◀──────────────────────│
  │                            │ presignedUrl           │
  │                            │                       │
  │◀───────────────────────────│                       │
  │ { presignedUrl, fileUrl }  │                       │
  │                            │                       │
  │ PUT presignedUrl           │                       │
  │ Body: PNG blob             │                       │
  │────���───────────────────────────────────────────────▶│
  │◀────────────────────────────────────────────────────│
  │ 200 OK                     │                       │
  │                            │                       │
  │ (Repeat for JSON file)     │                       │
  │                            │                       │
  │ POST /store/carts/{id}/line-items                  │
  │ { metadata: { design_png_url, design_json_url } }  │
  │───────────────────────────▶│                       │
  │                   (proxy to Medusa)                │
```

### Next.js API Route: `/api/upload-design`

```ts
// Input
{
  fileName: string        // e.g. "front.png"
  contentType: string     // "image/png" or "application/json"
  side: "front" | "back"
  orderId?: string        // for organizing S3 keys
}

// Processing
1. Validate input with Zod
2. Generate unique key: `designs/{uuid}/{side}.{ext}`
3. Create PutObjectCommand
4. Generate presigned URL (expires: 15 minutes)
5. Return { presignedUrl, fileUrl }

// Output
{
  presignedUrl: string    // Temporary upload URL
  fileUrl: string         // Permanent public URL
}
```

### S3 Key Structure
```
{S3_DESIGNS_PREFIX}{unique-id}/{side}.png
{S3_DESIGNS_PREFIX}{unique-id}/{side}.json

Example:
designs/a1b2c3d4-e5f6/front.png
designs/a1b2c3d4-e5f6/front.json
designs/a1b2c3d4-e5f6/back.png
designs/a1b2c3d4-e5f6/back.json
```

### Error Handling

| Error | Client Action |
|---|---|
| Presigned URL generation fails | Toast error, allow retry |
| S3 upload fails (network) | Retry up to 3 times, then toast error |
| S3 upload fails (403/expired) | Re-request presigned URL, retry upload |
| Add to cart fails | Toast error, keep design in memory for retry |

---

## Flow 4: Checkout & Payment

### Sequence: Shipping → Payment → Complete

```
Browser                          Medusa API              VNPay Gateway
  │                                   │                       │
  │ POST /store/carts/{id}            │                       │
  │ { shipping_address: {...},        │                       │
  │   email: "..." }                  │                       │
  │──────────────────────────────────▶│                       │
  │◀──────────────────────────────────│                       │
  │                                   │                       │
  │ POST /store/carts/{id}/           │                       │
  │      shipping-methods             │                       │
  │ { option_id: "so_xxx" }           │                       │
  │──────────────────────────────────▶│                       │
  │◀──────────────────────────────────│                       │
  │                                   │                       │
  │ ┌─── IF COD ──────────────────────────────────────────┐   │
  │ │                                 │                    │   │
  │ │ POST /store/carts/{id}/         │                    │   │
  │ │      payment-sessions           │                    │   │
  │ │ { provider_id: "manual" }       │                    │   │
  │ │────────────────────────────────▶│                    │   │
  │ │◀───────────────────────────────│                    │   │
  │ │                                 │                    │   │
  │ │ POST /store/carts/{id}/complete │                    │   │
  │ │────────────────────────────────▶│                    │   │
  │ │◀───────────────────────────────│                    │   │
  │ │ { type: "order", order }        │                    │   │
  │ │ → Navigate to /checkout/success │                    │   │
  │ └─────────────────────────────────────────────────────┘   │
  │                                   │                       │
  │ ┌─── IF VNPay ────────────────────────────────────────┐   │
  │ │                                 │                    │   │
  │ │ POST /store/carts/{id}/         │                    │   │
  │ │      payment-sessions           │                    │   │
  │ │ { provider_id: "vnpay" }        │                    │   │
  │ │────────────────────────────────▶│                    │   │
  │ │◀───────────────────────────────│                    │   │
  │ │ { payment_url: "https://..." }  │                    │   │
  │ │                                 │                    │   │
  │ │ → Redirect to VNPay            │                    │   │
  │ │─────────────────────────────────────────────────────▶│  │
  │ │                                 │                    │   │
  │ │ ◀── User completes payment ─────────────────────────│   │
  │ │ Callback: /checkout/success?    │                    │   │
  │ │          vnp_ResponseCode=00    │                    │   │
  │ │                                 │                    │   │
  │ │ POST /store/carts/{id}/complete │                    │   │
  │ │────────────────────────────────▶│                    │   │
  │ │◀───────────────────────────────│                    │   │
  │ └─────────────────────────────────────────────────────┘   │
```

### Checkout Steps (API sequence)

| # | Method | Endpoint | Purpose |
|---|---|---|---|
| 1 | POST | `/store/carts/{id}` | Update shipping address + email |
| 2 | GET | `/store/shipping-options?cart_id={id}` | Get available shipping methods |
| 3 | POST | `/store/carts/{id}/shipping-methods` | Set shipping method |
| 4 | POST | `/store/carts/{id}/payment-sessions` | Initialize payment (COD or VNPay) |
| 5 | POST | `/store/carts/{id}/complete` | Finalize order |

### Cart Complete Response
```ts
// Success
{
  type: "order",
  order: {
    id: "order_xxx",
    display_id: 1001,
    items: [...],
    shipping_address: {...},
    total: 350000,
    // ...
  }
}

// Requires action (e.g., VNPay redirect)
{
  type: "cart",
  cart: {
    payment_session: {
      data: { payment_url: "https://vnpay.vn/..." }
    }
  }
}
```

---

## Flow 5: Order Processing (Backend)

### Sequence: Order Placed → Print Shop → Status Updates

```
Medusa API                    Subscriber                Workflow                    Print Shop API
  │                              │                          │                            │
  │ order.placed event           │                          │                            │
  │─────────────────────────────▶│                          │                            │
  │                              │                          │                            │
  │                              │ Invoke workflow          │                            │
  │                              │─���───────────────────────▶│                            │
  │                              │                          │                            │
  │                              │              Step 1: fetchOrderStep                   │
  │                              │              GET order + line items                    │
  │                              │              + metadata (design URLs)                  │
  │                              │                          │                            │
  │                              │              Step 2: buildPayloadStep                 │
  │                              │              Transform order → PrintShopPayload       │
  │                              │                          │                            │
  │                              │              Step 3: sendToPrintShopStep              │
  │                              │                          │ POST /api/orders           │
  │                              │                          │ { order_id, customer,      │
  │                              │                          │   items: [...] }           │
  │                              │                          │───────────────────────────▶│
  │                              │                          │◀───────────────────────────│
  │                              │                          │ { job_id: "pj_xxx" }       │
  │                              │                          │                            │
  │                              │              Step 4: createPrintJobStep               │
  │                              │              Create PrintJob record                   │
  │                              │              status: "pending"                        │
  │                              │                          │                            │
  │                              │◀─────────────────────────│                            │
  │                              │ Workflow complete         │                            │
```

### Workflow Steps Detail

```ts
// Step 1: fetchOrderStep
Input:  { order_id: string }
Output: { order: Order & { items: LineItem[] } }
Action: Resolve "order" service → retrieveOrder with relations

// Step 2: buildPayloadStep
Input:  { order: Order }
Output: { payload: PrintShopWebhookPayload }
Action: Map order data to print shop format:
  - order.shipping_address → customer
  - order.items → items (extract design URLs from metadata)
  - order.created_at → created_at

// Step 3: sendToPrintShopStep
Input:  { payload: PrintShopWebhookPayload }
Output: { external_job_id: string }
Action: POST to PRINT_SHOP_API_URL with retry (1s, 4s, 16s)
Compensation: Cancel print job if later step fails

// Step 4: createPrintJobStep
Input:  { order_id, external_job_id, design_urls }
Output: { print_job: PrintJob }
Action: Create PrintJob record in database
```

### Print Shop Status Webhook (Incoming)

```
Print Shop API                Medusa API (/webhooks/print-shop)        Database
  │                                   │                                    │
  │ POST /webhooks/print-shop         │                                    │
  │ Headers: X-API-Key: xxx           │                                    │
  │ {                                 │                                    │
  │   order_id: "order_xxx",          │                                    │
  │   status: "printing",             │                                    │
  │   tracking_number: null           │                                    │
  │ }                                 │                                    │
  │──────────────────────────────────▶│                                    │
  │                                   │ 1. Validate X-API-Key              │
  │                                   │ 2. Parse body                      │
  │                                   │ 3. Update PrintJob                 │
  │                                   │────────────────────────────────────▶│
  │                                   │◀────────────────────────────────────│
  │                                   │                                    │
  │                                   │ (if status === "shipped")          │
  │                                   │ 4. Create Fulfillment              │
  │                                   │    with tracking_number            │
  │                                   │────────────────────────────────────▶│
  │                                   │◀────────────────────────────────────│
  │◀──────────────────────────────────│                                    │
  │ 200 OK                            │                                    │
```

### Status Transitions

```
                    ┌──────────┐
                    │ pending  │ (Order placed, sent to print shop)
                    └────┬─────┘
                         │
                    ┌────▼─────┐
               ┌───│ printing  │ (Print shop started production)
               │   └────┬─────┘
               │        │
               │   ┌────▼─────┐
               │   │ shipped   │ (Print shop shipped, tracking available)
               │   └────┬─────┘
               │        │
               │   ┌────▼─────┐
               │   │ delivered │ (Customer received)
               │   └──────────┘
               │
               │   ┌──────────┐
               └──▶│  failed  │ (Error at any step)
                   └──────────┘
```

---

## Flow 6: Design Re-edit

### Sequence: Cart → Load saved design → Edit → Re-upload

```
Browser                     S3                    Next.js API          Medusa API
  │                          │                        │                    │
  │  User clicks "Chỉnh sửa" on cart item             │                    │
  │                          │                        │                    │
  │  GET design_json_url     │                        │                    │
  │──���──────────────────────▶│                        │                    │
  │◀─────────────────────────│                        │                    │
  │  Fabric.js JSON          │                        │                    │
  │                          │                        │                    │
  │  Navigate to /design/[productId]?edit=true         │                    │
  │  canvas.loadFromJSON(json)                         │                    │
  │                          │                        │                    │
  │  ... user edits design ...                         │                    │
  │                          │                        │                    │
  │  Export new PNG + JSON    │                        │                    │
  │                          │                        │                    │
  │  POST /api/upload-design  │                        │                    │
  │──────────────────────────────────────────────────▶│                    │
  │◀──────────────────────────────────────────────────│                    │
  │  { presignedUrl, fileUrl }                         │                    │
  │                          │                        │                    │
  │  PUT new files to S3     │                        │                    │
  │─��───────────────────────▶│                        │                    │
  │◀─────────────────────────│                        │                    │
  │                          │                        │                    │
  │  POST /store/carts/{id}/line-items/{itemId}        │                    │
  │  { metadata: { new design URLs } }                 │                    │
  │────────────────────────────────────────────────────────────────────────▶│
  │◀───────────────────────────────────────────────────────────────────────│
```

---

## Data Flow Summary

### Client State (Zustand)

```
┌─────────────────────────────────────────────────────┐
│                  Zustand Stores                      │
├─────────────────────────────────────────────────────┤
│                                                      │
│  designStore                                         │
│  ├── canvas: fabric.Canvas | null                    │
│  ├── productId: string                               │
│  ├── variantId: string                               │
│  ├── side: "front" | "back"                          │
│  ├── history: string[] (JSON snapshots)              │
│  ├── historyIndex: number                            │
│  ├── activeTool: ActiveTool                          │
│  ├── frontCanvasJson: string | null                  │
│  ├── backCanvasJson: string | null                   │
│  ├── pngUrl: string | null (after S3 upload)         │
│  └── jsonUrl: string | null (after S3 upload)        │
│                                                      │
│  cartStore                                           │
│  ├── cartId: string | null                           │
│  ├── items: CartItem[]                               │
│  ├── total: number                                   │
│  └── itemCount: number                               │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Server State (Medusa + PostgreSQL)

```
┌─────────────────────────────────────────────────────┐
│              Medusa Core Tables                      │
├─────────────────────────────────────────────────────┤
│  Product → ProductVariant (color/size combos)        │
│  Cart → LineItem (with design metadata)              │
│  Order → OrderItem → Fulfillment                     │
│  Customer → Address                                  │
│  Payment → PaymentSession                            │
├─────────────────────────────────────────────────────┤
│              Custom Module Tables                    │
├─────────────────────────────────────────────────────┤
│  PrintJob                                            │
│  ├── id: string (PK)                                 │
│  ├── order_id: string (FK → Order)                   │
│  ├── status: enum                                    │
│  ├── design_png_url: string                          │
│  ├── design_json_url: string                         │
│  ├── external_id: string | null                      │
│  ├── tracking_number: string | null                  │
│  ├── created_at: timestamp                           │
│  └── updated_at: timestamp                           │
└─────────────────────────────────────────────────────┘
```

### External Integrations

```
┌────────────────────────────────────────────────────────────┐
│  AWS S3                                                     │
│  ├── Bucket: {S3_BUCKET_NAME}                               │
│  ├── Prefix: designs/                                       │
│  ├── Files: {uuid}/front.png, {uuid}/front.json            │
│  ├── Access: Presigned URLs (15min upload, public read)     │
│  └── Region: ap-southeast-1                                 │
├────────────────────────────────────────────────────────────┤
│  Print Shop API                                             │
│  ├── URL: {PRINT_SHOP_API_URL}                              │
│  ├── Auth: X-API-Key header                                 │
│  ├── Outgoing: POST /api/orders (PrintShopWebhookPayload)  │
│  ├── Incoming: POST /webhooks/print-shop (status updates)   │
│  └── Retry: 3 attempts, exponential backoff (1s, 4s, 16s)  │
├────────────────────────────────────────────────────────────┤
│  VNPay Gateway (Future)                                     │
│  ├── Payment initiation via Medusa payment provider         │
│  ├── Redirect flow                                          │
│  └── IPN callback for payment confirmation                  │
└────────────────────────────────────────────────────────────┘
```

---

## Security Considerations

| Area | Measure |
|---|---|
| S3 Upload | Presigned URLs expire after 15 minutes, server-side key generation |
| S3 Read | Public read for design files (or presigned read URLs for private buckets) |
| Cart API | Medusa publishable key required |
| Admin API | JWT auth required, separate CORS |
| Webhook endpoint | API key validation in header, skip Medusa auth |
| File upload | Client-side: type + size validation. Server-side: content-type validation |
| XSS | React auto-escapes. No `dangerouslySetInnerHTML` for user content |
| CORS | Strict: only `STORE_CORS` origins allowed |

## Performance Considerations

| Area | Strategy |
|---|---|
| Product images | Next.js Image component with S3/CDN, lazy loading |
| Design editor | Lazy import Fabric.js, code-split editor components |
| Canvas export | Web Worker for PNG generation (avoid blocking UI) |
| S3 upload | Direct upload via presigned URL (bypasses server) |
| Product data | SSR with ISR (revalidate: 60s) for product pages |
| Cart state | Client-side Zustand + Medusa API hydration |
| Bundle size | Tree-shake Fabric.js, dynamic imports for heavy components |
| Fonts | Google Fonts with `next/font`, preload critical fonts |
