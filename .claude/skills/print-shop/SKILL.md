---
name: print-shop
description: Print order management — internal print job tracking, status flow, admin management, S3 design file storage. Use when working on print order features.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Skill: Print Order Management

## Overview
When an order is placed, the system automatically creates a PrintJob record. The admin then manages the job manually: downloads design files, brings them to a print shop, and updates status as the job progresses. There is no external print shop API integration.

## Print Job Status Flow
```
pending → processing → shipped → delivered
    ↘         ↘
   cancelled  cancelled
```

### Status Definitions
| Status | Meaning |
|---|---|
| `pending` | Order placed, awaiting admin action |
| `processing` | Admin has started working on the print job |
| `shipped` | Printed and shipped to customer |
| `delivered` | Customer received the order |
| `cancelled` | Job cancelled (only from pending/processing) |

### Valid Transitions
```ts
const VALID_TRANSITIONS: Record<string, string[]> = {
  pending: ["processing", "cancelled"],
  processing: ["shipped", "cancelled"],
  shipped: ["delivered"],
  delivered: [],     // terminal
  cancelled: [],     // terminal
}
```

## PrintJob Data Model

```ts
// src/modules/print-order/models/print-job.ts
import { model } from "@medusajs/framework/utils"

const PrintJob = model.define("print_job", {
  id: model.id().primaryKey(),
  order_id: model.text(),
  status: model.enum(["pending", "processing", "shipped", "delivered", "cancelled"]),
  design_png_url: model.text(),
  design_json_url: model.text(),
  tracking_number: model.text().nullable(),
  notes: model.text().nullable(),
  metadata: model.json().nullable(),
})

export default PrintJob
```

## PrintOrderService

```ts
// src/modules/print-order/service.ts
class PrintOrderService extends MedusaService({ PrintJob }) {
  // Create a print job for an order
  async createForOrder(data: {
    order_id: string
    design_png_url: string
    design_json_url: string
  }): Promise<PrintJob> {
    return this.createPrintJobs({
      ...data,
      status: "pending",
    })
  }

  // Update status with transition validation
  async updateStatus(
    printJobId: string,
    status: string,
    notes?: string
  ): Promise<void> {
    const job = await this.retrievePrintJob(printJobId)
    // Validate transition
    if (!VALID_TRANSITIONS[job.status]?.includes(status)) {
      throw new Error(`Cannot transition from ${job.status} to ${status}`)
    }
    await this.updatePrintJobs(printJobId, {
      status,
      ...(notes !== undefined ? { notes } : {}),
    })
  }

  // Get all print jobs for an order
  async getByOrderId(orderId: string) {
    return this.listPrintJobs({ order_id: orderId })
  }

  // Cancel a pending/processing job
  async cancel(printJobId: string): Promise<void> {
    await this.updateStatus(printJobId, "cancelled")
  }
}
```

## Workflow: create-print-job

```ts
// Triggered by order.placed subscriber
export const createPrintJobWorkflow = createWorkflow(
  "create-print-job",
  (input: { order_id: string }) => {
    const order = fetchOrderStep(input)
    const designData = extractDesignDataStep({ order })
    const printJob = createPrintJobStep(designData)
    return new WorkflowResponse(printJob)
  }
)
```

## Admin Routes

| Method | Route | Action |
|---|---|---|
| GET | `/admin/print-orders` | List all print jobs (paginated, filterable) |
| GET | `/admin/print-orders/:id` | Get print job detail |
| POST | `/admin/print-orders/:id` | Update status (+ tracking_number, notes) |
| POST | `/admin/print-orders/:id/cancel` | Cancel a job |

## S3 Design Files

### Why Presigned URLs
- Design PNGs are ~5-15MB at 3000x3000 300DPI
- Presigned URLs allow direct download from S3
- URLs are time-limited (default 24h)

### URL Structure
```
s3://{bucket}/designs/{order_id}/{side}.png
s3://{bucket}/designs/{order_id}/{side}.json
```

### Admin Workflow
1. Admin sees new pending print job in admin panel
2. Downloads design PNG + JSON from S3 URLs
3. Brings files to print shop / prints locally
4. Updates status: pending → processing → shipped → delivered
5. Adds tracking number when shipped
