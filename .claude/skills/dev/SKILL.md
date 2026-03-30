---
name: dev
description: "Execute a development task end-to-end: check status → plan → code → write tests → verify → fix → update checklist. Usage: /dev BE01 or /dev FE04"
disable-model-invocation: true
argument-hint: "<task-id> (e.g. BE01, FE04, BE03.2)"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent, mcp__api-tester__*, mcp__claude-in-chrome__*
---

# Development Workflow

## Task ID: $ARGUMENTS

## Step 0 — Load Context

Read the following files to understand the task:

1. **Read the master checklist** at `docs/CHECKLIST.md` to determine if `$ARGUMENTS` is already completed.
2. **Read the task document**:
   - If task starts with `BE` → read `docs/tasks/be/$ARGUMENTS-*.md` (glob match)
   - If task starts with `FE` → read `docs/tasks/fe/$ARGUMENTS-*.md` (glob match)
3. **Read the test strategy** for this task at `docs/tasks/test-strategies.md` — find the section for `$ARGUMENTS` to know:
   - Which test categories apply (DB Check, Data Verify, API Test, Unit Test, E2E Browser)
   - Which unit tests to write
   - Which E2E tests to run
4. **Read the relevant CLAUDE.md** for coding patterns:
   - BE tasks → `tshirt-backend/CLAUDE.md`
   - FE tasks → `tshirt-store/CLAUDE.md`
5. **Read the relevant skill** if applicable:
   - Print order tasks (BE03, BE04, BE05) → `.claude/skills/print-shop/SKILL.md`
   - Design editor tasks (FE04, FE05) → `.claude/skills/design-editor/SKILL.md`
   - Medusa module tasks (BE03, BE04) → `.claude/skills/medusa-v2/SKILL.md`

## Step 1 — Check Status

From the checklist, determine the status of `$ARGUMENTS`:

- If ALL subtasks of `$ARGUMENTS` are marked `[x]` → **STOP**. Print:
  ```
  ✅ Task $ARGUMENTS is already completed. All subtasks done.
  ```
  List the completed subtasks and exit.

- If SOME subtasks are `[x]` and some `[ ]` → Print:
  ```
  ⏳ Task $ARGUMENTS is partially completed. Resuming from remaining subtasks.
  ```
  List which are done and which remain. Continue with remaining only.

- If NO subtasks are `[x]` → Print:
  ```
  🆕 Starting task $ARGUMENTS from scratch.
  ```
  Continue with all subtasks.

## Step 2 — Plan

Before writing any code:

1. Read the task document thoroughly (all subtasks, acceptance criteria, dependencies)
2. Check if dependencies are met:
   - Read the checklist to verify dependent tasks are completed
   - If dependencies NOT met → **STOP**. Print:
     ```
     ⚠️ Cannot start $ARGUMENTS. Missing dependencies: [list]
     Complete those tasks first.
     ```
3. Create a brief implementation plan:
   - List files to create/modify
   - Note any edge cases
   - Identify which tests to write (from test-strategies.md)
4. Print the plan and proceed

## Step 3 — Code

For each uncompleted subtask, in order:

1. Print which subtask you're working on: `📝 Working on $ARGUMENTS.N — [subtask name]`
2. Implement the subtask following:
   - Coding rules from the project CLAUDE.md
   - Patterns from the relevant skill files
   - Max 200 lines per file
   - TypeScript strict, no `any`
   - Comments in English
3. After completing each subtask, print: `✅ $ARGUMENTS.N done`

## Step 4 — Write Tests

After implementation, write tests as specified in `docs/tasks/test-strategies.md`:

### 4A — Unit Tests

Look up the "Unit Tests to Write" section for `$ARGUMENTS` in test-strategies.md.

For **BE tasks** (tshirt-backend):
- Create test files alongside source: `src/modules/.../___tests___/service.spec.ts`
- Use Medusa's test utilities or vitest
- Mock external services (print shop API, S3)
- Test file naming: `[feature].spec.ts`

