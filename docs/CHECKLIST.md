# Master Checklist — Custom T-Shirt Platform

## Documents

| Document | Path | Description |
|---|---|---|
| Requirements | `docs/REQUIREMENTS.md` | Full requirements spec |
| Init Guide | `docs/INIT.md` | CLI init commands |
| Business | `docs/business.md` | Business model, pricing, operations, KPI |
| User Flow | `docs/user-flow.md` | 8-step user journey, UI/UX detail |
| System Flow | `docs/system-flow.md` | 6 technical flows, sequence diagrams, data flow |

---

## Phase 1 — Foundation

### BE01 — Backend Project Setup → [`docs/tasks/be/BE01-project-setup.md`](tasks/be/BE01-project-setup.md)
- [x] BE01.1 — Database Setup (PostgreSQL + Redis)
- [x] BE01.2 — Medusa Config (CORS, modules)
- [x] BE01.3 — Seed Product Data
- [x] BE01.4 — Region & Shipping Setup
- [x] BE01.5 — Payment Provider Setup
- [x] BE01.6 — Admin User

### FE01 — Frontend Project Setup → [`docs/tasks/fe/FE01-project-setup.md`](tasks/fe/FE01-project-setup.md)
- [x] FE01.1 — Folder Structure
- [x] FE01.2 — Environment Config (lib/env.ts)
- [x] FE01.3 — Medusa SDK Client (lib/medusa.ts)
- [x] FE01.4 — S3 Helpers (lib/s3.ts)
- [x] FE01.5 — Utility Setup (cn, fonts, prettier)
- [x] FE01.6 — Layout & Providers
- [x] FE01.7 — Shared Types

---

## Phase 2 — Backend Core

### BE02 — Product Data & API → [`docs/tasks/be/BE02-product-data.md`](tasks/be/BE02-product-data.md)
- [x] BE02.1 — Product Collections
- [x] BE02.2 — Product Options (color, size)
- [x] BE02.3 — Products (4 types)
- [x] BE02.4 — Variants Generation (color × size)
- [x] BE02.5 — Product Images
- [x] BE02.6 — Seed Script
- [x] BE02.7 — Verify Store API

### BE03 — Print Order Module → [`docs/tasks/be/BE03-print-order-module.md`](tasks/be/BE03-print-order-module.md)
- [x] BE03.1 — PrintJob Data Model
- [x] BE03.2 — PrintOrderService
- [x] BE03.3 — Module Definition
- [x] BE03.4 — Register in medusa-config
- [x] BE03.5 — Migration
- [x] BE03.6 — Unit Tests

### BE04 — Workflow & Subscriber → [`docs/tasks/be/BE04-workflow-subscriber.md`](tasks/be/BE04-workflow-subscriber.md)
- [x] BE04.1 — fetchOrderStep
- [x] BE04.2 — extractDesignDataStep
- [x] BE04.3 — createPrintJobStep
- [x] BE04.4 — createPrintJobWorkflow
- [x] BE04.5 — order-placed Subscriber
- [x] BE04.6 — Unit Tests

### BE05 — Admin API Routes → [`docs/tasks/be/BE05-webhooks-api.md`](tasks/be/BE05-webhooks-api.md)
- [x] BE05.1 — Admin Print Orders List
- [x] BE05.2 — Admin Print Order Detail
- [x] BE05.3 — Admin Update Print Job Status
- [x] BE05.4 — Admin Cancel Print Job
- [x] BE05.5 — Admin Middleware
- [x] BE05.6 — Zod Schemas
- [x] BE05.7 — Link PrintJob to Order

---

## Phase 3 — Frontend Pages

### FE02 — Landing Page → [`docs/tasks/fe/FE02-landing-page.md`](tasks/fe/FE02-landing-page.md)
- [x] FE02.1 — Page Route & Layout
- [x] FE02.2 — Hero Section
- [x] FE02.3 — How It Works Section
- [x] FE02.4 — Template Gallery Section
- [x] FE02.5 — Social Proof Section
- [x] FE02.6 — Pricing Section
- [x] FE02.7 — FAQ Section
- [x] FE02.8 — Header Component
- [x] FE02.9 — Footer Component

