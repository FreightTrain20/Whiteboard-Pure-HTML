# Jamboard

A single-file interactive whiteboard application — no frameworks, no build step, pure HTML + CSS + JavaScript (~2100 lines). Open `Jamboard.html` in any modern browser and start creating.

## Features

### Infinite Canvas
- Pan by dragging with the middle mouse button or **Space + drag**
- Zoom with scroll wheel (**10% – 300%**), shown live in a HUD
- Dot-grid background that moves with the pan
- All nodes exist in canvas-space and transform with pan/zoom

### 9 Node Types

| Node | Description | Default Size |
|------|-------------|-------------|
| **Note** | Colored sticky note with textarea, choose from 6 pastel colors | 200 × 160 |
| **Image** | Displays an uploaded image (drag-drop or file picker) | 220 × 200 |
| **Video** | Looping muted video player with sound toggle | 220 × 200 |
| **Audio** | Audio player with play/pause, seekable progress bar, time label | 220 × 120 |
| **Text File** | Upload `.txt` or `.md` files; renders markdown (headers, bold, italic, code blocks, lists, links, images) | 300 × 250 |
| **Block** | Generic rounded rectangle with an editable label | 180 × 80 |
| **Checklist** | Add items, check/uncheck them, remove individually. Checked items move to the bottom but stay visible until manually removed. | 220 × 280 |
| **URL Embed** | Paste a URL to generate a QR code (Google QR Server API). Includes "Open in new tab" link. | 240 × 260 |
| **Kanban** | 3-column board — To Do, In Progress, Done. Drag cards between columns, add cards per column. | 500 × 300 |

### Connections
- Each node has connector ports on all four edges (top, bottom, left, right)
- Click a port to start a connection line; click another port to complete it
- Smooth SVG cubic bezier curves with clamped control points and endpoint clipping
- Lines update in real-time as nodes move
- Midpoint delete button (×) on each connection

### Node Interactions
- Drag by the header bar, resize from the bottom-right handle
- Delete any node with the × button
- Click a node to bring it to front (z-index management)
- Double-click a Note or Block to edit its text inline

### Toolbar & Menus
- **Add dropdown** — insert any node type at canvas center
- **Theme dropdown** — toggle Light / Dark mode (persisted in `localStorage`)
- **File dropdown** — Save (export `.jamboard` JSON), Import, Clear All
- Zoom In / Out / Reset buttons with current zoom percentage display
- Right-click context menu for quick node creation and file operations

### Save / Import
- Export the entire canvas as a downloadable `.jamboard` JSON file (all nodes, connections, placements, embedded media)
- Import to restore any previous state. Binary media are stored as base64 data URLs inside the JSON.

## Quick Start

1. Open `Jamboard.html` in Chrome, Firefox, or Safari
2. Click **Add → Note** (or right-click the canvas for a context menu)
3. Drag nodes around, connect them with bezier lines
4. Save your work via the File menu at any time

## Technical Highlights

- **Three coordinate systems**: viewport → canvas → SVG — converted explicitly through the transform chain (`canvasX * zoom + panX`)
- **SVG layer** for connections sits behind nodes in z-index; uses `overflow: visible` on a 1×1 px element so paths extend freely
- **FileReader API** for media uploads (image, video, audio, text) with base64 data URL encoding and size warnings for files >5 MB
- **Built-in markdown parser** — no external dependencies
- **requestAnimationFrame** loop for smooth audio progress tracking (more reliable than `timeupdate` events on data URLs)
- Works offline after the initial page load; no external libraries or CDNs

## Known Limitations

| Issue | Details |
|-------|---------|
| No persistent storage | Data is in-memory only. Save/Import via JSON download/upload works but doesn't auto-save. |
| No touch support | All interactions use mouse events. |
| No undo / redo | Node and connection changes are irreversible. |
| Window resize | `updateConnections()` runs on resize, but the SVG bounding rect change can cause a one-frame misalignment until the next interaction. |
| Large audio files (> 5 MB) | Base64 encoding inflates size by ~33%. A warning prompt appears before upload; consider IndexedDB for large binary storage in future iterations. |

## Project Structure

```
Jamboard.html          # Main app — embedded CSS, HTML, and JS (~2100 lines)
plan.md                # Original build specification
instruct.md            # Feature summary (10 node types)
nodes.md               # Node type status tracker (implemented / planned / denied)
bugs.md                # Bug fix report (14 bugs documented with root causes & fixes)
claude-info.md         # Architecture, bug history, design decisions, lessons learned
chat-recap.md          # Development session recap
requested-changes.md   # Iterative feedback log
```

## Lessons Learned

Several non-obvious technical insights emerged during development:

1. **Never use `getBoundingClientRect()` as a proxy for CSS transforms.** The bounding rect of a `transform: translate() scale()` element gives the viewport position, not internal coordinates. Always follow the transform chain explicitly.

2. **SVG element's own `getBoundingClientRect()` is unreliable under parent CSS transforms.** Its bounds shift when the canvas scales — use the explicit transform chain instead.

3. **CSS custom properties don't resolve in SVG `fill`/`stroke`** across all browsers. Use inline `style="fill: #xxx"` for SVG elements.

4. **Port positions require manual offset calculation.** Port dots with negative margins sit outside the node border; their center is not returned by `getBoundingClientRect()`.

5. **Bezier control points must be bidirectional.** A's control point pushes outward, B's pulls inward toward A. If both push outward, the curve loops back on itself.

6. **Base64 inflates binary data ~33%.** For large files this can cause silent failures during `FileReader.readAsDataURL()`. Consider IndexedDB for future iterations with heavy media use.
