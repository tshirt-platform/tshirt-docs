# FE04 — Design Editor (Core Feature)

## Scope
Canvas-based design editor — tính năng chính của platform.

## Reference
- `docs/user-flow.md` → Step 3
- `docs/system-flow.md` → Flow 3
- `.claude/skills/design-editor/SKILL.md`

## Tasks

### FE04.1 — Zustand Design Store (`lib/store/design.store.ts`)
- [ ] Define DesignState interface (from @tshirt/shared)
- [ ] Actions: setCanvas, setActiveTool, setSide, saveSnapshot, undo, redo
- [ ] Front/back canvas JSON storage
- [ ] History management (max 30 snapshots)
- [ ] Export state: pngUrl, jsonUrl

### FE04.2 — Canvas Initialization (`DesignCanvas.tsx`)
- [ ] Lazy import Fabric.js (`await import("fabric")`)
- [ ] Create `fabric.Canvas` with config (600×600, white bg)
- [ ] Store canvas reference in Zustand
- [ ] Cleanup on unmount (`canvas.dispose()`)
- [ ] ErrorBoundary wrapper

### FE04.3 — Print Area Overlay (`lib/canvas/constraints.ts`)
- [ ] Define PRINT_AREA constants (x, y, width, height)
- [ ] Draw dashed rectangle overlay (non-selectable, excluded from export)
- [ ] `isWithinPrintArea(obj)` validation function
- [ ] `validateAllObjects(canvas)` → returns out-of-bounds list
- [ ] Visual warning: red highlight on out-of-bounds elements

### FE04.4 — T-shirt Mockup Background
- [ ] Load T-shirt mockup image as canvas background
- [ ] Switch mockup khi đổi color
- [ ] Switch mockup khi đổi side (front/back)
- [ ] Mockup không selectable, không exportable

### FE04.5 — ToolBar Component
- [ ] Vertical toolbar (left side)
- [ ] Tool buttons: Select, Text, Image, Template, Undo, Redo
- [ ] Active state highlight
- [ ] Click → update activeTool in store
- [ ] Undo/Redo disabled state (check historyIndex)
- [ ] Mobile: bottom bar layout

### FE04.6 — Text Tool (`TextEditor.tsx`)
- [ ] Click canvas → add IText object
- [ ] Context panel khi text selected:
  - Font family select (10-15 Google Fonts)
  - Font size slider (12-120)
  - Color picker
  - Bold / Italic / Underline toggles
  - Text alignment (left/center/right)
  - Letter spacing slider
  - Line height slider
- [ ] Real-time preview khi thay đổi properties
- [ ] Double-click text → inline editing

### FE04.7 — Image Upload (`ImageUploader.tsx`)
- [ ] File input trigger (click Image tool)
- [ ] Drag & drop zone (canvas area)
- [ ] Validate: format (JPG/PNG/SVG), size (≤10MB)
- [ ] `FabricImage.fromURL()` → add to canvas
- [ ] Auto-scale to fit print area
- [ ] Center trong print area
- [ ] Error toast cho invalid files

### FE04.8 — Template Gallery (`TemplateGallery.tsx`)
- [ ] Drawer/Sheet component (from right)
- [ ] Template grid với preview thumbnails
- [ ] Category tabs: Sport, Minimal, Vintage, Funny, Text
- [ ] Click template → load JSON vào canvas
- [ ] Confirm dialog nếu canvas không trống
- [ ] 10-15 initial templates (stored as Fabric.js JSON)

### FE04.9 — Layer Panel (`LayerPanel.tsx`)
- [ ] Right sidebar panel
- [ ] List all canvas objects (reverse order = visual stack)
- [ ] Each layer row: thumbnail, name, visibility toggle, lock toggle, delete
- [ ] Drag & drop reorder (→ change z-index)
- [ ] Click layer → select object on canvas
- [ ] Highlight active layer khi select trên canvas
- [ ] Keyboard: Delete key → remove selected

### FE04.10 — Side Toggle (Front/Back)
- [ ] Button group: "Mặt trước" / "Mặt sau"
- [ ] Save current side JSON before switching
- [ ] Load target side JSON (or blank canvas)
- [ ] Update mockup background image
- [ ] Maintain separate history per side

### FE04.11 — Keyboard Shortcuts
- [ ] Ctrl+Z / Cmd+Z → Undo
- [ ] Ctrl+Shift+Z / Cmd+Shift+Z → Redo
- [ ] Delete / Backspace → Remove selected
- [ ] Ctrl+C / Ctrl+V → Copy / Paste
- [ ] Ctrl+A → Select all
- [ ] Ctrl+D → Duplicate
- [ ] Escape → Deselect
- [ ] Arrow keys → Nudge 1px (Shift+Arrow → 10px)

### FE04.12 — DesignEditorRoot (Wrapper)
- [ ] `dynamic()` import (SSR disabled)
- [ ] ErrorBoundary component
- [ ] Loading state: "Đang tải editor..."
- [ ] Read product info từ URL params
- [ ] Compose: ToolBar + DesignCanvas + LayerPanel + context panels

## Acceptance Criteria
- Editor loads trong < 3 giây
- Add text, image, template hoạt động
- Undo/Redo hoạt động (20+ steps)
- Layer panel sync đúng với canvas
- Front/back toggle giữ đúng state
- Print area validation hoạt động
- Keyboard shortcuts all working
- No SSR crashes
- ErrorBoundary catch và hiển thị fallback

## Dependencies
- FE01 (project setup)
- FE01.1 (Zustand setup)

## Estimated Subtasks: 12