For **FE tasks** (tshirt-store):
- Create test files: `src/[path]/___tests___/[component].test.tsx` or `.test.ts`
- Use vitest + @testing-library/react (if configured)
- Mock Medusa SDK, S3, Fabric.js
- Test file naming: `[component].test.tsx`

### 4B — E2E / Integration Tests

Write test scenarios but also RUN them immediately (see Step 5).

## Step 5 — Verify (Automated)

Run ALL applicable verification checks. Determine which checks apply from the test strategy:

### 5A — Build Check (ALWAYS)
```bash
# BE tasks
cd tshirt-backend && pnpm build 2>&1

# FE tasks
cd tshirt-store && pnpm tsc --noEmit 2>&1
```

### 5B — Unit Tests (ALWAYS if tests were written in Step 4)
```bash
# BE
cd tshirt-backend && pnpm test 2>&1

# FE
cd tshirt-store && pnpm test 2>&1
```

### 5C — DB Check (for BE01, BE03 — database tasks)

Use Bash to verify database state:
```bash
# Check PostgreSQL connection
pg_isready -h localhost -p 5432

# Check specific table exists
psql -h localhost -U postgres -d tshirt_db -c "\dt print_job" 2>&1

# Check migration applied
psql -h localhost -U postgres -d tshirt_db -c "SELECT COUNT(*) FROM print_job" 2>&1
```

### 5D — API Verification (for BE01, BE02, BE04, BE05 — API tasks)

Use `mcp__api-tester` tools:

1. Set base URL:
   ```
   mcp__api-tester__set_base_url({ url: "http://localhost:9000" })
   ```

2. Health check:
   ```
   mcp__api-tester__health_check()
   ```

3. Test specific endpoints per task:

   **BE01** (backend running):
   ```
   mcp__api-tester__http_get({ url: "/store/products" })
   → validate: status 200
   ```

   **BE02** (products seeded):
   ```
   mcp__api-tester__http_get({ url: "/store/products", params: { limit: "50" } })
   → validate: status 200, body.products.length >= 4

   mcp__api-tester__http_get({ url: "/store/collections" })
   → validate: status 200, body.collections.length >= 3
   ```

   **BE05** (webhook routes):
   ```
   mcp__api-tester__http_post({
     url: "/webhooks/print-shop",
     body: { order_id: "test", status: "printing" }
   })
   → without API key: validate status 401

   mcp__api-tester__http_post({
     url: "/webhooks/print-shop",
     body: { order_id: "test", status: "printing" },
     headers: { "X-API-Key": "test-key" }
   })
   → validate based on config
   ```

4. Validate responses:
   ```
   mcp__api-tester__validate_response({
     actual: <response>,
     expected: { status: 200, body: { ... } }
   })
   ```

### 5E — Browser E2E (for FE02–FE09 — UI tasks)

Use `mcp__claude-in-chrome` tools:

1. Get browser context:
   ```
   mcp__claude-in-chrome__tabs_context_mcp({ createIfEmpty: true })
   ```

2. Create new tab:
   ```
   mcp__claude-in-chrome__tabs_create_mcp()
   ```

3. Navigate to page under test:
   ```
   mcp__claude-in-chrome__navigate({ url: "http://localhost:3000", tabId: <id> })
   ```

4. Take screenshot and verify layout:
   ```
   mcp__claude-in-chrome__computer({ action: "screenshot", tabId: <id> })
   ```

5. Check for console errors:
   ```
   mcp__claude-in-chrome__read_console_messages({ tabId: <id>, pattern: "error|Error|ERR" })
   ```

