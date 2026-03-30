# FE06 — Cart Page

## Scope
Trang giỏ hàng — quản lý items, edit design, proceed to checkout.

## Reference
- `docs/user-flow.md` → Step 5
- `docs/system-flow.md` → Flow 2

## Tasks

### FE06.1 — Cart State Management
- [ ] Cart ID storage: localStorage
- [ ] `useCart()` hook hoặc Zustand cart store
- [ ] Auto-create cart nếu chưa có (POST /store/carts)
- [ ] Hydrate cart state từ API on page load
- [ ] Cart item count cho Header badge

### FE06.2 — Cart Page (`app/(store)/cart/page.tsx`)
- [ ] Fetch cart data từ Medusa API
- [ ] Layout: items list (left) + order summary (right)
- [ ] Responsive: stack on mobile

### FE06.3 — Cart Item Component
- [ ] Design mockup thumbnail (from design_png_url)
- [ ] Product name + variant info (color, size)
- [ ] Click thumbnail → lightbox full preview
- [ ] Quantity selector (min 1, update via API)
- [ ] Line item price × quantity = subtotal
- [ ] "Chỉnh sửa thiết kế" link
- [ ] Delete button (X icon)
- [ ] Loading state khi updating quantity

### FE06.4 — Edit Design Flow
- [ ] Click "Chỉnh sửa" → fetch design JSON from design_json_url
- [ ] Navigate to `/design/[productId]?edit=true&lineItemId=[id]`
- [ ] Editor loads JSON vào canvas
- [ ] On save: re-export → re-upload → update line item metadata
- [ ] Redirect back to `/cart`

### FE06.5 — Order Summary Sidebar
- [ ] Subtotal (sum of all line items)
- [ ] Shipping: "Tính khi checkout" hoặc estimated
- [ ] Discount code input + "Áp dụng" button (future)
- [ ] Total (bold)
- [ ] "Tiến hành thanh toán" CTA → `/checkout`
- [ ] Disabled state khi cart empty

### FE06.6 — Empty Cart State
- [ ] Illustration/icon
- [ ] "Giỏ hàng trống" message
- [ ] CTA: "Bắt đầu thiết kế" → `/products`

### FE06.7 — Cart API Integration
- [ ] GET /store/carts/{id} — fetch cart
- [ ] POST /store/carts/{id}/line-items/{itemId} — update quantity
- [ ] DELETE /store/carts/{id}/line-items/{itemId} — remove item
- [ ] Optimistic updates cho quantity change
- [ ] Error handling + toast

## Acceptance Criteria
- Cart items render đúng từ API
- Quantity update reflects in UI + API
- Delete item works
- Edit design flow: round-trip (load JSON → edit → re-upload → update)
- Empty state hiển thị khi no items
- Cart badge in header updates

## Dependencies
- FE01 (medusa.ts)
- FE05 (design export, S3 upload)

## Estimated Subtasks: 7
