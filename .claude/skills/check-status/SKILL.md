---
name: check-status
description: "Check the completion status of tasks from the checklist. Usage: /check-status or /check-status BE03"
disable-model-invocation: true
argument-hint: "[task-id] (optional, e.g. BE01, FE04, or blank for all)"
allowed-tools: Read, Glob, Grep
---

# Check Task Status

Read `docs/CHECKLIST.md` and report the status.

## If no argument provided ($ARGUMENTS is empty):

Print a summary of ALL tasks:

```
📊 Project Status Overview

Phase 1 — Foundation
  BE01 Backend Setup:     ▓▓▓▓░░ 4/6 (67%)
  FE01 Frontend Setup:    ▓▓▓▓▓▓▓ 7/7 (100%) ✅

Phase 2 — Backend Core
  BE02 Product Data:      ░░░░░░░ 0/7 (0%)
  ...

Overall: 15/102 subtasks (15%)
Progress: ▓▓░░░░░░░░ 15%
```

Use `▓` for completed and `░` for remaining. Show percentage per task.

## If task ID provided ($ARGUMENTS has value):

1. Find the task in the checklist
2. Print detailed status:

```
📋 Task $ARGUMENTS — [Task Name]

Status: ⏳ In Progress (3/6 subtasks done)

Subtasks:
  [x] $ARGUMENTS.1 — [name]
  [x] $ARGUMENTS.2 — [name]
  [x] $ARGUMENTS.3 — [name]
  [ ] $ARGUMENTS.4 — [name]     ← Next
  [ ] $ARGUMENTS.5 — [name]
  [ ] $ARGUMENTS.6 — [name]

Dependencies: BE01 ✅, BE02 ✅
Next action: /dev $ARGUMENTS to continue
```

## Rules
- Only READ files, never modify
- Count `[x]` as done, `[ ]` as not done
- Show which subtask should be worked on next
