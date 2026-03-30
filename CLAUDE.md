# Custom T-Shirt Platform — Monorepo Root

## Architecture
Three packages, deployed independently:

| Package | Tech | Port | Purpose |
|---|---|---|---|
| `tshirt-store/` | Next.js 16 (App Router) | :3000 | Customer-facing storefront |
| `tshirt-backend/` | Medusa.js v2 | :9000 | E-commerce backend + admin |
| `tshirt-shared/` | TypeScript + tsup | — | Shared types (`@tshirt/shared`) |

## Shared Coding Rules
1. TypeScript strict — no `any`, no `@ts-ignore`
2. Comments in English only
3. Max 200 lines per file — extract if exceeding
4. Absolute imports where supported

## Cross-Package Dependencies
- `tshirt-store` and `tshirt-backend` both depend on `@tshirt/shared`
- `tshirt-store` communicates with `tshirt-backend` via Medusa JS SDK
- Design files stored on AWS S3 (presigned URLs)

## Package-Specific Instructions
- See `tshirt-store/CLAUDE.md` for frontend patterns
- See `tshirt-backend/CLAUDE.md` for backend patterns

## Development Workflow Commands

| Command | Usage | Description |
|---|---|---|
| `/setup-project` | `/setup-project` | Clone repos, start Docker, install deps, migrate, seed |
| `/dev` | `/dev BE01` | Full workflow: check → plan → code → test → fix → update checklist |
| `/dev-subtask` | `/dev-subtask BE03.2` | Execute a single subtask only |
| `/plan-task` | `/plan-task FE04` | Plan implementation without coding |
| `/test-task` | `/test-task BE03` | Run tests and verify acceptance criteria |
| `/check-status` | `/check-status` or `/check-status BE03` | View task completion status |

### Workflow: `/dev <task-id>`
```
/dev BE01
  → Read CHECKLIST.md → Check if done
  → Done? → Notify and stop
  → Not done? → git checkout -b feat/BE01
  → Plan → Code (commit per subtask) → Write Tests → Verify → Fix loop
  → Update checklist → git push → gh pr create → Print PR URL
```

### Git Flow Rules
- Each task works on branch `feat/<task-id>` (e.g., `feat/BE01`)
- Commit after each subtask — no `Co-Authored-By` in messages
- After all tests pass → push branch → create PR to `master` via `gh`

### Testing: `/test-task <task-id>` — Smart Auto-Verification
Test categories are auto-selected based on task type:

| Task Type | Build | Unit Test | DB Check | API Test | Browser E2E |
|---|---|---|---|---|---|
| BE setup (BE01) | ✅ | — | ✅ psql | ✅ api-tester | — |
| BE data (BE02) | ✅ | ✅ | — | ✅ api-tester | — |
| BE module (BE03) | ✅ | ✅ | ✅ psql | — | — |
| BE workflow (BE04) | ✅ | ✅ | — | ✅ api-tester | — |
| BE routes (BE05) | ✅ | ✅ | — | ✅ api-tester | — |
| FE setup (FE01) | ✅ | ✅ | — | — | ✅ chrome |
| FE pages (FE02-08) | ✅ | ✅ | — | — | ✅ chrome |

Tools used:
- **DB Check**: `Bash` (psql) — verify tables, columns, data
- **API Test**: `mcp__api-tester` — HTTP requests + response validation
- **Browser E2E**: `mcp__claude-in-chrome` — navigate, screenshot, click, verify

## Available Skills (Reference)
- `design-editor` — Fabric.js v7 canvas patterns
- `medusa-v2` — Medusa.js v2 module/workflow/subscriber patterns
- `print-shop` — Print shop integration and webhook patterns

## Task Tracking
- Master checklist: `docs/CHECKLIST.md`
- Test strategies: `docs/tasks/test-strategies.md`
- FE task docs: `docs/tasks/fe/FE*.md`
- BE task docs: `docs/tasks/be/BE*.md`

## Running Locally
```bash
# Terminal 1: Backend
cd tshirt-backend && pnpm dev     # :9000

# Terminal 2: Frontend
cd tshirt-store && pnpm dev       # :3000

# Terminal 3: Shared types (watch mode)
cd tshirt-shared && pnpm dev
```
