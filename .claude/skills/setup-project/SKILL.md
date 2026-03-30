# Project Setup — New Machine

Automate full project setup on a fresh machine.

## Step 1 — GitHub Authentication

Check if GitHub credentials are available:

```bash
# Check for GITHUB_TOKEN env var
echo $GITHUB_TOKEN

# Or check git credential manager
git credential-manager github list 2>&1
```

### If NO credentials found:

Ask the user:
```
⚠️ GitHub credentials not found. Choose an option:
1. Provide a GitHub Personal Access Token (set GITHUB_TOKEN env var)
2. Let me open the browser to authenticate via GitHub

Which do you prefer? (1 or 2)
```

**Option 1** — User provides token:
- Save it for this session: `export GITHUB_TOKEN=<token>`

**Option 2** — Browser auth:
- Use `mcp__claude-in-chrome` to navigate to `https://github.com/settings/tokens/new`
- Guide user through creating a PAT with `repo` scope
- User copies the token manually
- IMPORTANT: Never enter or handle the token value directly — ask user to set it

### If credentials found:
```
✅ GitHub credentials available. Proceeding with clone.
```

## Step 2 — Clone Repositories

Determine the workspace directory (default: current working directory).

```bash
WORKSPACE=$(pwd)
ORG="tshirt-platform"
REPOS=("tshirt-docs" "tshirt-backend" "tshirt-shared" "tshirt-store")
```

For each repo, check if it already exists:

```bash
for repo in "${REPOS[@]}"; do
  if [ -d "$WORKSPACE/$repo" ] || [ -d "$WORKSPACE/$repo/.git" ]; then
    echo "✅ $repo already exists — pulling latest..."
    cd "$WORKSPACE/$repo" && git pull
  else
    echo "📦 Cloning $repo..."
    git clone "https://github.com/$ORG/$repo.git" "$WORKSPACE/$repo"
  fi
done
```

**Mapping local folders:**
- `tshirt-docs` → root files (`.claude/`, `docs/`, `CLAUDE.md`, etc.)
- `tshirt-backend` → `tshirt-backend/`
- `tshirt-shared` → `tshirt-shared/`
- `tshirt-store` → `tshirt-store/`

**Special handling for tshirt-docs:**
The `tshirt-docs` repo contains root-level files. Clone it first, then move contents:
```bash
# Clone tshirt-docs to temp dir, then copy root files
git clone "https://github.com/$ORG/tshirt-docs.git" "$WORKSPACE/_tshirt-docs-temp"
# Copy all files (including hidden) except .git to workspace root
cp -a "$WORKSPACE/_tshirt-docs-temp/." "$WORKSPACE/"
rm -rf "$WORKSPACE/_tshirt-docs-temp"
# Re-init root git pointing to tshirt-docs
cd "$WORKSPACE" && git init && git remote add origin "https://github.com/$ORG/tshirt-docs.git"
```

After cloning, verify:
```
📦 Clone Summary:
- tshirt-docs    → ✅ / ❌
- tshirt-backend → ✅ / ❌
- tshirt-shared  → ✅ / ❌
- tshirt-store   → ✅ / ❌
```

## Step 3 — Docker Services (PostgreSQL + Redis)

Check if Docker is available:
```bash
docker --version 2>&1
docker compose version 2>&1
```

### If Docker NOT available:
```
⚠️ Docker is not installed. Please install Docker Desktop:
https://www.docker.com/products/docker-desktop/
Then re-run /setup-project
```
STOP here.

### If Docker available:

Check if containers are already running:
```bash
docker ps --format '{{.Names}}' | grep -E 'tshirt_postgres|tshirt_redis'
```

If NOT running, start them:
```bash
cd "$WORKSPACE" && docker compose up -d
```

Wait and verify:
```bash
# Wait for PostgreSQL to be ready
sleep 5
docker exec tshirt_postgres pg_isready -U postgres
docker exec tshirt_redis redis-cli ping
```

