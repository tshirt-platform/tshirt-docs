# FE07 — Checkout Flow

## Scope
Checkout page — shipping form, payment, order completion.

## Reference
- `docs/user-flow.md` → Step 6
- `docs/system-flow.md` → Flow 4

## Tasks

### FE07.1 — Checkout Page (`app/(store)/checkout/page.tsx`)
- [ ] Layout: 2 columns — form (60%) + order summary (40%)
- [ ] Redirect to `/cart` nếu cart empty
- [ ] Mobile: stack vertically, summary collapsible

### FE07.2 — Shipping Form
- [ ] react-hook-form + Zod validation
- [ ] Fields:
  - Họ và tên (required, min 2)
  - Số điện thoại (required, VN format: 10 digits, starts 0)
  - Email (required, valid email)
  - Tỉnh/Thành phố (select, 63 options)
  - Quận/Huyện (select, depends on Tỉnh)
  - Phường/Xã (select, depends on Quận)
  - Địa chỉ chi tiết (textarea, required, min 5)
  - Ghi chú (textarea, optional)
- [ ] Field-level validation on blur
- [ ] Province → District → Ward cascading selects
- [ ] Data source: static JSON hoặc API cho địa chỉ VN

### FE07.3 — Payment Method Selection
- [ ] Radio group: COD / VNPay
- [ ] COD: default selected, icon + description
- [ ] VNPay: logo + "Chuyển đến VNPay để thanh toán"
- [ ] Payment method affects submission flow

### FE07.4 — Order Summary (Checkout)
- [ ] List cart items (compact: thumbnail, name, qty, price)
- [ ] Subtotal
- [ ] Shipping fee (calculated from address, or fixed tiers)
- [ ] Discount code input (future, placeholder)
- [ ] Total
- [ ] "Đặt hàng" CTA button

### FE07.5 — Checkout Submission (COD)
- [ ] On submit:
  1. Validate form (Zod)
  2. Update cart shipping address (POST /store/carts/{id})
  3. Set shipping method (POST /store/carts/{id}/shipping-methods)
  4. Create payment session (manual/COD)
  5. Complete cart (POST /store/carts/{id}/complete)
  6. Navigate to `/checkout/success?orderId=[id]`
- [ ] Loading overlay: "Đang xử lý đơn hàng..."
- [ ] Error handling: toast + enable retry

### FE07.6 — Checkout Submission (VNPay) — Future
- [ ] Create VNPay payment session
- [ ] Redirect to VNPay gateway
- [ ] Handle callback: verify payment → complete cart
- [ ] Error: payment failed → show retry

### FE07.7 — Success Page (`app/(store)/checkout/success/page.tsx`)
- [ ] Success animation (checkmark, motion)
- [ ] "Đặt hàng thành công!" heading
- [ ] Order ID (copyable)
- [ ] Order summary: items, total, shipping address
- [ ] Estimated delivery: "3-5 ngày làm việc"
- [ ] Actions: "Theo dõi đơn hàng", "Tiếp tục mua sắm", "Thiết kế thêm"

### FE07.8 — Zod Schemas
- [ ] `checkoutFormSchema` — all shipping fields
- [ ] `phoneSchema` — VN phone validation
- [ ] `addressSchema` — nested province/district/ward

## Acceptance Criteria
- Form validation works (field-level + submit)
- Cascading address selects work
- COD checkout: cart → complete → success page
- Success page shows correct order info
- Error states handled gracefully
- Mobile responsive

## Dependencies
- FE01 (medusa.ts, env.ts)
- FE06 (cart state)

## Estimated Subtasks: 8
