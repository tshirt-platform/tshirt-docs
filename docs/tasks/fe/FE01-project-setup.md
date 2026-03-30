# FE01 — Project Setup & Foundation

## Scope
Thiết lập cấu trúc dự án, config, shared utilities cho tshirt-store.

## Tasks

### FE01.1 — Folder Structure
- [ ] Tạo folder structure theo REQUIREMENTS.md:
  - `src/app/(store)/` — route group cho store pages
  - `src/app/api/` — API routes
  - `src/components/design-editor/`
  - `src/components/product/`
  - `src/components/checkout/`
  - `src/components/landing/`
  - `src/lib/canvas/`
  - `src/lib/store/`
  - `src/types/`

### FE01.2 — Environment Config (`lib/env.ts`)
- [ ] Setup `@t3-oss/env-nextjs` với Zod schema
- [ ] Define all env vars (NEXT_PUBLIC_MEDUSA_URL, S3, AWS...)
- [ ] Export typed `env` object
- [ ] Verify build fails nếu thiếu required env vars

### FE01.3 — Medusa SDK Client (`lib/medusa.ts`)
- [ ] Init Medusa JS SDK singleton
- [ ] Config `baseUrl` và `publishableKey` từ `env`
- [ ] Export `medusa` instance

### FE01.4 — S3 Helpers (`lib/s3.ts`)
- [ ] Setup S3Client với credentials từ `env`
- [ ] `generatePresignedUrl(fileName, contentType)` — returns `{ presignedUrl, fileUrl }`
- [ ] `getDesignKey(id, side, ext)` — generate S3 key

### FE01.5 — Utility Setup
- [ ] `cn()` helper đã có từ shadcn init (`lib/utils.ts`)
- [ ] Verify Tailwind v4 CSS config working
- [ ] Setup `next/font` cho Google Fonts (Inter default + design fonts)
- [ ] Prettier config (`.prettierrc`) với `prettier-plugin-tailwindcss`

### FE01.6 — Layout & Providers
- [ ] Root layout (`app/layout.tsx`): fonts, metadata, TooltipProvider
- [ ] Store layout (`app/(store)/layout.tsx`): Header, Footer, cart provider
- [ ] Setup Sonner toast provider

### FE01.7 — Shared Types
- [ ] Verify `@tshirt/shared` import works
- [ ] Create local types in `src/types/` nếu cần (component-specific types)

## Acceptance Criteria
- `pnpm dev` chạy không lỗi
- `pnpm build` pass (no TypeScript errors)
- `env.ts` throw error khi thiếu required env vars
- `medusa` client initialized thành công
- Folder structure match REQUIREMENTS.md

## Dependencies
- None (first task)

## Estimated Subtasks: 7
