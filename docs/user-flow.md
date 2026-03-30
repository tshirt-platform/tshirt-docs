# User Flow — Chi tiết từng bước

## Flow Overview

```
Landing Page → Chọn sản phẩm → Design Editor → Preview → Cart → Checkout → Order Success → Tracking
```

---

## Step 1 — Landing Page (`/`)

### Mục đích
Thu hút khách hàng, giới thiệu dịch vụ, dẫn vào luồng thiết kế.

### Layout & Sections

#### 1.1 Hero Section
- **Background**: Ảnh mockup áo custom full-width, có overlay gradient
- **Headline**: "Thiết kế áo của riêng bạn"
- **Subheadline**: "Upload ảnh, thêm text, chọn template — nhận áo trong 3-5 ngày"
- **CTA button**: "Thiết kế ngay" → navigate to `/products`
- **Secondary CTA**: "Xem mẫu có sẵn" → scroll to Template Gallery

#### 1.2 How It Works
- 3 bước với icon + mô tả ngắn:
  1. **Chọn áo** — Chọn loại áo, màu, size
  2. **Thiết kế** — Dùng editor kéo thả để tạo design
  3. **Đặt hàng** — Checkout và nhận áo tận nhà
- Animation: stagger fade-in khi scroll vào viewport (motion)

#### 1.3 Template Gallery
- Grid 3-4 cột hiển thị 6-8 template hot nhất
- Mỗi card: ảnh mockup + tên template + tag (Sport, Minimal, Vintage...)
- Click template → navigate to `/design/[productId]?template=[templateId]`
- "Xem tất cả" button → navigate to `/products`

#### 1.4 Social Proof
- Counter stats: "10,000+ áo đã in", "4.9/5 đánh giá", "2,000+ thiết kế"
- Grid 3 cột: ảnh khách hàng thực tế mặc áo custom
- Testimonial carousel (tên, avatar, quote)

#### 1.5 Pricing
- 3 cards: T-shirt / Polo / Hoodie
- Mỗi card: giá cơ bản, mô tả chất liệu, nút "Thiết kế"
- Highlight card phổ biến nhất (T-shirt) với badge "Phổ biến"

#### 1.6 FAQ
- Accordion (shadcn Collapsible hoặc custom)
- Câu hỏi thường gặp:
  - Thời gian giao hàng?
  - Chất liệu áo?
  - Có thể in 2 mặt không?
  - Kích thước file thiết kế?
  - Chính sách đổi trả?

#### 1.7 Footer
- Logo + tagline
- Links: Về chúng tôi, Liên hệ, Chính sách, FAQ
- Social icons
- Copyright

### Interactions
| Action | Result |
|---|---|
| Click "Thiết kế ngay" | Navigate to `/products` |
| Click template card | Navigate to `/design/[productId]?template=[templateId]` |
| Click pricing card "Thiết kế" | Navigate to `/products?type=[shirtType]` |
| Scroll | Section animations trigger on viewport entry |

---

## Step 2 — Chọn sản phẩm (`/products` & `/products/[id]`)

### 2A — Product Listing (`/products`)

#### Layout
- **Header**: Breadcrumb (Trang chủ > Sản phẩm)
- **Filter bar**: Lọc theo loại áo (Tất cả / T-shirt / Polo / Hoodie)
- **Product grid**: 3 cột desktop, 2 cột tablet, 1 cột mobile
- **Mỗi product card**:
  - Ảnh mockup áo (hover → đổi sang ảnh mặt sau)
  - Tên sản phẩm
  - Giá
  - Color swatches (3-5 màu phổ biến)
  - Badge: "Mới", "Bán chạy" (nếu có)

#### Data Source
- `GET /store/products` via Medusa JS SDK
- Filter by `collection` hoặc `tag` cho loại áo
- Pagination: infinite scroll hoặc "Xem thêm" button

