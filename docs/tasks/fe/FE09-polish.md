# FE09 — Polish & UX

## Scope
Loading states, error handling, responsive, animations, accessibility.

## Tasks

### FE09.1 — Error Boundary
- [ ] Global ErrorBoundary component
- [ ] Design editor ErrorBoundary (separate, with "Tải lại editor")
- [ ] Fallback UI: error message + retry button
- [ ] Log errors (console, future: error tracking service)

### FE09.2 — Loading States
- [ ] Product listing: skeleton cards
- [ ] Product detail: skeleton layout
- [ ] Design editor: full-page spinner + text
- [ ] Cart: skeleton items
- [ ] Checkout submit: overlay spinner
- [ ] Generic Skeleton component (reusable)

### FE09.3 — Toast Notifications
- [ ] Sonner setup (already installed)
- [ ] Success: "Đã thêm vào giỏ hàng", "Đặt hàng thành công"
- [ ] Error: "Không thể kết nối", "Tải file thất bại"
- [ ] Warning: "Element nằm ngoài vùng in"
- [ ] Consistent styling across app

### FE09.4 — Mobile Responsive
- [ ] Landing page: all sections stack properly
- [ ] Product pages: single column, drawer for filters
- [ ] Design editor: bottom toolbar, canvas scales, drawer panels
- [ ] Cart: stacked layout
- [ ] Checkout: stacked form + collapsible summary
- [ ] Test on 375px, 768px, 1024px, 1440px

### FE09.5 — Animations (motion)
- [ ] Page transitions (subtle fade)
- [ ] Section scroll-in animations (landing page)
- [ ] Modal enter/exit
- [ ] Button hover/press states
- [ ] Cart badge bounce on update
- [ ] Success page checkmark animation

### FE09.6 — 404 Page
- [ ] Custom not-found page
- [ ] Illustration + "Trang không tồn tại"
- [ ] CTA: "Về trang chủ"

### FE09.7 — SEO & Metadata
- [ ] Page-level metadata (title, description) cho mỗi route
- [ ] OpenGraph tags cho social sharing
- [ ] Structured data (Product schema) cho product pages
- [ ] Sitemap generation

## Acceptance Criteria
- No unhandled errors crash the app
- All loading states visible and smooth
- Mobile usable on all pages
- Lighthouse scores: Performance >80, Accessibility >90
- SEO: all pages have unique title + description

## Dependencies
- FE02–FE08 (all pages built)

## Estimated Subtasks: 7
