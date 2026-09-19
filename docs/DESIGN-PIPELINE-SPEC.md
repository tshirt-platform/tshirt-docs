# Design Pipeline Spec — Editor → Preview → Print Export

Spec kỹ thuật cho 3 giai đoạn của luồng thiết kế áo. Thay thế các hằng số hardcode hiện tại bằng mô hình dẫn xuất từ số đo áo thật.

> Trạng thái: **ĐÃ TRIỂN KHAI** (xem mục 9 về những chỗ khác spec). Tiến độ chi tiết: `docs/CHECKLIST.md` → Phase 7.

---

## 0. Bối cảnh — vì sao viết lại

Kiểm tra code hiện tại phát hiện 6 vấn đề gốc:

| # | Vấn đề | Vị trí |
|---|---|---|
| 1 | Ảnh upload bị co còn 14–28% print area. `scaleDpi = area.width/(dpi×inches)` là hằng số, không phụ thuộc kích thước ảnh | `constraints.ts:31` |
| 2 | Ba nguồn sự thật chỏi nhau: `DESIGN_EXPORT 3000×3000` (=10in), `PRINT_PHYSICAL_WIDTH_INCHES=12`, print area aspect 0.80 | `order.constants.ts:15`, `constraints.ts:19` |
| 3 | Test chốt sai: assert ảnh 3000px phủ print area = **250 DPI**, đang pass | `constraints.test.ts` |
| 4 | `VariantSelector` gửi `variantId/color/size/qty` qua URL nhưng `page.tsx` không đọc `searchParams` → editor không biết màu/size | `design/[productId]/page.tsx` |
| 5 | `COLOR_MAP` key tiếng Anh, DB lưu tiếng Việt → 9/10 swatch rơi về `bg-gray-300` | `VariantSelector.tsx:25` |
| 6 | `PrintJob` không lưu size/màu/mặt in/vị trí. `extract-design-data` drop `design_side` | `print-job.ts`, `extract-design-data.ts:30` |

Print area hiện tại (`{x:224,y:153,w:352,h:438}`) là số ước lượng bằng mắt, không đo từ art.

---

## 1. Quyết định đã chốt

| Hạng mục | Quyết định |
|---|---|
| Nguồn sự thật kích thước | **milimet**. Pixel và toạ độ canvas đều dẫn xuất, không hardcode |
| Khổ in trần (máy) | **279.4 × 355.6 mm** (11×14in) — vừa khổ A3, vừa platen DTG 14×16in |
| Khổ in thực tế | **Dẫn xuất từ số đo áo** (mục 2), trần ở giá trị trên |
| Một khổ cho mọi size | Có. Tính theo size nhỏ nhất để vừa tất cả size |
| Độ phân giải in | 300 DPI, sRGB, PNG nền trong suốt |
| Preview | Render **sau khi save editor**, không realtime |
| Engine preview | **Python + OpenCV**, service riêng trong docker-compose |
| Ảnh mockup | 1 ảnh/loại áo/mặt. Các màu còn lại **suy ra bằng thuật toán** |

---

## 2. Mô hình số đo & dẫn xuất print area

### 2.1 Admin nhập gì

Mỗi **product**, nhập bảng số đo theo size (đơn vị cm). Dữ liệu này hiện đang hardcode ở `SizeChart.tsx` → chuyển vào `product.metadata.size_chart`.

```jsonc
{
  "size_chart": [
    { "size": "S",   "shoulder": 42, "chest": 96,  "length": 68 },
    { "size": "M",   "shoulder": 44, "chest": 100, "length": 70 },
    { "size": "L",   "shoulder": 46, "chest": 104, "length": 72 },
    { "size": "XL",  "shoulder": 48, "chest": 108, "length": 74 },
    { "size": "XXL", "shoulder": 50, "chest": 112, "length": 76 }
  ],
  "neck_drop_front_cm": 8,
  "neck_drop_back_cm": 2.5
}
```

`chest` là **vòng ngực**; bề rộng thân phẳng = `chest / 2`.

### 2.2 Hằng số hệ thống

