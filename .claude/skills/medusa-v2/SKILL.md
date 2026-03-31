---
name: medusa-v2
description: Medusa.js v2 patterns — custom modules (defineModule + service), workflows (createWorkflow + createStep), subscribers, custom API routes, dependency injection. Use when building backend features.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Skill: Medusa.js v2 Patterns

## Custom Module

### 1. Define Data Model

```ts
// src/modules/<name>/models/<entity>.ts
import { model } from "@medusajs/framework/utils"

const MyEntity = model.define("my_entity", {
  id: model.id().primaryKey(),
  name: model.text(),
  status: model.enum(["active", "inactive"]),
  metadata: model.json().nullable(),
  // Timestamps (created_at, updated_at) are auto-added
})

export default MyEntity
```

### 2. Create Service

```ts
// src/modules/<name>/service.ts
import { MedusaService } from "@medusajs/framework/utils"
import MyEntity from "./models/my-entity"

class MyService extends MedusaService({ MyEntity }) {
  // Inherits: list, retrieve, create, update, delete, softDelete, restore
  async customMethod(id: string) {
    const entity = await this.retrieveMyEntity(id)
    return entity
  }
}

export default MyService
```

### 3. Module Definition

```ts
// src/modules/<name>/index.ts
import { Module } from "@medusajs/framework/utils"
import MyService from "./service"

export const MY_MODULE = "myModule"

export default Module(MY_MODULE, {
  service: MyService,
})
```

### 4. Register in Config

```ts
// medusa-config.ts
import { defineConfig } from "@medusajs/framework/utils"

export default defineConfig({
  projectConfig: { ... },
  modules: [
    { resolve: "./src/modules/<name>" },
  ],
})
```

### 5. Generate & Run Migration

```bash
pnpm medusa db:generate <migration_name>
pnpm medusa db:migrate
```

## Workflow

### Creating Steps

```ts
import { createStep, StepResponse } from "@medusajs/framework/workflows-sdk"

const myStep = createStep(
  "my-step-id",
  async (input: { data: string }, { container }) => {
    const service = container.resolve("myModule")
    const result = await service.doSomething(input.data)
    return new StepResponse(result, { id: result.id })
  },
  // Compensation (rollback)
  async (compensationData, { container }) => {
    const service = container.resolve("myModule")
    await service.undo(compensationData.id)
  }
)
```

### Creating a Workflow

```ts
import { createWorkflow, WorkflowResponse } from "@medusajs/framework/workflows-sdk"

export const myWorkflow = createWorkflow(
  "my-workflow-id",
  (input: { order_id: string }) => {
    const stepOneResult = stepOne(input)
    const stepTwoResult = stepTwo({ data: stepOneResult })
    return new WorkflowResponse(stepTwoResult)
  }
)
```

### Invoking a Workflow

```ts
const { result } = await myWorkflow(container).run({
  input: { order_id: "order_123" },
})
```

## Subscriber

```ts
// src/subscribers/<event-name>.subscriber.ts
import type { SubscriberArgs, SubscriberConfig } from "@medusajs/framework"

export default async function handler({
  event,
  container,
}: SubscriberArgs<{ id: string }>) {
  const entityId = event.data.id
  const service = container.resolve("myModule")
  await service.handleEvent(entityId)
}

export const config: SubscriberConfig = {
  event: "order.placed",
}
```

### Common Medusa Events
- `order.placed` — Order completed
- `order.updated` — Order updated
- `order.canceled` — Order canceled
- `product.created` — Product created
- `product.updated` — Product updated

## Custom API Route

### Basic Route

```ts
// src/api/store/my-route/route.ts
import type { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

export async function GET(req: MedusaRequest, res: MedusaResponse) {
  const service = req.scope.resolve("myModule")
  const data = await service.list()
  res.json({ data })
}

export async function POST(req: MedusaRequest, res: MedusaResponse) {
  const service = req.scope.resolve("myModule")
  const result = await service.create(req.body)
  res.status(201).json({ result })
}
```

### With Path Parameters

```ts
// src/api/admin/my-route/[id]/route.ts
export async function GET(req: MedusaRequest, res: MedusaResponse) {
  const { id } = req.params
  const service = req.scope.resolve("myModule")
  const item = await service.retrieve(id)
  res.json({ item })
}
```

### Route Prefixes
- `/store/...` — Customer-facing (publishable API key auth)
- `/admin/...` — Admin-only (admin auth required)
- `/webhooks/...` — External webhooks (no auth, validate manually)

## Dependency Injection

In API routes, use `req.scope`:
```ts
const orderService = req.scope.resolve("order")
const productService = req.scope.resolve("product")
const myService = req.scope.resolve("myModule")
```

## Pitfalls & Lessons

### Store API: `region_id` required for pricing
Any Store API call that includes `calculated_price` (products, variants) MUST pass `region_id`. Without it → runtime error.
```ts
// ✅ Always fetch region first
const { regions } = await medusa.store.region.list({ limit: 1 })
const regionId = regions[0].id
await medusa.store.product.list({ region_id: regionId, fields: "+variants.calculated_price" })
```

### Store API: `x-publishable-api-key` header required
All `/store/*` routes require `x-publishable-api-key` header. The JS SDK sets it automatically from `publishableKey` config. For manual testing, set header explicitly.

### defineLink: custom module `linkable` is empty at import time
Built-in modules (Order, Product, etc.) have `.linkable` populated at import. Custom modules do NOT — their `.linkable` is only populated at Medusa boot. Use manual `InputSource` shape:
```ts
defineLink(
  {
    linkable: {
      serviceName: "myModule",
      field: "myEntity",
      entity: "MyEntity",
      linkable: "my_entity_id",
      primaryKey: "id",
    },
    isList: true,
  },
  OrderModule.linkable.order
)
```

### Workflow step type compatibility
Step outputs are wrapped in `WorkflowData<T>`. When passing between steps:
- Use optional fields for arrays (e.g., `items?: Array<...>`)
- Default with `?? []` to handle undefined
- Add explicit types on compensation functions
