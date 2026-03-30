# Project Init — CLI Commands

## Prerequisites

```bash
node --version   # required: v20+
pnpm --version   # required: v9+

# Install pnpm if not available
npm install -g pnpm
```

---

## 1. tshirt-store (Next.js)

### Step 1 — Init Next.js project

```bash
npx create-next-app@latest tshirt-store \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --turbopack
```

> Chọn khi được hỏi:
> - Would you like to use Tailwind CSS? → **Yes**
> - Would you like to use App Router? → **Yes**
> - Would you like to use Turbopack? → **Yes**

```bash
cd tshirt-store
```

---

### Step 2 — Init shadcn/ui

```bash
npx shadcn@latest init
```

> Chọn khi được hỏi:
> - Which style would you like to use? → **Default**
> - Which color would you like to use as base color? → **Neutral**
> - Would you like to use CSS variables for colors? → **Yes**

---

### Step 3 — Add shadcn components

```bash
npx shadcn@latest add button input label textarea select \
  card dialog drawer sheet toast sonner \
  badge separator scroll-area \
  form slider toggle tooltip \
  dropdown-menu context-menu
```

---

### Step 4 — Install core dependencies

```bash
# State management
pnpm add zustand

# Canvas editor
pnpm add fabric

# Animation (formerly framer-motion)
pnpm add motion

# Forms & validation
pnpm add react-hook-form zod @hookform/resolvers

# Medusa SDK
pnpm add @medusajs/js-sdk

# AWS S3
pnpm add @aws-sdk/client-s3 @aws-sdk/s3-request-presigner

# Utilities
pnpm add clsx tailwind-merge lucide-react
pnpm add @t3-oss/env-nextjs     # typed env vars
```

---

### Step 5 — Install dev dependencies

```bash
pnpm add -D \
  @types/node \
  prettier \
  prettier-plugin-tailwindcss \
  eslint-plugin-import
```

---

### Step 6 — Verify package.json

```json
{
  "dependencies": {
    "@aws-sdk/client-s3": "^3.x",
    "@aws-sdk/s3-request-presigner": "^3.x",
    "@hookform/resolvers": "^5.2.2",
    "@medusajs/js-sdk": "^2.x",
    "@t3-oss/env-nextjs": "^0.x",
    "fabric": "^7.2.0",
    "lucide-react": "^0.x",
    "motion": "^12.x",
    "next": "^16.2.0",
    "react": "^19.2.0",
    "react-dom": "^19.2.0",
    "react-hook-form": "^7.71.1",
    "zod": "^4.3.5",
    "zustand": "^5.0.12"
  }
}
```

---

### Step 7 — Setup .env.local

```bash
cp .env.example .env.local 2>/dev/null || touch .env.local
```

Thêm vào `.env.local`:
```bash
NEXT_PUBLIC_MEDUSA_URL=http://localhost:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=
NEXT_PUBLIC_S3_BUCKET_URL=
NEXT_PUBLIC_STORE_URL=http://localhost:3000

AWS_REGION=ap-southeast-1
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
S3_BUCKET_NAME=
S3_DESIGNS_PREFIX=designs/
```

---

### Step 8 — Run dev server

```bash
pnpm dev
# → http://localhost:3000
```

---

## 2. tshirt-backend (Medusa.js)

### Step 1 — Init Medusa project

```bash
npx create-medusa-app@latest tshirt-backend \
  --no-nextjs-starter \
  --db-url "postgres://postgres:password@localhost:5432/tshirt_db"
```

> Chọn khi được hỏi:
> - Would you like to create the database? → **Yes**
> - Would you like to run migrations? → **Yes**
> - Would you like to create an admin user? → **Yes**

```bash
cd tshirt-backend
```

---

### Step 2 — Install additional dependencies

```bash
# AWS S3 (for storing design file URLs)
pnpm add @aws-sdk/client-s3

# Axios for calling print shop API
pnpm add axios

# Utilities
pnpm add zod
```

---

### Step 3 — Setup .env

```bash
cp .env .env.local 2>/dev/null
```

Thêm vào `.env`:
```bash
DATABASE_URL=postgres://postgres:password@localhost:5432/tshirt_db
REDIS_URL=redis://localhost:6379

JWT_SECRET=your-jwt-secret-change-in-production
COOKIE_SECRET=your-cookie-secret-change-in-production

STORE_CORS=http://localhost:3000
ADMIN_CORS=http://localhost:9000
AUTH_CORS=http://localhost:3000,http://localhost:9000

AWS_REGION=ap-southeast-1
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
S3_BUCKET_NAME=

# Print shop integration
PRINT_SHOP_API_URL=https://printshop.example.com/api
PRINT_SHOP_API_KEY=
```

---

### Step 4 — Run migrations & seed

```bash
pnpm medusa db:migrate
pnpm medusa seed --seed-file="./data/seed.json"  # optional
```

---

### Step 5 — Run dev server

```bash
pnpm dev
# → http://localhost:9000
# Admin → http://localhost:9000/app
```

---

## 3. tshirt-shared (Shared types package)

### Step 1 — Init package

```bash
mkdir tshirt-shared && cd tshirt-shared
pnpm init
```

---

### Step 2 — Install dependencies

```bash
pnpm add -D typescript tsup @types/node
```

---

### Step 3 — Init TypeScript

```bash
npx tsc --init
```

Cập nhật `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "declaration": true,
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

### Step 4 — Setup tsup config

```bash
touch tsup.config.ts
```

```ts
// tsup.config.ts
import { defineConfig } from "tsup"

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["cjs", "esm"],
  dts: true,
  clean: true,
})
```

---

### Step 5 — Update package.json

```json
{
  "name": "@tshirt/shared",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  },
  "scripts": {
    "build": "tsup",
    "dev": "tsup --watch"
  }
}
```

---

### Step 6 — Tạo folder structure

```bash
mkdir -p src/types src/constants
touch src/index.ts src/types/design.types.ts src/types/print-order.types.ts src/constants/order.constants.ts
```

---

### Step 7 — Build & link local

```bash
pnpm build

# Link vào store và backend (dev local)
cd ../tshirt-store && pnpm add ../tshirt-shared
cd ../tshirt-backend && pnpm add ../tshirt-shared
```

---

## Final Check — Tất cả services chạy

```bash
# Terminal 1
cd tshirt-backend && pnpm dev     # :9000

# Terminal 2
cd tshirt-store && pnpm dev       # :3000

# Terminal 3 (optional — watch shared types)
cd tshirt-shared && pnpm dev
```

---

## Folder sau khi init

```
~/projects/
├── tshirt-store/       # Next.js 16 — localhost:3000
├── tshirt-backend/     # Medusa 2.x — localhost:9000
└── tshirt-shared/      # @tshirt/shared types package
```
