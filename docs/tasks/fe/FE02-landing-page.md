# FE02 — Landing Page

## Scope
Trang chủ — thu hút khách, giới thiệu dịch vụ, dẫn vào luồng thiết kế.

## Reference
- `docs/user-flow.md` → Step 1

## Tasks

### FE02.1 — Page Route & Layout
- [ ] Tạo `app/(store)/page.tsx`
- [ ] Setup page metadata (title, description, OG tags)
- [ ] Import và compose các sections

### FE02.2 — Hero Section
- [ ] Full-width hero với background image/gradient
- [ ] Headline + subheadline text
- [ ] CTA button "Thiết kế ngay" → link to `/products`
- [ ] Secondary CTA "Xem mẫu" → scroll to templates
- [ ] Responsive: stack vertically on mobile
- [ ] Entry animation (motion fade-in)

### FE02.3 — How It Works Section
- [ ] 3 steps layout (icon + title + description)
- [ ] Icons từ lucide-react
- [ ] Stagger fade-in animation on scroll (motion + IntersectionObserver)
- [ ] Responsive: horizontal → vertical on mobile

### FE02.4 — Template Gallery Section
- [ ] Grid 4 cột desktop, 2 tablet, 1 mobile
- [ ] Template card: mockup image + name + tag badge
- [ ] Hiển thị 6-8 featured templates
- [ ] Click card → navigate to `/design/[productId]?template=[id]`
- [ ] "Xem tất cả" button → `/products`
- [ ] Data: hardcoded initially (static templates array)

### FE02.5 — Social Proof Section
- [ ] Stats counter (animated count-up on scroll)
- [ ] Customer photos grid (3 cột)
- [ ] Testimonial cards (tên, avatar, quote)
- [ ] Data: hardcoded initially

### FE02.6 — Pricing Section
- [ ] 3 pricing cards: T-shirt / Polo / Hoodie
- [ ] Mỗi card: giá, mô tả, features list, CTA button
- [ ] Highlight "Phổ biến" badge trên T-shirt card
- [ ] CTA → `/products?type=[type]`

### FE02.7 — FAQ Section
- [ ] Accordion component (shadcn Collapsible hoặc custom)
- [ ] 5-7 câu hỏi thường gặp
- [ ] Smooth expand/collapse animation

### FE02.8 — Header Component
- [ ] Logo (text hoặc SVG)
- [ ] Navigation: Trang chủ, Sản phẩm, Về chúng tôi
- [ ] Cart icon với badge (item count)
- [ ] Mobile: hamburger menu → Sheet drawer
- [ ] Sticky on scroll

### FE02.9 — Footer Component
- [ ] Logo + tagline
- [ ] Link columns: Sản phẩm, Hỗ trợ, Chính sách
- [ ] Social media icons
- [ ] Copyright text

## Acceptance Criteria
- Landing page render đúng trên desktop + mobile
- Tất cả CTA navigate đúng route
- Animations smooth, không jank
- Lighthouse Performance > 80

## Dependencies
- FE01 (project setup)

## Estimated Subtasks: 9
