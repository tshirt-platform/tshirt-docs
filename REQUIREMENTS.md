# Custom T-Shirt Platform — Requirements Spec

## What We're Building

A B2C e-commerce platform where customers design their own custom T-shirts using
an interactive canvas editor, place an order, and the design file is automatically
sent to a print shop for production.

**Two repos, deployed independently:**
- `tshirt-store` — Next.js 14 (App Router) — customer-facing storefront
- `tshirt-backend` — Medusa.js v2 — e-commerce backend + admin CRM

---

## Tech Stack

### tshirt-store
| Package | Version | Notes |
|---|---|---|
| next | 16.2 | App Router, Turbopack stable |
| react | 19.2 | View Transitions, useEffectEvent |
| typescript | 5.x strict | noImplicitAny, strictNullChecks |
| tailwindcss | 4.x | CSS-first config, no tailwind.config.js |
| shadcn/ui | latest (CLI) | `npx shadcn@latest add ...` |
| fabric | 7.2.0 | Canvas editor — ES module import |
| zustand | 5.0.12 | Bear state management |
| @medusajs/js-sdk | 2.x | Medusa store API client |
| react-hook-form | 7.71.1 | Uncontrolled forms |
| zod | 4.3.5 | Schema validation (v4 stable) |
| @hookform/resolvers | 5.2.2 | Bridge RHF + Zod |
| motion | 12.x | Animation (formerly framer-motion — package đổi tên) |
| @aws-sdk/client-s3 | 3.x | S3 presigned URL |
| @aws-sdk/s3-request-presigner | 3.x | Presigned URL generator |

> ⚠️ **Breaking change notes:**
> - `framer-motion` đã đổi tên package thành `motion` — import: `from "motion/react"`
> - Tailwind v4 không dùng `tailwind.config.js` nữa, config qua CSS file
> - Fabric.js v6→v7: import đổi từ `import { fabric } from 'fabric'` thành `import * as fabric from 'fabric'`
> - Zod v4: một số API thay đổi, dùng `import { z } from 'zod'` (không phải `zod/v4`)

### tshirt-backend
| Package | Version | Notes |
|---|---|---|
| @medusajs/medusa | 2.13.5 | Core backend |
| @medusajs/framework | 2.13.5 | Workflows, modules SDK |
| postgresql | 15+ | Database |
| redis | 7+ | Session, cache |
| node | 20+ LTS | Runtime requirement |

### Shared
- `@tshirt/shared` — npm package chứa TypeScript types dùng chung

---

## User Flow

### Step 1 — Landing page
- User vào trang chủ
- Thấy hero section với CTA "Thiết kế ngay"
- Có thể scroll xem: how it works, template gallery, social proof, pricing, FAQ

### Step 2 — Chọn sản phẩm
- User chọn loại áo: T-shirt / Polo / Hoodie
- Chọn màu sắc áo nền
- Chọn size (S / M / L / XL / XXL)
- Chọn số lượng
- Nhấn "Bắt đầu thiết kế" → chuyển sang editor

### Step 3 — Design Editor *(core feature)*
- Canvas hiển thị hình áo với vùng in được giới hạn (print area)
- User có thể:
  - Upload ảnh / logo (JPG, PNG, SVG — max 10MB)
  - Thêm text (chọn font, size, màu, căn lề)
  - Chọn template có sẵn từ thư viện
  - Kéo thả, resize, xoay từng element
  - Quản lý layers (reorder, xóa, ẩn)
  - Undo / Redo (tối thiểu 20 bước)
  - Chuyển giữa mặt trước / mặt sau
- Hệ thống cảnh báo nếu element nằm ngoài print area

### Step 4 — Preview
- User xem thiết kế được ghép lên mock-up áo thực tế
- Có thể quay lại edit
- Nhấn "Thêm vào giỏ hàng" → export + upload design → add to cart

### Step 5 — Checkout
- Điền thông tin giao hàng (tên, SĐT, địa chỉ)
- Chọn phương thức thanh toán: VNPay / COD
- Xác nhận đơn hàng

### Step 6 — Order confirmed
- Hệ thống tự động:
  - Export design PNG (3000×3000px, 300 DPI) + JSON state
  - Upload lên S3
  - Lưu URL vào order metadata
  - Gửi webhook đến Print Shop API kèm file thiết kế
