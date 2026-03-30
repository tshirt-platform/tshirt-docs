# FE08 — Order Tracking

## Scope
Trang theo dõi đơn hàng.

## Reference
- `docs/user-flow.md` → Step 8
- `docs/system-flow.md` → Flow 5

## Tasks

### FE08.1 — Order Tracking Page (`app/(store)/orders/[id]/page.tsx`)
- [ ] Fetch order từ Medusa API: GET /store/orders/{id}
- [ ] Layout: order header + progress tracker + details

### FE08.2 — Progress Tracker Component
- [ ] Visual step indicator (4 steps):
  - Đã đặt hàng → Đang in → Đang giao → Đã nhận
- [ ] Active step highlighted, completed steps checked
- [ ] Failed state: red indicator + error message
- [ ] Responsive: horizontal desktop, vertical mobile

### FE08.3 — Order Details
- [ ] Items list: design thumbnail, product info, qty, price
- [ ] Click design thumbnail → view full design image
- [ ] Shipping info: name, phone, address
- [ ] Payment info: method, status
- [ ] Order date, estimated delivery

### FE08.4 — Status Mapping
- [ ] Map PrintJob status → UI display:
  - pending → "Đang xử lý"
  - printing → "Đang in"
  - shipped → "Đang giao hàng" + tracking number
  - delivered → "Đã giao hàng"
  - failed → "Lỗi xử lý" + contact info
- [ ] Fetch status from order metadata hoặc custom endpoint

### FE08.5 — Order Lookup (No Auth)
- [ ] Simple form: nhập Order ID + email/phone
- [ ] Validate → fetch order → show tracking
- [ ] Route: `/orders?lookup=true` hoặc separate component

## Acceptance Criteria
- Order details render đúng từ API
- Progress tracker reflects correct status
- Design thumbnails viewable
- Works without user authentication
- Error state cho invalid order ID

## Dependencies
- FE01 (medusa.ts)
- BE04 (print job status in order)

## Estimated Subtasks: 5
