# FE06 — Cart Page

## Scope
Trang giỏ hàng — quản lý items, edit design, proceed to checkout.

## Reference
- `docs/user-flow.md` → Step 5
- `docs/system-flow.md` → Flow 2

## Tasks

### FE06.1 — Cart State Management
- [x] Cart ID storage: localStorage
- [x] `useCart()` hook hoặc Zustand cart store
- [x] Auto-create cart nếu chưa có (POST /store/carts)
- [x] Hydrate cart state từ API on page load
- [x] Cart item count cho Header badge

### FE06.2 — Cart Page (`app/(store)/cart/page.tsx`)
- [x] Fetch cart data từ Medusa API
- [x] Layout: items list (left) + order summary (right)
- [x] Responsive: stack on mobile

### FE06.3 — Cart Item Component
- [x] Design mockup thumbnail (from design_png_url)
- [x] Product name + variant info (color, size)
- [x] Click thumbnail → lightbox full preview
- [x] Quantity selector (min 1, update via API)
- [x] Line item price × quantity = subtotal
- [x] "Chỉnh sửa thiết kế" link
- [x] Delete button (X icon)
- [x] Loading state khi updating quantity

### FE06.4 — Edit Design Flow
- [x] Click "Chỉnh sửa" → fetch design JSON from design_json_url
- [x] Navigate to `/design/[productId]?edit=true&lineItemId=[id]`
- [x] Editor loads JSON vào canvas
- [x] On save: re-export → re-upload → update line item metadata
- [x] Redirect back to `/cart`

### FE06.5 — Order Summary Sidebar
- [x] Subtotal (sum of all line items)
- [x] Shipping: "Tính khi checkout" hoặc estimated
- [ ] Discount code input + "Áp dụng" button (future)
- [x] Total (bold)
- [x] "Tiến hành thanh toán" CTA → `/checkout`
- [x] Disabled state khi cart empty

### FE06.6 — Empty Cart State
- [x] Illustration/icon
- [x] "Giỏ hàng trống" message
- [x] CTA: "Bắt đầu thiết kế" → `/products`

### FE06.7 — Cart API Integration
- [x] GET /store/carts/{id} — fetch cart
- [x] POST /store/carts/{id}/line-items/{itemId} — update quantity
- [x] DELETE /store/carts/{id}/line-items/{itemId} — remove item
- [x] Optimistic updates cho quantity change
- [x] Error handling + toast

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