#### Interactions
| Action | Result |
|---|---|
| Click filter chip | Filter products by type, re-fetch |
| Hover product card | Show back image |
| Click product card | Navigate to `/products/[id]` |
| Click color swatch | Update card mockup image |

### 2B — Product Detail (`/products/[id]`)

#### Layout
- **Left**: Product image gallery (main image + thumbnails, front/back views)
- **Right**: Product info panel
  - Tên sản phẩm
  - Giá (format: xxx.000₫)
  - **Chọn màu áo**: Color picker grid (6-10 màu)
  - **Chọn size**: Button group (S / M / L / XL / XXL) + link "Bảng size"
  - **Số lượng**: Number input (min 1, max 100)
  - **CTA**: "Bắt đầu thiết kế" button (primary, full-width)
  - Thông tin thêm: Chất liệu, cách giặt, thời gian giao
- **Below**: Size chart modal/drawer, product description

#### Size Chart
- Trigger: Click "Bảng size" link
- UI: Sheet/Drawer from right (shadcn Sheet)
- Content: Table với các số đo (Vai, Ngực, Dài, Tay) theo size

#### State Management
- Selections lưu vào URL params: `/products/[id]?color=white&size=L&qty=2`
- Khi click "Bắt đầu thiết kế":
  1. Validate: color + size + quantity đã chọn
  2. Tạo cart (nếu chưa có) via Medusa SDK
  3. Navigate to `/design/[productId]?variantId=[id]&color=[color]&size=[size]&qty=[qty]`

#### Interactions
| Action | Result |
|---|---|
| Click color swatch | Update mockup image, update URL param |
| Click size button | Highlight selected, update URL param |
| Change quantity | Update URL param |
| Click "Bảng size" | Open size chart Sheet |
| Click "Bắt đầu thiết kế" | Validate → navigate to design editor |

---

## Step 3 — Design Editor (`/design/[productId]`)

### Mục đích
Core feature — cho phép user thiết kế trực tiếp trên mockup áo.

### Layout (Desktop)

```
┌─────────────────────────────────────────────────────────────┐
│  Header: Logo | Product info | Side toggle | Preview | Cart │
├──────────┬──────────────────────────────────┬───────────────┤
│          │                                  │               │
│ ToolBar  │       Design Canvas              │  Layer Panel  │
│ (left)   │   (T-shirt mockup + canvas)      │  (right)      │
│          │                                  │               │
│ - Select │   ┌────────────────────┐         │ - Layer list  │
│ - Text   │   │  Print Area        │         │ - Reorder     │
│ - Image  │   │  (dashed border)   │         │ - Visibility  │
│ - Shape  │   │                    │         │ - Delete      │
│ - Undo   │   │  [User elements]   │         │ - Lock        │
│ - Redo   │   │                    │         │               │
│          │   └────────────────────┘         │               │
│          │                                  │               │
├──────────┴──────────────────────────────────┴───────────────┤
│  Context Toolbar: (changes based on selected element)       │
│  Text: Font, Size, Color, Bold, Italic, Align              │
│  Image: Flip, Crop, Opacity, Filters                        │
└─────────────────────────────────────────────────────────────┘
```

### Layout (Mobile)
- Canvas full-width
- ToolBar: bottom sheet (fixed bottom)
- Layer Panel: drawer from right
- Context Toolbar: bottom sheet above ToolBar

### ToolBar (Left Panel)

| Tool | Icon | Action |
|---|---|---|
| Select | Cursor | Default mode — click to select, drag to move |
| Text | Type | Click canvas → add text box → open TextEditor |
| Image | Image | Open file picker / drag-drop zone |
| Template | Layout | Open TemplateGallery drawer |
| Undo | RotateCcw | Undo last action (Ctrl+Z) |
| Redo | RotateCw | Redo (Ctrl+Shift+Z) |

### Text Editor (Context Panel)

Xuất hiện khi chọn tool Text hoặc click vào text element.

