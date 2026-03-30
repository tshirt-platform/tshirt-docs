---
name: print-shop
description: Print shop integration — webhook payload format, retry logic with exponential backoff, incoming webhook handler, S3 presigned URL strategy. Use when working on print shop communication.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Skill: Print Shop Integration

## Overview
When an order is placed, the system automatically sends design files and order details to the print shop API. The print shop sends status updates back via webhook.

## Outgoing Webhook Payload

```ts
// Sent to PRINT_SHOP_API_URL when order.placed fires
interface PrintShopPayload {
  order_id: string
  customer: {
    name: string
    phone: string
    address: string
  }
  items: Array<{
    shirt_type: "tshirt" | "polo" | "hoodie"
    shirt_color: string
    shirt_size: string
    quantity: number
    design_png_url: string   // S3 URL — 3000x3000px 300 DPI PNG
    design_json_url: string  // S3 URL — Fabric.js JSON state
    design_side: "front" | "back"
  }>
  created_at: string         // ISO 8601
}
```

## Sending to Print Shop

```ts
// src/modules/print-order/service.ts
import axios from "axios"

class PrintOrderService extends MedusaService({ PrintJob }) {
  async sendToPrintShop(payload: PrintShopPayload): Promise<string> {
    const response = await this.callWithRetry(
      () => axios.post(
        process.env.PRINT_SHOP_API_URL!,
        payload,
        {
          headers: {
            "Content-Type": "application/json",
            "X-API-Key": process.env.PRINT_SHOP_API_KEY!,
          },
          timeout: 10_000,
        }
      )
    )

    const printJob = await this.createPrintJobs({
      order_id: payload.order_id,
      status: "pending",
      design_png_url: payload.items[0].design_png_url,
      design_json_url: payload.items[0].design_json_url,
      external_id: response.data.job_id ?? null,
    })

    return printJob.id
  }
}
```

## Retry Logic

Exponential backoff with max 3 retries.

```ts
private async callWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelay = 1000,
): Promise<T> {
  let lastError: Error | undefined

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn()
    } catch (error) {
      lastError = error as Error
      if (attempt === maxRetries) break
      // Exponential backoff: 1s, 4s, 16s
      const delay = baseDelay * Math.pow(4, attempt)
      await new Promise((resolve) => setTimeout(resolve, delay))
    }
  }

  throw lastError
}
```

### When retry exhausts:
1. Log the failure with full context (order_id, attempt count, error)
2. Update PrintJob status to `"failed"`
3. The admin can manually retry from the admin panel

## Incoming Webhook (from Print Shop)

```ts
// src/api/webhooks/print-shop/route.ts
import type { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

export async function POST(req: MedusaRequest, res: MedusaResponse) {
  // 1. Validate webhook authenticity
  const apiKey = req.headers["x-api-key"]
  if (apiKey !== process.env.PRINT_SHOP_API_KEY) {
    res.status(401).json({ error: "Unauthorized" })
    return
  }

  // 2. Parse payload
  const { order_id, status, tracking_number } = req.body as {
    order_id: string
    status: "printing" | "shipped" | "delivered" | "failed"
    tracking_number?: string
  }

  // 3. Update print job
  const printOrderService = req.scope.resolve("printOrder")
  await printOrderService.updatePrintJobStatus(order_id, status, tracking_number)

  // 4. Optionally update Medusa order fulfillment
  if (status === "shipped" && tracking_number) {
    // Create fulfillment with tracking info
  }

  res.sendStatus(200)
}
```

### Status Flow
```
pending → printing → shipped → delivered
                ↘ failed (at any step)
```

### Webhook Security
- Validate `X-API-Key` header matches `PRINT_SHOP_API_KEY`
- Skip Medusa auth middleware for `/webhooks/*` routes
- Log all incoming webhooks for debugging

## S3 Design Files

### Why Presigned URLs (not base64)
- Design PNGs are ~5-15MB at 3000x3000 300DPI
- Base64 increases size by ~33%
- Presigned URLs let the print shop download directly from S3
- URLs are time-limited (default 24h) — generate fresh URLs if needed

### URL Structure
```
s3://{bucket}/{prefix}{order_id}/{side}.png
s3://{bucket}/{prefix}{order_id}/{side}.json

Example:
s3://tshirt-designs/designs/order_abc123/front.png
s3://tshirt-designs/designs/order_abc123/front.json
```

## Workflow: send-to-print-shop

```ts
export const sendToPrintShopWorkflow = createWorkflow(
  "send-to-print-shop",
  (input: { order_id: string }) => {
    const order = fetchOrderStep(input)
    const payload = buildPayloadStep({ order })
    const printJob = sendToPrintShopStep({ payload })
    return new WorkflowResponse(printJob)
  }
)
```

### Compensation
If `sendToPrintShopStep` fails after creating a print job:
- Cancel the print job via print shop API (if external_id exists)
- Update local PrintJob status to `"failed"`
