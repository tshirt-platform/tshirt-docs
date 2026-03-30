# Master Checklist — Custom T-Shirt Platform

## Documents

| Document | Path | Description |
|---|---|---|
| Requirements | `REQUIREMENTS.md` | Full requirements spec |
| Init Guide | `INIT.md` | CLI init commands |
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
- [ ] BE02.1 — Product Collections
- [ ] BE02.2 — Product Options (color, size)
- [ ] BE02.3 — Products (4 types)
- [ ] BE02.4 — Variants Generation (color × size)
- [ ] BE02.5 — Product Images
- [ ] BE02.6 — Seed Script
- [ ] BE02.7 — Verify Store API

### BE03 — Print Order Module → [`docs/tasks/be/BE03-print-order-module.md`](tasks/be/BE03-print-order-module.md)
- [ ] BE03.1 — PrintJob Data Model
- [ ] BE03.2 — PrintOrderService
- [ ] BE03.3 — Module Definition
- [ ] BE03.4 — Register in medusa-config
- [ ] BE03.5 — Migration
- [ ] BE03.6 — Retry Logic Implementation
- [ ] BE03.7 — Unit Tests

### BE04 — Workflow & Subscriber → [`docs/tasks/be/BE04-workflow-subscriber.md`](tasks/be/BE04-workflow-subscriber.md)
- [ ] BE04.1 — fetchOrderStep
- [ ] BE04.2 — buildPayloadStep
- [ ] BE04.3 — sendToPrintShopStep
- [ ] BE04.4 — sendToPrintShopWorkflow
- [ ] BE04.5 — order-placed Subscriber
- [ ] BE04.6 — Integration Test

### BE05 — Webhooks & API Routes → [`docs/tasks/be/BE05-webhooks-api.md`](tasks/be/BE05-webhooks-api.md)
- [ ] BE05.1 — Print Shop Webhook Route
- [ ] BE05.2 — Webhook Middleware
- [ ] BE05.3 — Admin Print Orders List
- [ ] BE05.4 — Admin Print Order Detail
- [ ] BE05.5 — Admin Retry Failed Job
- [ ] BE05.6 — Zod Schemas
- [ ] BE05.7 — Link PrintJob to Order

---

## Phase 3 — Frontend Pages

### FE02 — Landing Page → [`docs/tasks/fe/FE02-landing-page.md`](tasks/fe/FE02-landing-page.md)
- [ ] FE02.1 — Page Route & Layout
- [ ] FE02.2 — Hero Section
- [ ] FE02.3 — How It Works Section
- [ ] FE02.4 — Template Gallery Section
- [ ] FE02.5 — Social Proof Section
- [ ] FE02.6 — Pricing Section
- [ ] FE02.7 — FAQ Section
- [ ] FE02.8 — Header Component
- [ ] FE02.9 — Footer Component

### FE03 — Product Pages → [`docs/tasks/fe/FE03-product-pages.md`](tasks/fe/FE03-product-pages.md)
- [ ] FE03.1 — Product Listing Page
- [ ] FE03.2 — Product Card Component
- [ ] FE03.3 — Product Filter
- [ ] FE03.4 — Product Detail Page
- [ ] FE03.5 — Image Gallery
- [ ] FE03.6 — Product Info Panel
- [ ] FE03.7 — Size Chart
- [ ] FE03.8 — Variant Selection Logic
- [ ] FE03.9 — Navigate to Editor

---

## Phase 4 — Design Editor (Core Feature)

### FE04 — Design Editor → [`docs/tasks/fe/FE04-design-editor.md`](tasks/fe/FE04-design-editor.md)
- [ ] FE04.1 — Zustand Design Store
- [ ] FE04.2 — Canvas Initialization
- [ ] FE04.3 — Print Area Overlay
- [ ] FE04.4 — T-shirt Mockup Background
- [ ] FE04.5 — ToolBar Component
- [ ] FE04.6 — Text Tool (TextEditor)
- [ ] FE04.7 — Image Upload (ImageUploader)
- [ ] FE04.8 — Template Gallery
- [ ] FE04.9 — Layer Panel
- [ ] FE04.10 — Side Toggle (Front/Back)
- [ ] FE04.11 — Keyboard Shortcuts
- [ ] FE04.12 — DesignEditorRoot (Wrapper)

### FE05 — Preview & Export → [`docs/tasks/fe/FE05-preview-export.md`](tasks/fe/FE05-preview-export.md)
- [ ] FE05.1 — Export Utilities (PNG + JSON)
- [ ] FE05.2 — Upload Design API Route
- [ ] FE05.3 — Upload Flow (Client)
- [ ] FE05.4 — Preview Modal
- [ ] FE05.5 — Add to Cart Flow
- [ ] FE05.6 — Validation Before Export

---

## Phase 5 — Checkout Flow

### FE06 — Cart → [`docs/tasks/fe/FE06-cart.md`](tasks/fe/FE06-cart.md)
- [ ] FE06.1 — Cart State Management
- [ ] FE06.2 — Cart Page
- [ ] FE06.3 — Cart Item Component
- [ ] FE06.4 — Edit Design Flow
- [ ] FE06.5 — Order Summary Sidebar
- [ ] FE06.6 — Empty Cart State
- [ ] FE06.7 — Cart API Integration

### FE07 — Checkout → [`docs/tasks/fe/FE07-checkout.md`](tasks/fe/FE07-checkout.md)
- [ ] FE07.1 — Checkout Page
- [ ] FE07.2 — Shipping Form
- [ ] FE07.3 — Payment Method Selection
- [ ] FE07.4 — Order Summary (Checkout)
- [ ] FE07.5 — Checkout Submission (COD)
- [ ] FE07.6 — Checkout Submission (VNPay) — Future
- [ ] FE07.7 — Success Page
- [ ] FE07.8 — Zod Schemas

---

## Phase 6 — Order & Polish

### FE08 — Order Tracking → [`docs/tasks/fe/FE08-order-tracking.md`](tasks/fe/FE08-order-tracking.md)
- [ ] FE08.1 — Order Tracking Page
- [ ] FE08.2 — Progress Tracker Component
- [ ] FE08.3 — Order Details
- [ ] FE08.4 — Status Mapping
- [ ] FE08.5 — Order Lookup (No Auth)

### FE09 — Polish & UX → [`docs/tasks/fe/FE09-polish.md`](tasks/fe/FE09-polish.md)
- [ ] FE09.1 — Error Boundary
- [ ] FE09.2 — Loading States
- [ ] FE09.3 — Toast Notifications
- [ ] FE09.4 — Mobile Responsive
- [ ] FE09.5 — Animations (motion)
- [ ] FE09.6 — 404 Page
- [ ] FE09.7 — SEO & Metadata

### BE06 — Testing & Deploy → [`docs/tasks/be/BE06-testing-deploy.md`](tasks/be/BE06-testing-deploy.md)
- [ ] BE06.1 — API Testing
- [ ] BE06.2 — Workflow Testing
- [ ] BE06.3 — Environment Hardening
- [ ] BE06.4 — Database (indexes, backup)
- [ ] BE06.5 — Docker Setup
- [ ] BE06.6 — Documentation

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