| Control | Type | Options |
|---|---|---|
| Font family | Select dropdown | 10-15 fonts (Google Fonts: Roboto, Open Sans, Montserrat, Playfair Display, Dancing Script...) |
| Font size | Slider + input | 12–120px |
| Color | Color picker | Preset colors + custom hex input |
| Bold | Toggle | fontWeight 400/700 |
| Italic | Toggle | fontStyle normal/italic |
| Underline | Toggle | underline true/false |
| Alignment | Button group | left / center / right |
| Letter spacing | Slider | 0–20px |
| Line height | Slider | 1.0–3.0 |

### Image Uploader

| Feature | Detail |
|---|---|
| Trigger | Click Image tool hoặc drag file vào canvas |
| Accepted formats | JPG, PNG, SVG |
| Max file size | 10MB |
| Upload flow | File → `URL.createObjectURL` → add to canvas (local only, no server upload yet) |
| Auto-scale | Scale to fit within print area while maintaining aspect ratio |
| Error states | File too large, wrong format → toast error |

### Template Gallery

- Trigger: Click Template tool → open drawer/modal
- Grid hiển thị templates preview
- Categories: Sport, Minimal, Vintage, Funny, Custom text
- Click template → load Fabric.js JSON vào canvas
- Confirm dialog nếu canvas đang có elements: "Áp dụng template sẽ xóa thiết kế hiện tại"

### Layer Panel (Right Panel)

| Feature | Detail |
|---|---|
| Layer list | Hiển thị tất cả elements, layer trên cùng = trên cùng canvas |
| Thumbnail | Preview nhỏ của element |
| Name | Auto-generated: "Text 1", "Image 2", hoặc custom name |
| Reorder | Drag & drop để thay đổi z-index |
| Visibility | Eye icon toggle → show/hide element |
| Lock | Lock icon toggle → prevent selection/editing |
| Delete | Trash icon → xóa element (with undo support) |
| Select | Click layer → select element on canvas |

### Canvas Interactions

| Interaction | Behavior |
|---|---|
| Click element | Select → show bounding box + handles |
| Drag element | Move within canvas (snap to guides if near center/edge) |
| Drag handle (corner) | Resize proportionally |
| Drag handle (edge) | Resize freely |
| Rotate handle | Rotate element |
| Double-click text | Enter inline editing mode |
| Delete key | Remove selected element |
| Ctrl+Z / Ctrl+Shift+Z | Undo / Redo |
| Ctrl+C / Ctrl+V | Copy / Paste element |
| Ctrl+A | Select all elements |
| Element outside print area | Red highlight on element + warning toast |

### Side Toggle (Front/Back)

- Button group: "Mặt trước" / "Mặt sau"
- Switching sides:
  1. Save current side's canvas JSON to Zustand
  2. Load other side's canvas JSON (or blank if first time)
  3. Update mockup background image
- Mỗi side có canvas state riêng biệt

### Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+Z` | Undo |
| `Ctrl+Shift+Z` | Redo |
| `Delete` / `Backspace` | Delete selected |
| `Ctrl+C` | Copy |
| `Ctrl+V` | Paste |
| `Ctrl+A` | Select all |
| `Ctrl+D` | Duplicate selected |
| `Escape` | Deselect all |
| `Arrow keys` | Nudge selected 1px |
| `Shift+Arrow` | Nudge selected 10px |

---

## Step 4 — Preview (`PreviewModal`)

### Trigger
- Click "Xem trước" button trên header của editor

### Layout
- Full-screen modal (shadcn Dialog full-screen)
- Center: Mockup áo thực tế với design được ghép lên
- Below: Product info (tên áo, màu, size, số lượng, giá)
- Actions:
  - "Quay lại chỉnh sửa" → close modal, back to editor
  - "Thêm vào giỏ hàng" → trigger export + upload + add to cart

### Mockup Generation
- Overlay design PNG (từ canvas export) lên ảnh mockup áo
- Perspective transform nếu cần (cho realistic look)
- Hiển thị cả front và back (nếu đã design cả 2 mặt)