- User nhận email xác nhận đơn hàng

### Step 7 — Production & Delivery
- Xưởng in nhận thông tin, in áo
- Cập nhật trạng thái đơn qua webhook ngược lại
- User theo dõi đơn hàng qua trang "Đơn của tôi"
- Nhận hàng

---

## System Flow

```
[Browser]
    │
    ├─── GET /products          ──▶  [Medusa API]  ──▶  [PostgreSQL]
    ├─── GET /products/:id      ──▶  [Medusa API]
    │
    ├─── Design on canvas (Fabric.js — client only)
    │
    ├─── POST /api/upload-design  ──▶  [Next.js API Route]
    │         └── generates presigned URL
    ├─── PUT [presigned URL]    ──▶  [AWS S3]  (PNG + JSON)
    │
    ├─── POST /store/carts/:id/line-items
    │         └── metadata: { design_png_url, design_json_url }
    │                         ──▶  [Medusa API]
    │
    ├─── POST /store/carts/:id/complete
    │                         ──▶  [Medusa API]
    │                                   │
    │                         [order.placed event]
    │                                   │
    │                         [Medusa Subscriber]
    │                                   │
    │                         [send-to-print-shop Workflow]
    │                                   │
    │                         POST ──▶  [Print Shop API]
    │                                   └── payload: order info + S3 design URLs
    │
    └─── GET /store/orders/:id  ──▶  [Medusa API]  (order tracking)
```

---

## Repositories & Structure

### tshirt-store (Next.js)

```
src/
├── app/
│   ├── (store)/
│   │   ├── page.tsx                      # Landing page
│   │   ├── products/page.tsx             # Product listing
│   │   ├── products/[id]/page.tsx        # Product detail
│   │   ├── design/[productId]/page.tsx   # Design editor
│   │   ├── cart/page.tsx
│   │   ├── checkout/page.tsx
│   │   └── checkout/success/page.tsx
│   └── api/
│       └── upload-design/route.ts        # Presigned URL generator
├── components/
│   ├── design-editor/                    # Core feature
│   │   ├── DesignEditorRoot.tsx
│   │   ├── DesignCanvas.tsx              # Fabric.js canvas
│   │   ├── ToolBar.tsx
│   │   ├── LayerPanel.tsx
│   │   ├── TextEditor.tsx
│   │   ├── ImageUploader.tsx
│   │   ├── TemplateGallery.tsx
│   │   └── PreviewModal.tsx
│   ├── product/
│   ├── checkout/
│   └── landing/
├── lib/
│   ├── env.ts                            # Typed env vars (Zod)
│   ├── medusa.ts                         # Medusa SDK singleton
│   ├── s3.ts                             # S3 upload helpers
│   ├── canvas/
│   │   ├── fabric-config.ts
│   │   ├── export.ts                     # exportToPng + exportToJson
│   │   └── constraints.ts               # Print area bounds
│   └── store/
│       └── design.store.ts              # Zustand design state
└── types/
```

### tshirt-backend (Medusa.js v2)

```
src/
├── modules/
│   └── print-order/
│       ├── index.ts
│       ├── service.ts                    # PrintOrderService
│       └── models/
│           └── print-job.ts
├── workflows/
│   └── send-to-print-shop.ts            # Medusa workflow
├── subscribers/
│   └── order-placed.subscriber.ts       # Triggers on order confirmed
└── api/
    ├── admin/print-orders/              # Custom admin routes
    └── webhooks/print-shop/             # Receive status from print shop
```

---

## Agents

### Agent 1 — `CLAUDE.md` (tshirt-store root)

Nội dung cần có:
- Project overview + tech stack
- Env vars và cách lấy qua `lib/env.ts`
- Folder structure + trách nhiệm từng file chính
- Core patterns: Medusa SDK usage, Zustand design state, S3 presigned upload, error handling
- Design editor spec: canvas constraints, export flow, print area validation
- Coding rules (xem phần dưới)
- Cross-repo dependencies

### Agent 2 — `CLAUDE.md` (tshirt-backend root)

Nội dung cần có:
- Medusa v2 module system (cách tạo custom module)
- Workflow syntax (step → workflow → invoke)
- Subscriber pattern (event → handler)
- Print shop integration: webhook payload format, retry logic
- Custom API route format trong Medusa v2
- Env vars

---

## Skills