### FE03 — Product Pages → [`docs/tasks/fe/FE03-product-pages.md`](tasks/fe/FE03-product-pages.md)
- [x] FE03.1 — Product Listing Page
- [x] FE03.2 — Product Card Component
- [x] FE03.3 — Product Filter
- [x] FE03.4 — Product Detail Page
- [x] FE03.5 — Image Gallery
- [x] FE03.6 — Product Info Panel
- [x] FE03.7 — Size Chart
- [x] FE03.8 — Variant Selection Logic
- [x] FE03.9 — Navigate to Editor

---

## Phase 4 — Design Editor (Core Feature)

### FE04 — Design Editor → [`docs/tasks/fe/FE04-design-editor.md`](tasks/fe/FE04-design-editor.md)
- [x] FE04.1 — Zustand Design Store
- [x] FE04.2 — Canvas Initialization
- [x] FE04.3 — Print Area Overlay
- [x] FE04.4 — T-shirt Mockup Background
- [x] FE04.5 — ToolBar Component
- [x] FE04.6 — Text Tool (TextEditor)
- [x] FE04.7 — Image Upload (ImageUploader)
- [x] FE04.8 — Template Gallery
- [x] FE04.9 — Layer Panel
- [x] FE04.10 — Side Toggle (Front/Back)
- [x] FE04.11 — Keyboard Shortcuts
- [x] FE04.12 — DesignEditorRoot (Wrapper)

### FE05 — Preview & Export → [`docs/tasks/fe/FE05-preview-export.md`](tasks/fe/FE05-preview-export.md)
- [x] FE05.1 — Export Utilities (PNG + JSON)
- [x] FE05.2 — Upload Design API Route *(upload đi qua backend `PUT /store/designs/...`, không còn route ở store)*
- [x] FE05.3 — Upload Flow (Client)
- [x] FE05.4 — Preview Modal
- [x] FE05.5 — Add to Cart Flow
- [x] FE05.6 — Validation Before Export

---

## Phase 5 — Checkout Flow

### FE06 — Cart → [`docs/tasks/fe/FE06-cart.md`](tasks/fe/FE06-cart.md)
- [x] FE06.1 — Cart State Management
- [x] FE06.2 — Cart Page
- [x] FE06.3 — Cart Item Component
- [x] FE06.4 — Edit Design Flow
- [x] FE06.5 — Order Summary Sidebar
- [x] FE06.6 — Empty Cart State
- [x] FE06.7 — Cart API Integration

### FE07 — Checkout → [`docs/tasks/fe/FE07-checkout.md`](tasks/fe/FE07-checkout.md)
- [x] FE07.1 — Checkout Page
- [x] FE07.2 — Shipping Form *(Tỉnh → Phường/Xã: Việt Nam bỏ cấp Quận/Huyện từ 1/7/2025)*
- [x] FE07.3 — Payment Method Selection *(VNPay hiện "Sắp ra mắt")*
- [x] FE07.4 — Order Summary (Checkout) *(ô mã giảm giá là placeholder)*
- [x] FE07.5 — Checkout Submission (COD)
- [ ] FE07.6 — Checkout Submission (VNPay) — Future
- [x] FE07.7 — Success Page *(nút "Theo dõi đơn hàng" chờ FE08)*
- [x] FE07.8 — Zod Schemas

---

## Phase 6 — Order & Polish

### FE08 — Order Tracking → [`docs/tasks/fe/FE08-order-tracking.md`](tasks/fe/FE08-order-tracking.md)
- [x] FE08.1 — Order Tracking Page *(tra cứu qua `POST /store/order-lookup` vì `GET /store/orders/{id}` của Medusa cần đăng nhập)*
- [x] FE08.2 — Progress Tracker Component
- [x] FE08.3 — Order Details
- [x] FE08.4 — Status Mapping *(theo trạng thái print job của mỗi mặt in)*
- [x] FE08.5 — Order Lookup (No Auth) *(mã đơn + email; email không nằm trong URL)*