6. Test interactions per task:

   **FE02** (Landing page):
   - Navigate to `/` → screenshot → verify Hero visible
   - Find "Thiết kế ngay" button → click → verify URL is `/products`
   - Navigate back → scroll down → verify sections render
   - Resize to 375px width → screenshot → verify mobile layout

   **FE03** (Product pages):
   - Navigate to `/products` → verify product cards render
   - Find filter button → click → verify filtered results
   - Click product card → verify `/products/:id` loads
   - Find color swatch → click → verify mockup updates
   - Find size button → click → verify selection state
   - Find "Bắt đầu thiết kế" → click → verify navigation

   **FE04** (Design editor):
   - Navigate to `/design/:id` → verify canvas loads
   - Find text tool → click → click canvas → verify text added
   - Find undo button → click → verify text removed
   - Find redo button → click → verify text restored
   - Find layer panel → verify element listed
   - Take screenshot → verify layout correct

   **FE06** (Cart):
   - Navigate to `/cart` → verify items or empty state
   - If items: change quantity → verify total updates
   - If items: click delete → verify item removed

   **FE07** (Checkout):
   - Navigate to `/checkout` → verify form renders
   - Fill invalid phone → blur → verify error message
   - Fill all valid fields → submit → verify success page

7. Record GIF for visual evidence (optional):
   ```
   mcp__claude-in-chrome__gif_creator({ action: "start_recording", tabId: <id> })
   ... perform actions ...
   mcp__claude-in-chrome__gif_creator({ action: "stop_recording", tabId: <id> })
   mcp__claude-in-chrome__gif_creator({ action: "export", tabId: <id>, download: true, filename: "FE02-landing-test.gif" })
   ```

### 5F — Print Test Report

```
🧪 Test Report for $ARGUMENTS

Build:        ✅ Pass / ❌ Fail
Unit Tests:   ✅ 12/12 Pass / ❌ 10/12 Pass (2 failures)
DB Check:     ✅ Tables exist / ❌ Missing table X / ⏭️ N/A
API Tests:    ✅ 5/5 endpoints OK / ❌ 3/5 (failures listed) / ⏭️ N/A
Browser E2E:  ✅ All checks pass / ❌ Issues found / ⏭️ N/A
Console:      ✅ No errors / ❌ N errors found

Overall: ✅ ALL PASS / ❌ FAIL (N issues)
```

## Step 6 — Fix Loop

If ANY test fails:

1. Print: `🔧 Fixing: [category] — [error description]`
2. Diagnose root cause — read the error carefully
3. Fix the issue in source code
4. If unit test failed → fix test or source → re-run unit tests
5. If API test failed → fix route/service → re-run API test
6. If browser E2E failed → fix component → refresh browser → re-check
7. If build failed → fix TypeScript errors → re-build
8. Repeat until all checks pass (max 5 iterations per issue)
9. If stuck after 5 attempts on same issue:
   ```
   ⚠️ Could not auto-fix after 5 attempts:
   [describe the issue and what was tried]
   Needs manual review.
   ```

## Step 7 — Update Checklist

Once ALL tests pass:

1. **Update `docs/CHECKLIST.md`**: Change `[ ]` to `[x]` for each completed subtask
2. Print final summary:
   ```
   ✅ Task $ARGUMENTS completed!

   Subtasks completed:
   - [x] $ARGUMENTS.1 — [name]
   - [x] $ARGUMENTS.2 — [name]
   ...

   Files created/modified:
   - path/to/file1.ts
   - path/to/file2.ts

   Tests written:
   - path/to/test1.spec.ts (N tests)
   - path/to/test2.test.tsx (N tests)

   Verification:
   - Build: ✅
   - Unit Tests: ✅ (N/N pass)
   - API Tests: ✅ (N endpoints) / ⏭️ N/A
   - Browser E2E: ✅ (N checks) / ⏭️ N/A
   ```

## Rules

- NEVER skip the status check (Step 1)
- NEVER skip writing tests (Step 4) — every task MUST have tests
- NEVER skip verification (Step 5)
- NEVER mark a task done without ALL applicable tests passing
- If a subtask ID is given (e.g., `BE03.2`), only work on that specific subtask
- Follow the project's CLAUDE.md coding rules strictly
- For browser tests: ALWAYS call tabs_context_mcp first, then create a new tab
- For API tests: ALWAYS set_base_url first
- Unit tests are source of truth — if a test fails, determine if the code or the test is wrong
