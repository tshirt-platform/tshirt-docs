---
name: test-task
description: "Smart auto-test: DB check, API verify, browser E2E, unit tests — based on task type. Usage: /test-task BE03"
disable-model-invocation: true
argument-hint: "<task-id> (e.g. BE01, FE04)"
allowed-tools: Read, Bash, Glob, Grep, mcp__api-tester__*, mcp__claude-in-chrome__*
---

# Smart Test Runner: $ARGUMENTS

## Step 1 — Load Test Strategy

1. Read `docs/tasks/test-strategies.md` — find the section for `$ARGUMENTS`
2. Read the task document for acceptance criteria
3. Determine which test categories apply:
   - **DB Check** — task involves database (BE01.1, BE03.1, BE03.5)
   - **Data Verify** — task seeds or creates data (BE01.3-5, BE02)
   - **API Test** — task creates API routes (BE01.2, BE02.7, BE05)
   - **Build Check** — always applicable
   - **Unit Test** — tests written for this task
   - **E2E Browser** — task creates UI (FE02-FE09)

## Step 2 — Run Tests by Category

Execute each applicable category. Skip categories marked ⏭️ N/A.

---

### Category: Build Check ✅ (ALWAYS)

```bash
# Determine project
# BE* → tshirt-backend
# FE* → tshirt-store
cd [project-dir] && pnpm build 2>&1
```

Report:
- ✅ Build passes
- ❌ Build fails → list errors

---

### Category: DB Check 🗄️

**When**: BE01.1 (DB setup), BE03.1/BE03.5 (PrintJob model/migration)

Run via Bash:

```bash
# 1. PostgreSQL running?
pg_isready -h localhost -p 5432 2>&1

# 2. Database exists?
psql -h localhost -U postgres -d tshirt_db -c "SELECT 1" 2>&1

# 3. Tables exist? (task-specific)
# BE01: Check Medusa core tables
psql -h localhost -U postgres -d tshirt_db -c "\dt public.*" 2>&1

# BE03: Check print_job table
psql -h localhost -U postgres -d tshirt_db -c "\d print_job" 2>&1

# 4. Check columns match expected schema
psql -h localhost -U postgres -d tshirt_db -c "SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'print_job'" 2>&1
```

Report:
- ✅ PostgreSQL running
- ✅ Database `tshirt_db` exists
- ✅ Table `print_job` exists with correct columns
- ❌ Table missing / wrong schema

---

### Category: Data Verify 📦

**When**: BE01.3-5 (seed), BE02 (products/collections)

Use `mcp__api-tester`:

```
Step 1: mcp__api-tester__set_base_url({ url: "http://localhost:9000" })
Step 2: mcp__api-tester__health_check()
```

Then run task-specific checks:

**BE01.3 — Products seeded:**
```
mcp__api-tester__http_get({ url: "/store/products", params: { limit: "50" } })
→ Validate: status 200, products array not empty
```

**BE01.4 — Regions configured:**
```
mcp__api-tester__http_get({ url: "/store/regions" })
→ Validate: status 200, has region with currency_code "vnd"
```

**BE01.5 — Payment providers:**
```
mcp__api-tester__http_get({ url: "/store/payment-providers" })
→ Validate: status 200, includes provider "manual" (COD)
```

**BE02.1 — Collections:**
```
mcp__api-tester__http_get({ url: "/store/collections" })
→ Validate: 3 collections (tshirt, polo, hoodie)
```

**BE02.3 — Products with detail:**
```
mcp__api-tester__http_get({ url: "/store/products", params: { limit: "50", fields: "+variants,+images" } })
→ Validate: 4+ products, each has variants, each has images
```

**BE02.4 — Variants:**
```
# For each product
mcp__api-tester__http_get({ url: "/store/products/{id}" })
→ Validate: product.variants.length > 0, has color + size options
```

Use `mcp__api-tester__validate_response` to check each response:
```
mcp__api-tester__validate_response({
  actual: <response>,
  expected: { status: 200 }
})
```

Report:
- ✅ 4 products found
- ✅ Each product has 50 variants (10 colors × 5 sizes)
- ✅ 3 collections found
- ❌ Missing: only 2 products found (expected 4)

---

### Category: API Test 🔌

**When**: BE05 (webhooks/admin routes), BE01.2 (health), BE02.7 (store API)

Use `mcp__api-tester`:

**BE05.1 — Webhook auth:**
```
# Without API key → should 401
mcp__api-tester__http_post({
  url: "/webhooks/print-shop",
  body: { order_id: "test_order", status: "printing" }
})
→ Validate: status 401

# With API key → should 200 or 400 (depending on data)
mcp__api-tester__http_post({
  url: "/webhooks/print-shop",
  body: { order_id: "test_order", status: "printing" },
  headers: { "X-API-Key": "<from .env PRINT_SHOP_API_KEY>" }
})
→ Validate: status != 401 (auth passed)
```

**BE05.3 — Admin routes (need auth):**
```
# Without auth → should 401
mcp__api-tester__http_get({ url: "/admin/print-orders" })
→ Validate: status 401

# With admin auth
mcp__api-tester__set_bearer_token({ token: "<admin-token>" })
mcp__api-tester__http_get({ url: "/admin/print-orders" })
→ Validate: status 200, body has print_jobs array
```

**BE05.5 — Retry route:**
```
mcp__api-tester__http_post({ url: "/admin/print-orders/nonexistent/retry" })
→ Validate: status 404 or appropriate error
```