Print status:
```
🐳 Docker Services:
- PostgreSQL (tshirt_postgres) → ✅ Ready / ❌ Failed
- Redis (tshirt_redis)         → ✅ Ready / ❌ Failed
```

## Step 4 — Install Dependencies

```bash
# Shared types (build first — other packages depend on it)
cd "$WORKSPACE/tshirt-shared" && pnpm install && pnpm build

# Backend
cd "$WORKSPACE/tshirt-backend" && pnpm install

# Frontend
cd "$WORKSPACE/tshirt-store" && pnpm install
```

Print status:
```
📦 Dependencies:
- tshirt-shared  → ✅ installed + built
- tshirt-backend → ✅ installed
- tshirt-store   → ✅ installed
```

## Step 5 — Database Setup

Check if database has been migrated:
```bash
docker exec tshirt_postgres psql -U postgres -d tshirt_db -c "SELECT COUNT(*) FROM product" 2>&1
```

### If DB is empty or migration needed:
```bash
cd "$WORKSPACE/tshirt-backend" && npx medusa db:migrate
```

### Check if data is seeded:
```bash
docker exec tshirt_postgres psql -U postgres -d tshirt_db -c "SELECT COUNT(*) FROM product" 2>&1
```

If count is 0:
```bash
cd "$WORKSPACE/tshirt-backend" && pnpm seed
```

Print status:
```
🗄️ Database:
- Migrations → ✅ Applied
- Seed data  → ✅ 4 products, Vietnam region, 3 shipping options
```

## Step 6 — Environment Files

Check if `.env` files exist, create from templates if missing:

**tshirt-backend/.env:**
```env
DATABASE_URL=postgres://postgres:postgres@localhost:5432/tshirt_db
REDIS_URL=redis://localhost:6379
STORE_CORS=http://localhost:3000
ADMIN_CORS=http://localhost:9000
AUTH_CORS=http://localhost:3000,http://localhost:9000
JWT_SECRET=supersecret
COOKIE_SECRET=supersecret
```

**tshirt-store/.env.local:**
```env
NEXT_PUBLIC_MEDUSA_URL=http://localhost:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=<get from DB>
NEXT_PUBLIC_STORE_URL=http://localhost:3000
```

For the publishable key, query the database:
```bash
docker exec tshirt_postgres psql -U postgres -d tshirt_db -t -c \
  "SELECT token FROM api_key WHERE type = 'publishable' LIMIT 1"
```

## Step 7 — Verify

Start services and verify everything works:

```bash
# Start backend (background)
cd "$WORKSPACE/tshirt-backend" && npx medusa develop &

# Wait for backend to start
sleep 15
curl -s -o /dev/null -w "%{http_code}" http://localhost:9000/health
```

If backend is healthy (200):
```bash
# Start frontend (background)
cd "$WORKSPACE/tshirt-store" && pnpm dev &

# Wait for frontend
sleep 10
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000
```

Print final summary:
```
🚀 Project Setup Complete!

Services:
- PostgreSQL    → ✅ localhost:5432
- Redis         → ✅ localhost:6379
- Medusa Backend → ✅ localhost:9000
- Next.js Store  → ✅ localhost:3000
- Medusa Admin   → ✅ localhost:9000/app

Admin credentials:
- Email: admin@tshirt.vn
- Password: admin123456

Next steps:
- Backend dev:  cd tshirt-backend && pnpm dev
- Frontend dev: cd tshirt-store && pnpm dev
- Shared types: cd tshirt-shared && pnpm dev
```

## Error Handling

If any step fails:
1. Print the error clearly
2. Suggest fix
3. Ask user if they want to retry or skip
4. Do NOT proceed to later steps if a critical step fails (clone, docker, migrations)

## Rules
- NEVER handle or store GitHub tokens directly — let user input them
- NEVER commit `.env` files
- Always check if a step was already done before re-doing it (idempotent)
- Use `docker compose` (v2 syntax), not `docker-compose` (v1)
