# BE05 — Webhooks & Custom API Routes

## Scope
Print shop incoming webhook + admin print order routes.

## Reference
- `.claude/skills/print-shop/SKILL.md`
- `docs/system-flow.md` → Flow 5

## Tasks

### BE05.1 — Print Shop Webhook Route
- [ ] `src/api/webhooks/print-shop/route.ts`
- [ ] POST handler:
  1. Validate X-API-Key header
  2. Parse body: { order_id, status, tracking_number? }
  3. Validate with Zod schema
  4. Update PrintJob via printOrderService
  5. If shipped + tracking → update order fulfillment
  6. Return 200
- [ ] Error responses: 401 (bad API key), 400 (validation), 500

### BE05.2 — Webhook Middleware
- [ ] `src/api/middlewares.ts`
- [ ] Skip Medusa auth for `/webhooks/*` routes
- [ ] Add request logging for webhook routes
- [ ] Rate limiting (optional, future)

### BE05.3 — Admin Print Orders List
- [ ] `src/api/admin/print-orders/route.ts`
- [ ] GET: List all print jobs (with pagination, filters)
- [ ] Filters: status, date range, order_id
- [ ] Response: { print_jobs, count }

### BE05.4 — Admin Print Order Detail
- [ ] `src/api/admin/print-orders/[id]/route.ts`
- [ ] GET: Retrieve single print job with order info
- [ ] Include: order details, design URLs, status history

### BE05.5 — Admin Retry Failed Job
- [ ] `src/api/admin/print-orders/[id]/retry/route.ts`
- [ ] POST: Retry a failed print job
- [ ] Action: re-invoke sendToPrintShop with original payload
- [ ] Update status back to "pending"
- [ ] Validation: only retry if status === "failed"

### BE05.6 — Zod Schemas
- [ ] `webhookPayloadSchema` — validate incoming print shop webhook
- [ ] `printOrderQuerySchema` — validate admin list query params
- [ ] `retryRequestSchema` — validate retry request

### BE05.7 — Link PrintJob to Order
- [ ] Define link: PrintJob → Order (Medusa Links)
- [ ] `src/links/print-job-order.ts`
- [ ] Enable querying print jobs from order and vice versa

## Acceptance Criteria
- Webhook receives and processes status updates
- API key validation works (reject invalid keys)
- Admin routes return correct data (auth required)
- Retry mechanism works for failed jobs
- All inputs validated with Zod

## Dependencies
- BE03 (PrintOrder module)
- BE04 (workflow)
- BE01 (Medusa auth system)

## Estimated Subtasks: 7