```ts
MAX_PRINT_W_MM   = 279.4   // 11in — trần máy
MAX_PRINT_H_MM   = 355.6   // 14in — trần máy
PRINT_ASPECT     = 279.4 / 355.6   // 0.7857
CHEST_USAGE      = 0.55    // % bề rộng thân được phép in
GAP_BELOW_COLLAR = 50      // mm, khoảng trống giữa mép dưới cổ áo và cạnh trên hình in
HEM_MARGIN       = 60      // mm, chừa trên lai
TARGET_DPI       = 300
```

**Chuỗi đo theo chiều dọc** (`GAP_BELOW_COLLAR` nằm ở đâu):

```
   ─── HPS — điểm cao nhất của vai (mốc chuẩn ngành)
    │
    │   neck_drop = 80mm        độ sâu cổ áo
    │
   ─┴─  mép dưới cổ áo
    │
    │   GAP_BELOW_COLLAR = 50mm  ← khoảng trống bắt buộc
    │
   ┌┴──────────────────┐         cạnh trên vùng in
   │                   │
   │   VÙNG IN         │  336mm
   │   264 × 336 mm    │
   └───────────────────┘
    │
    │   ≥ HEM_MARGIN = 60mm
   ─┴─  lai áo

   top_offset_from_HPS = neck_drop + GAP = 80 + 50 = 130mm
```

Vì sao cần gap, không in sát cổ:
1. Hình sát cổ trông như dán sticker, không giống áo in thật
2. Vải ngay dưới cổ cong theo xương đòn → in ở đó dễ méo và nhăn
3. Bàn ép DTG khó ép phẳng vùng sát cổ
4. Đây là tham số chỉnh "gu": gap nhỏ = hình cao/năng động, gap lớn = hình thấp/trầm

Vì sao các hằng số này an toàn khi **chưa có xưởng in**: mọi con số đều chọn theo hướng "xưởng nào cũng làm được" — khổ 264×336mm lọt khổ **DTF A3** (297×420mm, phổ biến và rẻ nhất ở VN) và nhỏ hơn mọi bàn ép DTG thông dụng (14×16in trở lên). Khi tìm được xưởng có giới hạn khác, chỉ sửa `MAX_PRINT_W_MM` / `MAX_PRINT_H_MM` là toàn hệ thống tự tính lại.

### 2.3 Công thức

```
size nhỏ nhất (S) làm ràng buộc:
  flat_width_mm  = chest_cm × 10 / 2
  body_length_mm = length_cm × 10

  print_w_mm = min(MAX_PRINT_W_MM, flat_width_mm × CHEST_USAGE)
  print_h_mm = min(MAX_PRINT_H_MM, print_w_mm / PRINT_ASPECT)

  top_offset_front_mm = neck_drop_front_cm × 10 + GAP_BELOW_COLLAR
  top_offset_back_mm  = neck_drop_back_cm  × 10 + GAP_BELOW_COLLAR

  validate: top_offset_mm + print_h_mm <= body_length_mm - HEM_MARGIN

  artwork_px = round(print_mm / 25.4 × TARGET_DPI)
```

### 2.4 Kết quả cho T-shirt Basic (size S làm chuẩn)

| Thông số | Giá trị |
|---|---|
| flat_width | 480 mm |
| print_w × print_h | **264 × 336 mm** |
| artwork | **3118 × 3969 px** @ 300 DPI |
| top_offset front / back | 130 mm / 75 mm |
| Kiểm tra dọc (front) | 130 + 336 = 466 ≤ 680 − 60 = 620 ✓ |

Đổi số đo trong admin → print area và canvas tự tính lại. Thêm Polo/Hoodie chỉ cần nhập số đo, không sửa code.

### 2.5 Ánh xạ lên ảnh line-art

Mỗi ảnh mockup line-art lưu 3 tham số calibration, **tự động detect khi upload** (thuật toán đã verify trên 2 ảnh hiện có):

| Ảnh | bodyWidthPx | centerX | hpsY |
|---|---|---|---|
| `front-unline.png` (902×899) | 505 | 451 | 15 |
| `back-unline.png` (817×814) | 458 | 408 | 12 |

Thuật toán detect: quét từng hàng tìm biên silhouette (alpha > 40), lấy 2 nét dọc ổn định dưới vùng tay áo làm đường sườn → `bodyWidthPx`, `centerX`; hàng opaque đầu tiên vùng vai → `hpsY`.

