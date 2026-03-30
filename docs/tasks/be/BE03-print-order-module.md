# BE03 — Print Order Module

## Scope
Custom Medusa module to manage print jobs internally. No external print shop API — the admin manages job status manually, exports design files, and brings them to a print shop.

## Reference
- `.claude/skills/medusa-v2/SKILL.md`
- `.claude/skills/print-shop/SKILL.md`

## Tasks

### BE03.1 — PrintJob Data Model
- [ ] `src/modules/print-order/models/print-job.ts`
- [ ] Fields:
  - id (PK, auto-generated)
  - order_id (text, indexed)
  - status (enum: pending, processing, shipped, delivered, cancelled)
  - design_png_url (text — S3 presigned URL for design PNG)
  - design_json_url (text — S3 presigned URL for Fabric.js JSON)
  - tracking_number (text, nullable)
  - notes (text, nullable — admin notes)
  - metadata (json, nullable)
  - created_at, updated_at (auto)

### BE03.2 — PrintOrderService
- [ ] `src/modules/print-order/service.ts`
- [ ] Extends MedusaService({ PrintJob })
- [ ] Custom methods:
  - `createForOrder(data)` — create a new print job for an order
  - `updateStatus(printJobId, status, notes?)` — update job status + optional notes
  - `getByOrderId(orderId)` — retrieve print jobs for an order
  - `cancel(printJobId)` — cancel a pending/processing job

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

### BE03.6 — Unit Tests
- [ ] Test createForOrder: creates record with correct fields
- [ ] Test updateStatus: updates status + notes correctly
- [ ] Test updateStatus: rejects invalid status transitions
- [ ] Test getByOrderId: returns jobs for order
- [ ] Test cancel: sets status to cancelled
- [ ] Test cancel: rejects non-cancellable jobs (shipped/delivered)

## Acceptance Criteria
- PrintJob table exists in database
- CRUD operations work via service
- Status transitions are validated
- Module resolves correctly in container
- All unit tests pass

## Dependencies
- BE01 (database, Medusa running)

## Estimated Subtasks: 6
