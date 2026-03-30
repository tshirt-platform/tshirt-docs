# BE03 — Print Order Module

## Scope
Custom Medusa module để quản lý print jobs.

## Reference
- `.claude/skills/medusa-v2/SKILL.md`
- `.claude/skills/print-shop/SKILL.md`
- `docs/system-flow.md` → Flow 5

## Tasks

### BE03.1 — PrintJob Data Model
- [ ] `src/modules/print-order/models/print-job.ts`
- [ ] Fields:
  - id (PK, auto-generated)
  - order_id (text, indexed)
  - status (enum: pending, printing, shipped, delivered, failed)
  - design_png_url (text)
  - design_json_url (text)
  - external_id (text, nullable — print shop's job ID)
  - tracking_number (text, nullable)
  - error_message (text, nullable)
  - metadata (json, nullable)
  - created_at, updated_at (auto)

### BE03.2 — PrintOrderService
- [ ] `src/modules/print-order/service.ts`
- [ ] Extends MedusaService({ PrintJob })
- [ ] Custom methods:
  - `sendToPrintShop(payload)` — POST to print shop API with retry
  - `updatePrintJobStatus(orderId, status, trackingNumber?)` — update status
  - `getByOrderId(orderId)` — retrieve print jobs for an order
  - `retryFailedJob(printJobId)` — retry a failed job
- [ ] Private: `callWithRetry(fn, maxRetries, baseDelay)` — exponential backoff

### BE03.3 — Module Definition
- [ ] `src/modules/print-order/index.ts`
- [ ] Export `PRINT_ORDER_MODULE = "printOrder"`
- [ ] Register module with Module()

### BE03.4 — Register in medusa-config.ts
- [ ] Add `{ resolve: "./src/modules/print-order" }` to modules array

### BE03.5 — Migration
- [ ] Generate: `pnpm medusa db:generate print_order_setup`
- [ ] Run: `pnpm medusa db:migrate`
- [ ] Verify table created in PostgreSQL

### BE03.6 — Retry Logic Implementation
- [ ] Exponential backoff: 1s, 4s, 16s (base × 4^attempt)
- [ ] Max 3 retries
- [ ] On final failure: update PrintJob status to "failed", store error_message
- [ ] Log each attempt with context

### BE03.7 — Unit Tests
- [ ] Test callWithRetry: success, retry then success, all retries fail
- [ ] Test sendToPrintShop: mock axios, verify payload format
- [ ] Test updatePrintJobStatus: verify DB update

## Acceptance Criteria
- PrintJob table exists in database
- CRUD operations work via service
- sendToPrintShop sends correct payload format
- Retry logic works (tested with mock failures)
- Module resolves correctly in container

## Dependencies
- BE01 (database, Medusa running)

## Estimated Subtasks: 7