### FE09 — Polish & UX → [`docs/tasks/fe/FE09-polish.md`](tasks/fe/FE09-polish.md)
- [x] FE09.1 — Error Boundary
- [x] FE09.2 — Loading States
- [x] FE09.3 — Toast Notifications
- [x] FE09.4 — Mobile Responsive *(không tràn ngang ở 375/768/1024/1440 trên 7 trang; editor giữ thanh công cụ ngang phía trên thay vì thanh dưới)*
- [x] FE09.5 — Animations (motion)
- [x] FE09.6 — 404 Page
- [x] FE09.7 — SEO & Metadata *(Lighthouse mobile: trang chủ A11y 96 / BP 100 / SEO 100; trang sản phẩm 96 / 100 / 92, phần thiếu do Next 16 stream thẻ description vào body với UA thường, thẻ có trong DOM)*

### BE06 — Testing & Deploy → [`docs/tasks/be/BE06-testing-deploy.md`](tasks/be/BE06-testing-deploy.md)
- [x] BE06.1 — API Testing *(luồng khách hàng kiểm bằng `pnpm smoke` (12 bước); webhook và retry của nhà in ngoài không còn vì in thủ công)*
- [x] BE06.2 — Workflow Testing *(unit test workflow + subscriber được smoke test kích hoạt thật; không cần mock nhà in)*
- [x] BE06.3 — Environment Hardening *(giới hạn tần suất áp cho upload và tra cứu đơn, các route công khai duy nhất)*
- [x] BE06.4 — Database (indexes, backup) *(index `order_id`, `status`; `scripts/backup-db.sh` + chiến lược trong `docs/DEPLOYMENT.md`)*
- [x] BE06.5 — Docker Setup *(image build và chạy production, vượt smoke test)*
- [x] BE06.6 — Documentation *(`docs/BACKEND-API.md`, `docs/DEPLOYMENT.md`)*

---

## Dependency Graph

```
Phase 1 (Foundation)
  BE01 ──┬──▶ BE02 ──▶ BE03 ──▶ BE04 ──▶ BE05 ──▶ BE06
         │
  FE01 ──┼──▶ FE02 ──▶ FE03 ──────────────────────────▶ FE09
         │              │
         │              ▼
         └──▶ FE04 ──▶ FE05 ──▶ FE06 ──▶ FE07 ──▶ FE08
```

## Phase 7 — Design Pipeline → [`docs/DESIGN-PIPELINE-SPEC.md`](DESIGN-PIPELINE-SPEC.md)

Editor → Preview → Export cho nhà in. `[x]` = đã code + có test; mục ghi *(chưa kiểm chứng)* cần xem tay.

### DP01 — Print area derivation (`@tshirt-platform/shared`)
- [x] DP01.1 — `derivePrintArea` (mm → px → rect trên line-art), `parsePrintConfig`, colour registry
- [x] DP01.2 — Release qua GitHub Packages: `package.json`, workflow publish theo tag, `.npmrc`
- [x] DP01.3 — Publish `v1.0.0` và chuyển store/backend sang `^1.0.0`

### DP02 — Line-art calibration
- [x] DP02.1 — `detectArtCalibration` + registry có test chống trôi

### DP03 — Editor
- [x] DP03.1 — Scene 800 đơn vị, export nhân multiplier lên 300 DPI
- [x] DP03.2 — Tint màu vải, nét sáng trên vải tối; đổi màu trong editor
- [x] DP03.3 — Nhận `variantId/color/size/qty` từ URL; bảng màu và bảng size đọc từ sản phẩm
- [x] DP03.4 — Ảnh upload đặt ở 70% bề rộng; cảnh báo DPI; cảnh báo chữ khó nhìn trên màu áo