```
mmPerPx   = flat_width_mm / bodyWidthPx
rect_w_px = print_w_mm / mmPerPx
rect_h_px = print_h_mm / mmPerPx
rect_x    = centerX - rect_w_px / 2
rect_y    = hpsY + top_offset_mm / mmPerPx
```

Kết quả cho T-shirt Basic:

| Mặt | rect (px trên ảnh gốc) |
|---|---|
| front | x=312, y=152, w=278, h=354 |
| back | x=282, y=84, w=252, h=321 |

> ⚠️ Line-art hiện tại vẽ áo **hẹp hơn áo thật**: tỉ lệ thân art 505/883 = 0.57, áo thật size S là 480/680 = 0.71. Ta ưu tiên giữ **đúng aspect khổ in** (0.786) để WYSIWYG không vỡ, scale theo bề rộng. Hệ quả: khung trên art trông thấp hơn thực tế ~10%. Chấp nhận cho v1; thay art đúng tỉ lệ sau.

---

## 3. Phần A — Editor

### A1. Canvas
- Backing store ở **độ phân giải master** = `artwork_px` (3118×3969). Hiển thị thu nhỏ bằng CSS (retina pattern) → export không cần tính lại, không mất nét.
- Print area = toàn bộ canvas. Line-art vẽ **ngoài** canvas như lớp nền định vị, không phải background của Fabric.
- Đổi cách hiện tại: bỏ `CANVAS_SIZE = 800×800` cố định.

### A2. Nền line-art + màu vải
- Layer thứ tự: `[line-art đã tint] → [vùng print area] → [object user]`
- Art có thân **trắng đục**, ngoài **trong suốt**, nét đen (đã sample pixel) → dùng `BlendColor` mode `multiply` với hex màu vải.
- Màu tối (Đen/Navy/Xám đậm) làm nét đen chìm mất → chuyển art sang **SVG 2 lớp** (thân + nét) để vải tối thì đổi nét sang màu sáng. Nếu chưa kịp làm SVG, v1 dùng `multiply` + tăng sáng nét bằng overlay stroke.

### A3. Color registry
Thay `COLOR_MAP` (Tailwind class, key tiếng Anh, 9/10 màu không khớp DB) bằng registry thật, lưu ở `product.metadata.colors`:

```jsonc
{
  "name": "Đen",
  "hex": "#1A1A1A",
  "is_dark": true,
  "needs_underbase": true,
  "pantone_tcx": null,      // điền khi có swatch card của NCC
  "supplier_code": null     // điền khi chốt nhà cung cấp vải
}
```

Bảng 10 màu — `hex` là **màu hiển thị trên màn hình**, chọn theo sắc độ vải cotton thật (vải luôn trầm hơn màu digital thuần):

| Tên (DB) | hex | is_dark | needs_underbase |
|---|---|---|---|
| Trắng | `#F4F4F0` | false | false |
| Kem | `#E6DCC6` | false | true |
| Xám nhạt | `#C8C8C4` | false | true |
| Vàng | `#F2C83C` | false | true |
| Đỏ | `#C8102E` | true | true |
| Xanh lá | `#2E7D4F` | true | true |
| Xanh dương | `#1D4F91` | true | true |
| Xám đậm | `#4A4A4A` | true | true |
| Navy | `#1F2A44` | true | true |
| Đen | `#1A1A1A` | true | true |

- `is_dark` → dùng cho UI: vải tối thì đổi nét line-art sang màu sáng, và cảnh báo design tối trên vải tối.
- `needs_underbase` → dùng cho `spec.json`: DTG cần lớp lót trắng trên **mọi vải không phải trắng**, ảnh hưởng giá in.
- `pantone_tcx` / `supplier_code` để `null` ở v1. Hex **không phải** mã vải — mã thật chỉ có khi cầm swatch card của nhà cung cấp. Xem mục 7.

### A4. Truyền context vào editor
`design/[productId]/page.tsx` đọc `searchParams` → `variantId`, `color`, `size`, `qty` → store. `VariantSelector` đã gửi sẵn, chỉ thiếu đầu nhận.

### A5. Ảnh upload — sửa logic scale
- Đặt ảnh ở **70% bề rộng print area**, căn giữa (thay cho công thức DPI hiện tại).
- Tính DPI hiệu dụng, **cảnh báo** chứ không cưỡng chế co ảnh:
  - `≥300`: xanh — `150–299`: vàng — `<150`: đỏ + confirm dialog