Report per endpoint:
- ✅ `POST /webhooks/print-shop` (no key) → 401
- ✅ `POST /webhooks/print-shop` (valid key) → 200
- ✅ `GET /admin/print-orders` (no auth) → 401
- ✅ `GET /admin/print-orders` (admin) → 200
- ❌ `POST /admin/print-orders/:id/retry` → 500 (unexpected)

---

### Category: Unit Tests 🧪

**When**: Tests exist for this task (check `test-strategies.md` and file system)

```bash
# Find test files for this task
# BE: look in src/modules/*/__tests__/, src/workflows/__tests__/
# FE: look in src/**/__tests__/, src/**/*.test.ts(x)

# Run tests
cd [project-dir] && pnpm test 2>&1

# Or run specific test file
cd [project-dir] && pnpm test -- [test-file-pattern] 2>&1
```

Report:
- ✅ 12/12 tests pass
- ❌ 10/12 pass, 2 failures:
  - `callWithRetry: fails after max retries` — expected Error, got undefined
  - `sendToPrintShop: creates PrintJob` — timeout

---

### Category: Browser E2E 🌐

**When**: FE02–FE09 (UI tasks)

**IMPORTANT**: Only run if `tshirt-store` dev server is running on localhost:3000.

First, check if dev server is running:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 2>/dev/null || echo "NOT_RUNNING"
```

If not running → report ⚠️ and skip browser tests.

If running, execute browser E2E:

```
# 1. Setup browser
mcp__claude-in-chrome__tabs_context_mcp({ createIfEmpty: true })
mcp__claude-in-chrome__tabs_create_mcp()

# 2. Navigate to page under test
mcp__claude-in-chrome__navigate({ url: "http://localhost:3000/[route]", tabId: <id> })

# 3. Wait for page load
mcp__claude-in-chrome__computer({ action: "wait", duration: 2, tabId: <id> })

# 4. Screenshot for visual check
mcp__claude-in-chrome__computer({ action: "screenshot", tabId: <id> })
```

Then run task-specific checks:

**FE01** (Setup):
- Navigate to `/` → page loads → no error screen
- Check console: `mcp__claude-in-chrome__read_console_messages({ tabId, pattern: "[Ee]rror" })`

**FE02** (Landing):
- Navigate to `/` → screenshot → verify Hero section visible
- Find CTA button: `mcp__claude-in-chrome__find({ query: "Thiết kế ngay", tabId })`
- Click CTA → verify URL changed to `/products`
- Navigate back → scroll down → screenshot → verify sections
- Resize to mobile (375px): `mcp__claude-in-chrome__resize_window({ width: 375, height: 812, tabId })`
- Screenshot → verify mobile layout
- Resize back: `mcp__claude-in-chrome__resize_window({ width: 1440, height: 900, tabId })`

**FE03** (Products):
- Navigate to `/products` → screenshot → verify product grid
- Find product card → click → verify navigated to detail
- On detail: find color swatch → click → screenshot
- Find size button → click → screenshot
- Find "Bắt đầu thiết kế" → verify it exists

**FE04** (Editor):
- Navigate to `/design/[id]` → wait 3s (canvas load) → screenshot
- Verify canvas element exists: `mcp__claude-in-chrome__find({ query: "canvas", tabId })`
- Find toolbar buttons → verify they exist
- Find layer panel → verify it exists
- Check console for no Fabric.js errors

**FE06** (Cart):
- Navigate to `/cart` → screenshot
- Verify empty state OR item list renders

**FE07** (Checkout):
- Navigate to `/checkout` → screenshot
- Find form fields: name, phone, email
- Fill invalid phone → tab away → screenshot (check error)

**FE08** (Order tracking):
- Navigate to `/orders/test` → screenshot
- Verify error state for invalid order OR order details

Report per check:
- ✅ Page loads at `/products` (screenshot captured)
- ✅ Product grid renders with 4+ cards
- ✅ Filter works: clicked "Polo" → 1 product shown
- ✅ No console errors
- ❌ Mobile layout: cards overflow on 375px (screenshot captured)
- ⏭️ Dev server not running — browser tests skipped

---

## Step 3 — Print Full Report

```
═══════════════════════════════════════════════
🧪 TEST REPORT — $ARGUMENTS
═══════════════════════════════════════════════

Build:          ✅ Pass
Unit Tests:     ✅ 12/12 Pass
DB Check:       ✅ All tables correct          / ⏭️ N/A
Data Verify:    ✅ 4 products, 3 collections   / ⏭️ N/A
API Tests:      ✅ 5/5 endpoints OK            / ⏭️ N/A
Browser E2E:    ✅ 8/8 checks pass             / ⏭️ N/A
Console Errors: ✅ None                        / ⏭️ N/A

═══════════════════════════════════════════════
OVERALL: ✅ ALL PASS / ❌ FAIL (N issues)
═══════════════════════════════════════════════

Issues (if any):
1. [Category] [Description] [Suggested fix]
2. ...
```

## Rules
- NEVER modify any source code — this is a read-only test runner
- ALWAYS set_base_url before API tests
- ALWAYS tabs_context_mcp before browser tests
- ALWAYS create a NEW tab for browser tests
- Check if servers are running before API/browser tests
- Report ⏭️ N/A for non-applicable categories (don't fail)
- Take screenshots as evidence for browser tests
- If a server is not running, clearly state it and skip those tests