### DP04 — Export
- [x] DP04.1 — PNG nền trong suốt, đúng pixel, nhúng 300 DPI (pHYs)
- [x] DP04.2 — `/api/upload-design` (S3 presigned; lưu cục bộ khi dev), `/api/files`
- [x] DP04.3 — Upload thật lên S3 *(chưa kiểm chứng: chưa có AWS)* *(đã thử với Cloudflare R2 thật: upload qua backend, mở công khai, đọc lại scene)*

### DP05 — Render service (`tshirt-render/`)
- [x] DP05.1 — Tách áo khỏi nền, sinh bản đồ bóng/nếp vải, đổi màu, warp phối cảnh, ghép lớp
- [x] DP05.2 — API templates + render, cache, khoá API tuỳ chọn
- [x] DP05.3 — `docker compose build render`: container healthy, user không phải root, ghi được volume, bind 127.0.0.1
- [x] DP05.4 — Thử với ảnh áo thật *(1 ảnh Pexels áo thun trắng, nền đồng nhất, chạy end to end trong giao diện; áo hoodie/polo chưa có ảnh phù hợp. Lần thử làm lộ lỗi tách áo trắng trên nền nhạt, đã sửa ở tshirt-render#1)*

### DP06 — Admin
- [x] DP06.1 — Proxy `/admin/mockups*` sang service render (có xác thực, stream upload)
- [x] DP06.2 — Widget "Cấu hình in" trên trang sản phẩm: số đo, màu, mockup, kéo 4 góc
- [x] DP06.3 — Xem giao diện widget bằng tay *(qua Chrome DevTools MCP: số đo, khổ in tính được, màu vải, tải ảnh mockup, đặt 4 góc, lưu; cấu hình lưu bền và xem trước dùng ảnh áo thật)*

### DP07 — Save & Preview
- [x] DP07.1 — Nút Lưu thiết kế, modal xem trước (ảnh mockup hoặc bản phẳng), xác nhận + upload
- [x] DP07.2 — Đưa thiết kế đã lưu vào giỏ hàng (thêm, cập nhật, sửa lại từ giỏ) — xem FE06

### DP08 — Backend cho nhà in
- [x] DP08.1 — `PrintJob` thêm side/size/màu/vị trí/proof; migration; trạng thái `proof_approved`
- [x] DP08.2 — Workflow đọc metadata nhiều mặt, tạo 1 job/mặt
- [x] DP08.3 — Gói file: artwork theo tên chuẩn, proof, `spec.json`, `workorder.html`; chặn SSRF
- [x] DP08.4 — Workflow tạo print job chạy với đơn thật trong DB (đơn 2 mặt và mục kiểu cũ)

---

### Parallel Tracks
- **BE track**: BE01 → BE02 → BE03 → BE04 → BE05 → BE06
- **FE pages track**: FE01 → FE02 → FE03 → FE09
- **FE editor track**: FE01 → FE04 → FE05 → FE06 → FE07 → FE08
- FE03 + FE04 có thể chạy song song sau FE01
- BE track + FE track có thể chạy song song (FE dùng mock data ban đầu)

---

## Summary

| Category | Tasks | Subtasks |
|---|---|---|
| Backend (BE01–BE06) | 6 | 39 |
| Frontend (FE01–FE09) | 9 | 63 |
| **Total** | **15** | **102** |

| Phase | Tasks | Description |
|---|---|---|
| Phase 1 | BE01, FE01 | Foundation — setup, config |
| Phase 2 | BE02–BE05 | Backend core — products, module, workflow, API |
| Phase 3 | FE02, FE03 | Frontend pages — landing, products |
| Phase 4 | FE04, FE05 | Design editor — core feature |
| Phase 5 | FE06, FE07 | Checkout — cart, payment |
| Phase 6 | FE08, FE09, BE06 | Polish — tracking, UX, testing |