- `DpiIndicator` giữ nguyên, thêm cảnh báo tại thời điểm upload.

### A6. Guard
- Cảnh báo design tối trên áo tối / sáng trên áo sáng.
- Cảnh báo object vượt print area (đã có `validateAllObjects`).

---

## 4. Phần B — Preview

### B1. Luồng (chốt: render sau khi save)

```
User bấm "Lưu thiết kế"
  → export artwork PNG (master res, nền trong) → S3
  → POST /render tới Python service
  → service render → upload preview → trả URL
  → hiện PreviewModal
  → user xác nhận → add to cart kèm metadata
```

Không render realtime → không giới hạn thời gian, chạy server-side thoải mái.

### B2. Python service

Container mới trong `docker-compose.yml`. Stack: FastAPI + OpenCV + NumPy + Pillow.

```
POST /render
{
  "artwork_url": "https://s3/.../design.png",
  "template_id": "tshirt-front-model-01",
  "garment_hex": "#1A1A1A",
  "output_width": 1200
}
→ { "preview_url": "https://s3/.../preview.jpg" }
```

Pipeline:

| Bước | Kỹ thuật |
|---|---|
| 1. Đổi màu áo theo màu khách chọn | `garment_mask × target_hex × luminance(base)` — giữ nguyên bóng đổ |
| 2. Warp artwork vào đúng vị trí/góc nghiêng | `cv2.getPerspectiveTransform` → `cv2.warpPerspective` (4 điểm góc) |
| 3. Nếp gấp vải | `cv2.remap` với offset từ displacement map |
| 4. Ánh sáng | nhân artwork với luminance chuẩn hoá của vùng vải |
| 5. Vật che (tay/tóc) | áp occlusion mask alpha |
| 6. Ghép + encode | composite lên ảnh gốc → JPEG/WebP → S3 |

Chi phí **$0/ảnh**, ~100–300ms. Cache trên S3 theo `hash(artwork) + template_id + garment_hex`.

### B3. Admin upload mockup

Chỉ cần **1 ảnh áo trơn** cho mỗi loại áo × mặt (v1: flat-lay / ghost mannequin — xem 7.2). Các map được **sinh tự động**, không cần designer:

| Map | Cách sinh |
|---|---|
| Displacement | `grayscale(base)` → `GaussianBlur` |
| Shading | kênh luminance của `base`, chuẩn hoá về quanh 1.0 |
| Garment mask | auto bằng `rembg`/GrabCut → admin sửa tay nếu lệch |
| Quad 4 góc | **admin click 4 góc** vùng in (thao tác ~10 giây) |
| Occlusion mask | tách từ garment mask; vẽ tay nếu có tay/tóc che |

Đây chính là quy trình Photoshop chuẩn (duplicate → desaturate → blur cho displacement; Multiply cho shadow), chỉ là viết bằng NumPy.

**1 ảnh → đủ 10 màu** nhờ bước recolor ở B2.1.

### B4. Nguồn ảnh
Stock miễn phí có license thương mại, **không lấy ảnh có mặt người**. Chi tiết và lý do pháp lý ở mục 7.2.

---

## 5. Phần C — Export cho nhà in

### C1. Chuẩn artwork
| Thuộc tính | Giá trị |
|---|---|
| Format | PNG, nền trong suốt |
| Kích thước | `artwork_px` của product (T-shirt Basic: 3118×3969) |
| DPI metadata | 300 (nhúng thật qua `sharp().withMetadata({density:300})`) |
| Color space | sRGB (**không** convert CMYK — CMYK chỉ cho in lụa/offset) |
| Nội dung | chỉ design. Không mockup, không guide, không nền |
| Bán trong suốt | flatten; cảnh báo nếu có drop shadow / gradient mờ dần |

### C2. Chuẩn vị trí
Đo từ **HPS** (High Point of Shoulder — điểm cao nhất của vai):
- Cạnh trên hình cách HPS: `top_offset_mm` (front 130, back 75 cho T-shirt Basic)
- Căn giữa ngang, offset 0

### C3. Tên file
```
{orderNo}-{itemNo}-{size}-{colorCode}-{side}.png
ví dụ: 1001-1-L-BLK01-front.png
```

