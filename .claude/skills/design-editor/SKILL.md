---
name: design-editor
description: Fabric.js v7 canvas editor patterns — initialization, print area constraints, PNG/JSON export, undo/redo, drag & drop image upload. Use when building or modifying the design editor components.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
---

# Skill: Design Editor (Fabric.js v7)

## Fabric.js v7 Canvas Initialization

CRITICAL: Fabric.js must be lazy-loaded to avoid SSR crashes.

```tsx
// components/design-editor/DesignCanvas.tsx
"use client"

import { useEffect, useRef } from "react"
import { useDesignStore } from "@/lib/store/design.store"

export default function DesignCanvas() {
  const canvasRef = useRef<HTMLCanvasElement>(null)
  const setCanvas = useDesignStore((s) => s.setCanvas)

  useEffect(() => {
    let fabricCanvas: fabric.Canvas | null = null

    async function init() {
      // Lazy import — NEVER import at top level
      const fabric = await import("fabric")

      if (!canvasRef.current) return

      fabricCanvas = new fabric.Canvas(canvasRef.current, {
        width: 600,
        height: 600,
        backgroundColor: "#ffffff",
        selection: true,
        preserveObjectStacking: true,
      })

      setCanvas(fabricCanvas)
    }

    init()

    return () => {
      fabricCanvas?.dispose()
    }
  }, [setCanvas])

  return <canvas ref={canvasRef} />
}
```

### Lazy Loading in Next.js Page

```tsx
import dynamic from "next/dynamic"

const DesignEditorRoot = dynamic(
  () => import("@/components/design-editor/DesignEditorRoot"),
  { ssr: false }
)
```

## Print Area Constraint

The print area is a rectangular safe zone on the T-shirt where designs can be printed.

```ts
// lib/canvas/constraints.ts
export const PRINT_AREA = {
  // Relative to canvas (600x600)
  x: 100,
  y: 80,
  width: 400,
  height: 440,
} as const

export function isWithinPrintArea(obj: fabric.FabricObject): boolean {
  const bounds = obj.getBoundingRect()
  return (
    bounds.left >= PRINT_AREA.x &&
    bounds.top >= PRINT_AREA.y &&
    bounds.left + bounds.width <= PRINT_AREA.x + PRINT_AREA.width &&
    bounds.top + bounds.height <= PRINT_AREA.y + PRINT_AREA.height
  )
}

export function validateAllObjects(canvas: fabric.Canvas): {
  valid: boolean
  outOfBounds: fabric.FabricObject[]
} {
  const objects = canvas.getObjects()
  const outOfBounds = objects.filter((obj) => !isWithinPrintArea(obj))
  return { valid: outOfBounds.length === 0, outOfBounds }
}
```

### Drawing Print Area Overlay

```ts
export async function drawPrintAreaOverlay(canvas: fabric.Canvas) {
  const fabric = await import("fabric")

  const printArea = new fabric.Rect({
    left: PRINT_AREA.x,
    top: PRINT_AREA.y,
    width: PRINT_AREA.width,
    height: PRINT_AREA.height,
    fill: "transparent",
    stroke: "#00aaff",
    strokeWidth: 2,
    strokeDashArray: [5, 5],
    selectable: false,
    evented: false,
    excludeFromExport: true,
  })

  canvas.add(printArea)
  canvas.sendObjectToBack(printArea)
}
```

## Export PNG at 300 DPI

```ts
// lib/canvas/export.ts
import { DESIGN_EXPORT } from "@tshirt/shared"

export function exportToPng(canvas: fabric.Canvas): Blob {
  const multiplier = DESIGN_EXPORT.WIDTH / canvas.getWidth()

  const dataUrl = canvas.toDataURL({
    format: "png",
    quality: 1,
    multiplier,
    filter: (obj: fabric.FabricObject) => !obj.excludeFromExport,
  })

  const byteString = atob(dataUrl.split(",")[1])
  const mimeString = dataUrl.split(",")[0].split(":")[1].split(";")[0]
  const ab = new ArrayBuffer(byteString.length)
  const ia = new Uint8Array(ab)
  for (let i = 0; i < byteString.length; i++) {
    ia[i] = byteString.charCodeAt(i)
  }
  return new Blob([ab], { type: mimeString })
}
```

## Export JSON State

```ts
export function exportToJson(canvas: fabric.Canvas): string {
  return JSON.stringify(canvas.toJSON([
    "excludeFromExport",
    "name",
    "layerName",
  ]))
}

export function loadFromJson(canvas: fabric.Canvas, json: string): Promise<void> {
  return canvas.loadFromJSON(JSON.parse(json))
}
```

## Undo/Redo with History Snapshots

```ts
// Inside design.store.ts
const MAX_HISTORY = 30

saveSnapshot: () => {
  const { canvas, history, historyIndex } = get()
  if (!canvas) return

  const json = JSON.stringify(canvas.toJSON())
  const newHistory = history.slice(0, historyIndex + 1)
  newHistory.push(json)

  if (newHistory.length > MAX_HISTORY) {
    newHistory.shift()
  }

  set({ history: newHistory, historyIndex: newHistory.length - 1 })
}

undo: () => {
  const { canvas, history, historyIndex } = get()
  if (!canvas || historyIndex <= 0) return
  const newIndex = historyIndex - 1
  canvas.loadFromJSON(JSON.parse(history[newIndex]))
  set({ historyIndex: newIndex })
}

redo: () => {
  const { canvas, history, historyIndex } = get()
  if (!canvas || historyIndex >= history.length - 1) return
  const newIndex = historyIndex + 1
  canvas.loadFromJSON(JSON.parse(history[newIndex]))
  set({ historyIndex: newIndex })
}
```

## Drag & Drop Image Upload

```ts
const MAX_FILE_SIZE = 10 * 1024 * 1024 // 10MB
const ACCEPTED_TYPES = ["image/jpeg", "image/png", "image/svg+xml"]

function handleDrop(e: DragEvent) {
  e.preventDefault()
  const file = e.dataTransfer?.files[0]
  if (!file) return

  if (!ACCEPTED_TYPES.includes(file.type)) {
    toast.error("Only JPG, PNG, SVG files are supported")
    return
  }
  if (file.size > MAX_FILE_SIZE) {
    toast.error("File must be under 10MB")
    return
  }

  addImageToCanvas(file)
}

async function addImageToCanvas(file: File) {
  const fabric = await import("fabric")
  const url = URL.createObjectURL(file)
  const img = await fabric.FabricImage.fromURL(url)

  const scale = Math.min(
    PRINT_AREA.width / img.width!,
    PRINT_AREA.height / img.height!,
    1
  )
  img.scale(scale)
  img.set({
    left: PRINT_AREA.x + (PRINT_AREA.width - img.getScaledWidth()) / 2,
    top: PRINT_AREA.y + (PRINT_AREA.height - img.getScaledHeight()) / 2,
  })

  canvas.add(img)
  canvas.setActiveObject(img)
  canvas.renderAll()
  saveSnapshot()
}
```
