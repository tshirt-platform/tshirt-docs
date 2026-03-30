# BE04 — Workflow & Subscriber

## Scope
create-print-job workflow + order.placed subscriber. When an order is placed, automatically create a PrintJob record so the admin can track and manage it.

## Reference
- `.claude/skills/medusa-v2/SKILL.md`

## Tasks

### BE04.1 — fetchOrderStep
- [ ] `src/workflows/steps/fetch-order.ts`
- [ ] Input: `{ order_id: string }`
- [ ] Action: resolve "order" service → retrieve order with line items + metadata
- [ ] Output: StepResponse with full order data
- [ ] Error: throw if order not found

### BE04.2 — extractDesignDataStep
- [ ] `src/workflows/steps/extract-design-data.ts`
- [ ] Input: `{ order }` from previous step
- [ ] Action: extract design file URLs from order item metadata
  - Each line item should have `design_png_url` and `design_json_url` in metadata
  - Validate that design URLs exist
- [ ] Output: StepResponse with `{ order_id, items: [{ design_png_url, design_json_url }] }`

### BE04.3 — createPrintJobStep
- [ ] `src/workflows/steps/create-print-job.ts`
- [ ] Input: `{ order_id, items }` from previous step
- [ ] Action: resolve "printOrder" service → createForOrder() for each item
- [ ] Output: StepResponse with { print_job_ids }
- [ ] Compensation: cancel created print jobs on failure

### BE04.4 — createPrintJobWorkflow
- [ ] `src/workflows/create-print-job.ts`
- [ ] Compose: fetchOrderStep → extractDesignDataStep → createPrintJobStep
- [ ] Input: `{ order_id: string }`
- [ ] Output: WorkflowResponse with print job result

### BE04.5 — order-placed Subscriber
- [ ] `src/subscribers/order-placed.subscriber.ts`
- [ ] Event: `order.placed`
- [ ] Handler: invoke createPrintJobWorkflow with order_id
- [ ] Error handling: log failure (don't re-throw — avoid blocking order)

### BE04.6 — Unit Tests
- [ ] Test each step individually with mocks
- [ ] Test full workflow with mocks
- [ ] Test subscriber triggers workflow
- [ ] Test compensation on failure

## Acceptance Criteria
- order.placed event triggers subscriber
- Workflow executes all steps in order
- PrintJob record created with correct design URLs
- Compensation works on failure
- Errors logged but don't block order

## Dependencies
- BE03 (PrintOrder module)
- BE01 (Medusa running, database)

## Estimated Subtasks: 6