### C4. Package mỗi line item
```
order_1001/
  1001-1-L-BLK01-front.png     artwork
  proof-front.jpg               ảnh preview để thợ đối chiếu
  spec.json                     máy đọc
  workorder.pdf                 thợ in ra giấy
```

### C5. `spec.json`
```jsonc
{
  "spec_version": "1.0",
  "order_no": "1001",
  "item_no": 1,
  "garment": {
    "sku": "TSHIRT_BASIC-L-DEN",
    "style": "tshirt",
    "size": "L",
    "color_name": "Đen",
    "color_hex": "#1A1A1A",
    "supplier_color_code": "BLK-01",
    "quantity": 2
  },
  "prints": [{
    "side": "front",
    "method": "DTG",
    "white_underbase": true,
    "artwork_file": "1001-1-L-BLK01-front.png",
    "artwork_px": { "width": 3118, "height": 3969 },
    "dpi": 300,
    "print_size_mm": { "width": 264, "height": 336 },
    "placement": {
      "reference": "HPS",
      "top_offset_mm": 130,
      "horizontal": "center",
      "horizontal_offset_mm": 0
    }
  }],
  "proof_file": "proof-front.jpg"
}
```

`white_underbase` = true khi `color.is_dark` — ảnh hưởng giá in, phải ghi rõ.

---

## 6. Thay đổi data model

### 6.1 `@tshirt-platform/shared`
```ts
// design.types.ts — mở rộng
export interface PrintPlacement {
  side: DesignSide
  print_size_mm: { width: number; height: number }
  artwork_px: { width: number; height: number }
  top_offset_mm: number
  horizontal_offset_mm: number
}

export interface CartLineItemMetadata {
  design_png_url: string
  design_json_url: string
  design_side: DesignSide
  preview_url: string
  placement: PrintPlacement
  garment: { size: string; color_name: string; color_hex: string; supplier_color_code: string }
}
```

### 6.2 Backend
| Việc | File |
|---|---|
| Giữ `design_side`, lấy thêm variant info | `steps/extract-design-data.ts` |
| Thêm cột: `side`, `size`, `color_code`, `print_size_mm`, `placement`, `proof_url`, `spec_url` | `models/print-job.ts` + migration |
| Thống nhất status enum: model có `processing/cancelled`, shared có `printing/failed` | `print-job.ts` ↔ `order.constants.ts` |
| Thêm state `proof_approved` trước sản xuất | service + routes |
| Route xuất package zip | `api/admin/print-orders/[id]/package` |

### 6.3 Xoá / thay
- `PRINT_AREAS`, `CANVAS_SIZE`, `PRINT_PHYSICAL_WIDTH_INCHES`, `DESIGN_EXPORT.WIDTH/HEIGHT` → thay bằng hàm dẫn xuất.
- `constraints.test.ts` — sửa assert 250 DPI.
- `SIZE_DATA` trong `SizeChart.tsx` → đọc từ `product.metadata.size_chart`.
- `COLOR_MAP` trong `VariantSelector.tsx` + `ProductCard.tsx` → color registry.
- `public/images/design-editor/front.png`, `back.png` (bản có khung gạch đứt, không code nào dùng) → move sang `docs/reference/`.

---

## 7. Giả định v1 (chưa có xưởng in / nhà cung cấp thật)

Tất cả đều là **giá trị mặc định an toàn**, không chặn việc code. Khi có đối tác thật thì thay số, kiến trúc không đổi.

| Hạng mục | Giả định v1 | Thay bằng gì sau này |
|---|---|---|
| Giới hạn khổ in | 279.4 × 355.6 mm (vừa DTF A3) | Hỏi xưởng khổ tối đa → sửa 2 hằng số |
| `GAP_BELOW_COLLAR` | 50 mm | Chuẩn riêng của xưởng, hoặc chỉnh theo thẩm mỹ |
| `pantone_tcx`, `supplier_code` | `null` | Xin swatch card của NCC vải, ghi mã theo **Pantone TCX** (chuẩn dệt may, dye-on-cotton) |
| Ảnh mockup | Stock **không có mặt người** | Nâng cấp lên ảnh model có model release nếu cần |

### 7.1 Vì sao mã màu phải để trống ở v1
`hex` trong bảng A3 chỉ là **màu hiển thị**. Nó không dùng đặt vải được, vì:
- Pantone không có giá trị digital chính xác — hex chỉ là xấp xỉ
- Cùng một tên màu ("Navy") khác nhau giữa các xưởng dệt
- Mã thật nằm trên swatch card vật lý của NCC

