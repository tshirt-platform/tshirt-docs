# BE06 — Testing & Deployment Prep

## Scope
End-to-end testing, environment hardening, deployment preparation.

## Tasks

### BE06.1 — API Testing
- [ ] Test Store API endpoints:
  - GET /store/products (list)
  - GET /store/products/{id} (detail)
  - POST /store/carts (create)
  - POST /store/carts/{id}/line-items (add item with design metadata)
  - POST /store/carts/{id}/complete (checkout)
- [ ] Test custom endpoints:
  - POST /webhooks/print-shop (status update)
  - GET /admin/print-orders (list)
  - POST /admin/print-orders/{id}/retry

### BE06.2 — Workflow Testing
- [ ] Test send-to-print-shop workflow end-to-end
- [ ] Mock print shop API for testing
- [ ] Test compensation on failure
- [ ] Test subscriber triggers correctly

### BE06.3 — Environment Hardening
- [ ] Strong JWT_SECRET and COOKIE_SECRET
- [ ] Validate all env vars on startup
- [ ] CORS: restrict to production domains
- [ ] Rate limiting on webhook endpoints

### BE06.4 — Database
- [ ] Verify all migrations up-to-date
- [ ] Backup strategy for production
- [ ] Index on print_jobs.order_id, print_jobs.status

### BE06.5 — Docker Setup (Optional)
- [ ] Dockerfile for Medusa backend
- [ ] docker-compose.yml: backend + postgres + redis
- [ ] Health check endpoints

### BE06.6 — Documentation
- [ ] API documentation (routes, payloads, responses)
- [ ] Environment variables reference
- [ ] Deployment checklist

## Acceptance Criteria
- All API endpoints tested and working
- Workflow tested with mock print shop
- Security measures in place
- Database properly indexed
- Deployable configuration

## Dependencies
- BE01–BE05 (all backend tasks)

## Estimated Subtasks: 6