### "Thêm vào giỏ hàng" Flow
1. Validate: tất cả elements trong print area
2. Export PNG (3000×3000, 300 DPI)
3. Export JSON (canvas state)
4. `POST /api/upload-design` → get presigned URLs
5. `PUT` PNG và JSON lên S3 via presigned URLs
6. `POST /store/carts/{cartId}/line-items` with metadata:
   ```json
   {
     "variant_id": "variant_xxx",
     "quantity": 2,
     "metadata": {
       "design_png_url": "https://s3.../designs/abc/front.png",
       "design_json_url": "https://s3.../designs/abc/front.json",
       "design_side": "front"
     }
   }
   ```
7. Success → navigate to `/cart`
8. Error → toast error, keep modal open

### Loading States
- "Đang xuất thiết kế..." (export PNG/JSON)
- "Đang tải lên..." (upload to S3)
- "Đang thêm vào giỏ..." (add to cart API)
- Progress bar hoặc spinner trong CTA button

---

## Step 5 — Cart (`/cart`)

### Layout
- **Cart items list**: Mỗi item hiển thị:
  - Thumbnail: design mockup preview
  - Product name + variant (màu, size)
  - Design preview (click để xem lớn)
  - Quantity selector (có thể chỉnh, min 1)
  - Price × quantity = subtotal
  - "Chỉnh sửa thiết kế" link → navigate back to `/design/[productId]` with saved JSON
  - Delete button (X)
- **Order summary** (sidebar right):
  - Subtotal
  - Phí vận chuyển (tính sau khi nhập địa chỉ, hoặc hiện "Tính khi checkout")
  - **Total**
  - "Tiến hành thanh toán" button → navigate to `/checkout`
- **Empty state**: Illustration + "Giỏ hàng trống" + CTA "Bắt đầu thiết kế"

### "Chỉnh sửa thiết kế" Flow
1. Load design JSON từ S3 URL trong metadata
2. Navigate to `/design/[productId]?edit=true&jsonUrl=[url]`
3. Editor loads JSON vào canvas
4. User edit → re-export → re-upload → update cart line item metadata

### Interactions
| Action | Result |
|---|---|
| Change quantity | Update cart via Medusa SDK, recalculate totals |
| Delete item | Remove line item, update cart |
| Click design thumbnail | Open lightbox/modal with full preview |
| Click "Chỉnh sửa" | Navigate back to editor with saved state |
| Click "Tiến hành thanh toán" | Navigate to `/checkout` |

---

## Step 6 — Checkout (`/checkout`)

### Layout
- 2 columns: Form (left 60%) + Order Summary (right 40%)

### Form Sections (react-hook-form + Zod validation)

#### 6.1 Thông tin giao hàng
| Field | Type | Validation |
|---|---|---|
| Họ và tên | Text input | Required, min 2 chars |
| Số điện thoại | Tel input | Required, VN phone format (10 digits, starts with 0) |
| Email | Email input | Required, valid email |
| Tỉnh/Thành phố | Select | Required, list of 63 tỉnh/thành |
| Quận/Huyện | Select | Required, depends on Tỉnh |
| Phường/Xã | Select | Required, depends on Quận |
| Địa chỉ chi tiết | Textarea | Required, min 5 chars |
| Ghi chú | Textarea | Optional |

#### 6.2 Phương thức thanh toán
| Method | Description |
|---|---|
| COD | Thanh toán khi nhận hàng — default selected |
| VNPay | Thanh toán online qua VNPay — redirect to VNPay gateway |

- Radio button group để chọn
- VNPay: hiển thị logo VNPay + note "Bạn sẽ được chuyển đến VNPay để thanh toán"

#### 6.3 Order Summary (Right sidebar)
- List cart items (thumbnail, name, qty, price)
- Subtotal
- Phí vận chuyển (calculated based on address)
- Discount code input + Apply button
- **Total** (bold, large)
- "Đặt hàng" button