Khi chốt NCC: xin swatch card → ghi mã TCX hoặc mã nội bộ vào `supplier_code` → `spec.json` sẽ mang mã đó cho xưởng in.

### 7.2 Ảnh mockup — ràng buộc pháp lý
**Quyết định v1: chỉ dùng ảnh flat-lay hoặc ghost mannequin, không có mặt người nhận diện được.**

Lý do: Unsplash/Pexels cho dùng thương mại nhưng **không xác minh model release** và **không bồi thường pháp lý** ($0 indemnification). Dùng ảnh người nhận diện được để bán hàng chạm vào quyền hình ảnh cá nhân — rủi ro riêng, độc lập với bản quyền ảnh. Cả 2 nền tảng đều ghi rõ trách nhiệm thuộc về người dùng.

Không có mặt người ⇒ không có rủi ro này. Và trùng khớp với hướng kỹ thuật: ảnh flat-lay chụp thẳng ít biến dạng phối cảnh nên render dễ và đẹp hơn.

Nếu sau này cần ảnh model thật: mua ở Adobe Stock / Shutterstock / iStock (có xác minh model release + bồi thường), hoặc crop không lộ mặt.

---

## 8. Thứ tự implement

| Bước | Nội dung | Phụ thuộc |
|---|---|---|
| 1 | Module dẫn xuất print area (mm → px → art rect) + test | — |
| 2 | Auto-calibrate line-art khi upload | 1 |
| 3 | Editor: canvas master-res, tint màu vải, nhận searchParams, sửa scale ảnh upload | 1, 2 |
| 4 | Export artwork + spec.json | 1, 3 |
| 5 | Python render service + docker-compose | — (song song được) |
| 6 | Admin upload mockup + click 4 góc | 5 |
| 7 | PreviewModal nối vào luồng save | 4, 5 |
| 8 | Backend: model, workflow, package zip | 4 |

---

## 9. Khác biệt giữa spec và bản triển khai

| Spec | Thực tế | Lý do |
|---|---|---|
| Canvas backing store 3118×3969 (retina) | Scene **800 đơn vị** ngang print area, export nhân `multiplier` lên 300 DPI | Không phải vẽ canvas 12 MP mỗi lần kéo thả; font, bước phím, slider giữ được cỡ dễ dùng |
| `hpsY` = 15 / 12 (ước lượng) | Dò tự động: `hpsY = 1` cho cả hai ảnh; `bodyWidthPx` = 507 (front) / 459 (back) | Định nghĩa tất định: hàng đầu tiên có điểm ảnh của áo |
| Rect front `x=312,y=152,w=278,h=354` | Tính từ số dò được, lệch khoảng 14 px so với bảng ở mục 2.5 | Hệ quả của `hpsY` |
| Print area tính trong store | Nằm ở **`@tshirt-platform/shared`** | Admin (backend) cũng cần tính khổ in |
| Package `@tshirt/shared` | Đổi thành `@tshirt-platform/shared` | GitHub Packages bắt buộc scope trùng tên org |
| Phiếu sản xuất PDF | **`workorder.html`** (in ra PDF từ trình duyệt) | Tránh thêm thư viện PDF; mọi giá trị được escape |
| Tách mask bằng `rembg` | GrabCut của OpenCV + cho phép admin tải mask tự vẽ | Không kéo thêm mô hình ML nặng; giới hạn: cần ảnh áo **sáng màu trên nền đơn sắc** |
| Cache preview trên S3 | Cache trên đĩa trong service render, khoá theo `hash(artwork)+template+màu+độ rộng` | Đơn giản hơn cho v1 |
| Trạng thái job | `pending → proof_approved → processing → shipped → delivered`, `cancelled` | Đã bỏ `pending → processing` trực tiếp; `printing/failed` cũ không còn |
| Thiết kế một mặt | `CartLineItemMetadata.designs[]`, một `PrintJob` cho mỗi mặt in | Thiết kế hai mặt là hai lệnh in |

Bảo mật thêm: URL file trong metadata giỏ hàng do khách sửa được, nên route tải gói file chỉ lấy từ các origin trong `DESIGN_FILE_ORIGINS` (mặc định rỗng ở production).
