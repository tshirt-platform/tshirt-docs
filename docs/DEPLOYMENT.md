# Deployment

Four pieces: **tshirt-store** (Next.js), **tshirt-backend** (Medusa), **tshirt-render** (FastAPI preview
service, internal), and their datastores: Postgres, Redis and an S3-compatible bucket (Cloudflare R2 or AWS S3).

## Environment variables

### tshirt-backend

The server validates these at startup (`src/lib/env-check.ts`). In production a problem stops it from
starting; in development it prints a warning.

| Variable | Required | Notes |
|---|---|---|
| `DATABASE_URL` | yes | Postgres connection string. In production Medusa connects with SSL: a managed database works as it is; for one without SSL (such as the compose container) append `?sslmode=disable` |
| `REDIS_URL` | production | events and caching |
| `VISION_API_KEY`, `VISION_MODEL` (render service) | no | switch on AI analysis of mockup photos. Both are needed; no model name is assumed. Put them in `tshirt-render/.env` (git-ignored) or the environment. Each analysed photo is sent to the provider once, so only use photos you may share. `VISION_API_URL` points at another OpenAI-compatible endpoint |
| `ADMIN_SESSION_TTL_HOURS` | no | admin login lifetime in hours, default 10. Sliding once set. Sessions are kept in Redis, so they survive a backend restart. Development may use up to 720 (e.g. 336); production refuses more than 24 |
| `JWT_SECRET`, `COOKIE_SECRET` | production | at least 32 characters, not a placeholder. Generate with `openssl rand -hex 32` |
| `STORE_CORS`, `ADMIN_CORS`, `AUTH_CORS` | yes | comma-separated origins. Production refuses `*` and `localhost` |
| `S3_BUCKET_NAME` | production | without it, design files go to `./static`, which a deploy wipes |
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` | with a bucket | R2 API token scoped to the bucket, Object Read & Write |
| `AWS_REGION` | with a bucket | `auto` for R2 |
| `S3_ENDPOINT` | R2 only | `https://<account-id>.r2.cloudflarestorage.com` |
| `S3_PUBLIC_URL` | with `S3_ENDPOINT` | where files are served from: r2.dev URL or custom domain, no trailing slash |
| `DESIGN_FILE_ORIGINS` | production | comma-separated origins the print package may download design files from. Include `S3_PUBLIC_URL`'s origin |
| `BACKEND_PUBLIC_URL` | development | builds file URLs when files are on local disk |
| `RENDER_SERVICE_URL`, `RENDER_API_KEY` | preview | address and key of the render service |

### tshirt-store

| Variable | Notes |
|---|---|
| `NEXT_PUBLIC_MEDUSA_URL` | backend origin |
| `NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY` | from the Medusa admin (Settings > Publishable API keys) |
| `NEXT_PUBLIC_STORE_URL` | the store's own public origin: canonical links, sitemap, Open Graph |
| `RENDER_SERVICE_URL`, `RENDER_API_KEY` | server-side only, for `/api/preview` |

The store holds no storage credentials: uploads go to the backend.

### tshirt-render

`RENDER_API_KEY` (the service checks `X-Api-Key` on everything except `/health`). Bind it to a private
network or `127.0.0.1`; only the store's server and the backend call it.

## Building the backend image

`@tshirt-platform/shared` is a private GitHub Packages package, so the build needs a token that can read
packages. It is passed as a build secret and never stored in a layer.

```bash
GITHUB_TOKEN=$(gh auth token) docker compose --profile backend build backend
docker compose --profile backend up -d backend      # needs tshirt-backend/.env.production
```

The container runs `medusa db:migrate` and then `medusa start`, so a new image brings its own schema.
Its health check is `GET /health`.

The store's build needs the same token in `.npmrc` (`read:packages`).

## Checklist before going live

- [ ] `JWT_SECRET` and `COOKIE_SECRET` generated (32+ characters), different per environment
- [ ] CORS lists only the real store and admin origins
- [ ] Bucket created; API token limited to that bucket; public access on (r2.dev for testing, a custom domain for production)
- [ ] `DESIGN_FILE_ORIGINS` contains the bucket's public origin
- [ ] `RENDER_API_KEY` set on both the render service and its callers; the service is not reachable from the internet
- [ ] Backend behind exactly one reverse proxy that sets `X-Forwarded-For` (Medusa trusts one hop). With none, or two, the rate limiters see the wrong address
- [ ] `medusa db:migrate` run; `pnpm seed` only on a new empty database
- [ ] Admin user created (`pnpm medusa user -e <email> -p <password>`) with a strong password
- [ ] Publishable API key created in the admin and set in the store's environment
- [ ] Shipping options and prices exist for the region (the seed creates three flat rates)
- [ ] Products carry `print_config` (`pnpm exec medusa exec ./src/scripts/backfill-print-config.ts` fills it for existing ones)
- [ ] Preview templates uploaded in the admin (Product > Print configuration), otherwise previews use the flat rendering
- [ ] `pnpm smoke` passes against staging
- [ ] A backup has been taken and restored once (see below)

## Backups

**Database.** `tshirt-backend/scripts/backup-db.sh [dir]` writes a compressed `pg_dump` and checks it can be
read back with `pg_restore --list`. Run it nightly and copy the file off the database's disk, for example
into a second R2 bucket. Keep about 14 daily and 8 weekly copies. If Postgres is managed (Neon, RDS,
Supabase), turn on its point-in-time recovery as well. Restore into a scratch database first:

```bash
createdb tshirt_restore && pg_restore --no-owner -d tshirt_restore backups/tshirt_db-<stamp>.dump
```

**Design files.** They are the customer's artwork and the source of every print. Turn on R2 object
versioning or a lifecycle rule that keeps deleted objects for 30 days. Designs that never became an
order are not cleaned up yet: a later job can delete `designs/*` older than N days whose id appears in
no order.

**Preview templates.** The render service keeps them on its `/data` volume (`tshirt_renderdata`); back that
volume up too, or re-upload the photos.

## Scaling notes

- The rate limiters (design upload, order lookup) count in memory per backend instance. With several
  instances, each keeps its own count; move them to Redis before scaling out.
- Medusa runs subscribers in the same process, so a second backend instance also handles events: keep
  Redis (`REDIS_URL`) as the event bus in production.