### Form Submission Flow
1. Validate all fields (Zod schema)
2. Update cart shipping address via Medusa SDK
3. Add shipping method
4. If COD:
   - `POST /store/carts/{id}/complete`
   - Navigate to `/checkout/success?orderId=[id]`
5. If VNPay:
   - Create payment session
   - Redirect to VNPay gateway
   - VNPay callback → complete cart → redirect to success

### Error Handling
- Field-level validation errors (inline, real-time on blur)
- API errors → toast notification
- Payment failure → show error, allow retry
- Network error → "Vui lòng thử lại" with retry button

---

## Step 7 — Order Success (`/checkout/success`)

### Trigger
- Redirect sau khi cart complete thành công
- URL: `/checkout/success?orderId=[id]`

### Layout
- Success icon (checkmark animation — motion)
- "Đặt hàng thành công!" heading
- Order ID (copyable)
- Order summary: items, total, shipping address
- Estimated delivery: "3-5 ngày làm việc"
- Actions:
  - "Theo dõi đơn hàng" → navigate to `/orders/[id]`
  - "Tiếp tục mua sắm" → navigate to `/`
  - "Thiết kế thêm" → navigate to `/products`

### Background Actions (triggered by backend)
Khi order được placed, backend tự động:
1. `order.placed` event fires
2. Subscriber triggers `send-to-print-shop` workflow
3. Workflow: fetch order → build payload → send to print shop API
4. Create PrintJob record with status "pending"
5. (Future) Send confirmation email to customer

---

## Step 8 — Order Tracking (`/orders/[id]`)

### Layout
- **Order header**: Order ID, ngày đặt, trạng thái tổng
- **Progress tracker**: Visual step indicator
  ```
  Đã đặt hàng → Đang in → Đang giao → Đã nhận
       ✅           🔄         ⬜         ⬜
  ```
- **Order details**:
  - Items list (design thumbnail, product info, qty, price)
  - Click design thumbnail → view full design
- **Shipping info**: Tên, SĐT, địa chỉ
- **Payment info**: Phương thức, trạng thái

### Status Mapping
| PrintJob Status | UI Display | Progress Step |
|---|---|---|
| `pending` | "Đang xử lý" | Step 1 active |
| `printing` | "Đang in" | Step 2 active |
| `shipped` | "Đang giao hàng" | Step 3 active |
| `delivered` | "Đã giao hàng" | Step 4 complete |
| `failed` | "Lỗi xử lý" | Error state with contact info |

### Data Source
- `GET /store/orders/{id}` via Medusa SDK
- PrintJob status from order metadata or custom endpoint
- Poll every 30s hoặc dùng SSE cho real-time updates (future)

---

## Responsive Breakpoints

| Breakpoint | Width | Layout changes |
|---|---|---|
| Mobile | < 768px | Single column, bottom toolbars, stacked forms |
| Tablet | 768-1024px | 2 columns where possible, side drawers |
| Desktop | > 1024px | Full layout, 3-column editor, side panels |

## Error States (Global)

| Scenario | UI |
|---|---|
| Network error | Toast: "Không thể kết nối. Vui lòng kiểm tra mạng." |
| API error (4xx) | Toast with specific message |
| API error (5xx) | Toast: "Đã xảy ra lỗi. Vui lòng thử lại sau." |
| Page not found | Custom 404 page with CTA back to home |
| Canvas crash | ErrorBoundary: "Editor gặp lỗi" + "Tải lại" button |
| Upload failed | Toast: "Tải file thất bại" + retry option |

## Loading States

| Page/Action | Loading UI |
|---|---|
| Product listing | Skeleton cards (grid layout) |
| Product detail | Skeleton layout (image + info) |
| Design editor init | Full-page spinner + "Đang tải editor..." |
| Export design | Button spinner + progress text |
| Add to cart | Button spinner |
| Checkout submit | Full overlay spinner + "Đang xử lý đơn hàng..." |
| Order tracking | Skeleton timeline |