### Skill 1 — `design-editor.skill.md`
- Fabric.js v6 khởi tạo canvas
- Print area constraint (vùng an toàn để in)
- Export PNG 300 DPI từ canvas
- Export JSON state (để user edit lại sau)
- Undo/redo với history snapshots
- Drag & drop image upload
- Lazy import Fabric.js (tránh SSR crash)

### Skill 2 — `medusa-v2.skill.md`
- Tạo custom module (`defineModule` + service class)
- Tạo workflow (`createWorkflow` + `createStep`)
- Subscribe event (`subscriber` function + `config`)
- Custom admin API route
- Inject dependencies trong service

### Skill 3 — `print-shop-integration.skill.md`
- Webhook payload format gửi đến xưởng in
- Retry logic khi print shop API down
- Nhận webhook ngược lại để cập nhật trạng thái
- S3 presigned URL vs base64 (dùng presigned URL)

---

## Coding Rules

1. TypeScript strict — không dùng `any`, không dùng `@ts-ignore`
2. Env vars luôn qua `lib/env.ts` — không dùng `process.env.X` trực tiếp
3. Không gọi `fetch` trong component — dùng lib helpers hoặc custom hooks
4. Zustand cho design state — không dùng `useState` cho canvas/design data
5. Zod schema cho mọi API route input
6. `cn()` cho conditional Tailwind class — không nối string thủ công
7. Fabric.js luôn lazy import — `dynamic(() => import('fabric'), { ssr: false })`
8. Validate print area trước khi export design
9. Max 200 lines / file — extract nếu vượt
10. Absolute imports — `@/components/...` không dùng `../../`
11. Comment code bằng tiếng Anh
12. Wrap design editor trong `<ErrorBoundary>`

---

## Key Data Shapes

### Design state (Zustand)
```ts
{
  canvas: fabric.Canvas | null
  productId: string | null
  variantId: string | null
  side: "front" | "back"
  history: string[]          // Fabric JSON snapshots
  historyIndex: number
  activeTool: "select" | "text" | "image" | "shape"
  pngUrl: string | null      // Sau khi export lên S3
  jsonUrl: string | null
}
```

### Cart line item metadata
```ts
{
  design_png_url: string     // S3 public URL — file in ấn
  design_json_url: string    // S3 public URL — file edit lại
  design_side: "front" | "back"
}
```

### Print shop webhook payload
```ts
{
  order_id: string
  customer: { name: string; phone: string; address: string }
  items: Array<{
    shirt_type: "tshirt" | "polo" | "hoodie"
    shirt_color: string
    shirt_size: string
    quantity: number
    design_png_url: string   // URL file PNG 300 DPI trên S3
    design_json_url: string
    design_side: "front" | "back"
  }>
  created_at: string
}
```

---

## Environment Variables

### tshirt-store
```bash
NEXT_PUBLIC_MEDUSA_URL=
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=
NEXT_PUBLIC_S3_BUCKET_URL=
NEXT_PUBLIC_STORE_URL=
AWS_REGION=ap-southeast-1
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
S3_BUCKET_NAME=
S3_DESIGNS_PREFIX=designs/
```

### tshirt-backend
```bash
DATABASE_URL=
REDIS_URL=
JWT_SECRET=
COOKIE_SECRET=
STORE_CORS=
ADMIN_CORS=
AUTH_CORS=
AWS_REGION=ap-southeast-1
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
S3_BUCKET_NAME=
PRINT_SHOP_API_URL=
PRINT_SHOP_API_KEY=
```

---

## What AI Should Build — Priority Order

1. **Project setup** — khởi tạo 2 repo, cài dependencies, cấu hình TypeScript/Tailwind/ESLint
2. **tshirt-backend** — Medusa setup, custom print-order module, send-to-print-shop workflow, subscriber
3. **tshirt-store / lib layer** — `env.ts`, `medusa.ts`, `s3.ts`, `design.store.ts`, canvas utils
4. **Design Editor** — `DesignCanvas`, `ToolBar`, `LayerPanel`, `TextEditor`, `ImageUploader`, `PreviewModal`
5. **Store pages** — Landing, Product listing, Product detail, Design page
6. **Checkout flow** — Cart, Checkout form, VNPay integration, Success page
7. **Admin** — Print order management trong Medusa Admin
8. **Polish** — Loading states, error boundaries, mobile responsive, animations
