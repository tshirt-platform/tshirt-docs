# BE04 — Workflow & Subscriber

## Scope
send-to-print-shop workflow + order.placed subscriber.

## Reference
- `.claude/skills/medusa-v2/SKILL.md`
- `docs/system-flow.md` → Flow 5

## Tasks

### BE04.1 — fetchOrderStep
- [ ] `src/workflows/steps/fetch-order.ts`
- [ ] Input: `{ order_id: string }`
- [ ] Action: resolve "order" service → retrieve order with line items + metadata
- [ ] Output: StepResponse with full order data
- [ ] Error: throw if order not found

### BE04.2 — buildPayloadStep
- [ ] `src/workflows/steps/build-payload.ts`
- [ ] Input: `{ order }` from previous step
- [ ] Action: transform Medusa order → PrintShopWebhookPayload
  - order.shipping_address → customer (name, phone, address)
  - order.items → items (extract design URLs from metadata, variant info)
  - Validate: all items have design_png_url in metadata
- [ ] Output: StepResponse with payload
- [ ] Use types from `@tshirt/shared`

### BE04.3 — sendToPrintShopStep
- [ ] `src/workflows/steps/send-to-print-shop.ts`
- [ ] Input: `{ payload: PrintShopWebhookPayload }`
- [ ] Action: resolve "printOrder" service → sendToPrintShop(payload)
- [ ] Output: StepResponse with { print_job_id }
- [ ] Compensation: cancel print job if exists (update status to "failed")

### BE04.4 — sendToPrintShopWorkflow
- [ ] `src/workflows/send-to-print-shop.ts`
- [ ] Compose: fetchOrderStep → buildPayloadStep → sendToPrintShopStep
- [ ] Input: `{ order_id: string }`
- [ ] Output: WorkflowResponse with print job result

### BE04.5 — order-placed Subscriber
- [ ] `src/subscribers/order-placed.subscriber.ts`
- [ ] Event: `order.placed`
- [ ] Handler: invoke sendToPrintShopWorkflow with order_id
- [ ] Error handling: log failure (don't re-throw — avoid blocking order)

### BE04.6 — Integration Test
- [ ] Test full flow: create order → subscriber fires → workflow runs
- [ ] Mock print shop API → verify payload
- [ ] Verify PrintJob created in database
- [ ] Test compensation: print shop fails → PrintJob status = "failed"

## Acceptance Criteria
- order.placed event triggers subscriber
- Workflow executes all steps in order
- PrintShopWebhookPayload matches expected format
- PrintJob record created with correct data
- Compensation works on failure
- Errors logged but don't block order

## Dependencies
- BE03 (PrintOrder module)
- BE01 (Medusa running, database)

## Estimated Subtasks: 6
