---
name: dev-subtask
description: "Execute a single subtask with tests. Usage: /dev-subtask BE03.2"
disable-model-invocation: true
argument-hint: "<task-id.subtask> (e.g. BE03.2, FE04.6)"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent, mcp__api-tester__*, mcp__claude-in-chrome__*
---

# Execute Single Subtask: $ARGUMENTS

Parse the task ID: split by `.` to get parent task and subtask number.
Example: `BE03.2` → parent=`BE03`, subtask=`2`

## Step 1 — Load & Check

1. Read `docs/CHECKLIST.md` — find the specific subtask
2. If already `[x]` → print "✅ Subtask $ARGUMENTS already completed" and stop
3. Read the parent task document for full subtask details
4. Read `docs/tasks/test-strategies.md` — find tests for this subtask
5. Read relevant CLAUDE.md and skills

## Step 2 — Implement

1. Print: `📝 Working on $ARGUMENTS — [subtask name]`
2. Implement only this specific subtask
3. Follow all coding rules from CLAUDE.md

## Step 3 — Write Tests

From `test-strategies.md`, check if this subtask needs unit tests:
- If yes → write the relevant test cases
- Test file goes next to source: `__tests__/[name].spec.ts` or `[name].test.ts`

## Step 4 — Verify

Run the applicable checks for this subtask:

1. **Build check** (always): `pnpm build` or `pnpm tsc --noEmit`
2. **Unit tests** (if written): `pnpm test -- [pattern]`
3. **DB check** (if DB subtask): verify table/columns via psql
4. **API test** (if API subtask): use `mcp__api-tester` to hit endpoint
5. **Browser check** (if UI subtask): use `mcp__claude-in-chrome` to verify visually

Fix any failures (max 3 attempts per issue).

## Step 5 — Update

1. Mark subtask as `[x]` in `docs/CHECKLIST.md`
2. Print completion summary:
   ```
   ✅ $ARGUMENTS completed!
   Files: [list]
   Tests: [list with pass/fail]
   ```
