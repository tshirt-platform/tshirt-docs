# BE05 — Admin API Routes

## Scope
Admin routes to manage print jobs: list, view detail, update status, cancel, and download design files. No external webhooks — all management is done through the admin panel.

## Tasks

### BE05.1 — Admin Print Orders List
- [ ] `src/api/admin/print-orders/route.ts`
- [ ] GET: List all print jobs (with pagination, filters)
- [ ] Filters: status, order_id
- [ ] Response: { print_jobs, count, offset, limit }

### BE05.2 — Admin Print Order Detail
- [ ] `src/api/admin/print-orders/[id]/route.ts`
- [ ] GET: Retrieve single print job with order info
- [ ] Include: order details, design URLs, status, notes

### BE05.3 — Admin Update Print Job Status
- [ ] `src/api/admin/print-orders/[id]/route.ts`
- [ ] POST: Update print job status
- [ ] Body: { status, tracking_number?, notes? }
- [ ] Validation: only valid status transitions allowed

### BE05.4 — Admin Cancel Print Job
- [ ] `src/api/admin/print-orders/[id]/cancel/route.ts`
- [ ] POST: Cancel a pending/processing print job
- [ ] Validation: only cancel if status is pending or processing

### BE05.5 — Admin Middleware
- [ ] `src/api/middlewares.ts`
- [ ] Ensure admin auth for `/admin/print-orders/*` routes

### BE05.6 — Zod Schemas
- [ ] `updatePrintJobSchema` — validate status update body
- [ ] `printOrderQuerySchema` — validate admin list query params

### BE05.7 — Link PrintJob to Order
- [ ] Define link: PrintJob → Order (Medusa Links)
- [ ] `src/links/print-job-order.ts`
- [ ] Enable querying print jobs from order and vice versa

## Acceptance Criteria
- Admin routes return correct data (auth required)
- Status updates work with valid transitions
- Cancel works for pending/processing jobs only
- All inputs validated with Zod
- Print jobs linked to orders

## Dependencies
- BE03 (PrintOrder module)
- BE01 (Medusa auth system)

## Estimated Subtasks: 7
