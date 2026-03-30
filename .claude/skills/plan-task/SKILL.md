---
name: plan-task
description: "Create a detailed implementation plan for a task without coding. Usage: /plan-task FE04"
disable-model-invocation: true
argument-hint: "<task-id> (e.g. BE03, FE04)"
allowed-tools: Read, Glob, Grep
---

# Plan Task: $ARGUMENTS

## Step 1 — Gather Context

1. Read the task document: `docs/tasks/[be|fe]/$ARGUMENTS-*.md`
2. Read the project CLAUDE.md (store or backend)
3. Read relevant skills if applicable
4. Read existing code in the project to understand current state
5. Check dependencies in `docs/CHECKLIST.md` — are they completed?

## Step 2 — Analyze

For each subtask:
1. List exact files to create or modify (full paths)
2. List key functions/components to implement
3. Note imports and dependencies needed
4. Identify edge cases
5. Estimate complexity (simple / medium / complex)

## Step 3 — Output Plan

Print a structured plan:

```
📋 Implementation Plan for $ARGUMENTS — [Task Name]

Dependencies: [list] — Status: ✅ Met / ❌ Not met

Subtask $ARGUMENTS.1 — [name]
  Files: src/path/to/file.ts (CREATE)
  Key: [what to implement]
  Complexity: simple
  Notes: [edge cases, patterns to follow]

Subtask $ARGUMENTS.2 — [name]
  Files: src/path/to/file.ts (MODIFY), src/other.ts (CREATE)
  Key: [what to implement]
  Complexity: medium
  Notes: [edge cases]

...

Suggested order: $ARGUMENTS.1 → $ARGUMENTS.3 → $ARGUMENTS.2 (due to dependency)
Estimated files to create: N
Estimated files to modify: N

Ready to implement? Run: /dev $ARGUMENTS
```

## Rules
- Do NOT write any code — only plan
- Do NOT modify any files
- Be specific about file paths and function signatures
- Identify potential blockers early
