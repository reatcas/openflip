# OpenFlip — Build Plan

## Phase 1: Foundation (Scene + Static Book)
1. `npm init`, install Three.js + Vite
2. `Scene.js` — WebGLRenderer, PerspectiveCamera, ambient+spot lights, shadow maps
3. `Page.js` — PlaneGeometry(1, 1.414, 32, 48), MeshStandardMaterial, double-sided
4. `Book.js` — Stack pages, position them at spine, front/back cover
5. `demo/index.html` — Render a static book with colored pages, verify shadows work
6. **Checkpoint**: A 3D book visible on screen with proper lighting and shadows

## Phase 2: Vertex Shader (Page Bend)
1. `shaders/page.vert.glsl` — Cylindrical bend uniforms: bendForce, foldAngle, bendRadius, growthFactor
2. `shaders/page.frag.glsl` — Texture sampling + paper grain noise + soft self-shadow near fold
3. `PageMaterial.js` — ShaderMaterial wrapping the GLSL, with paper texture support
4. Test: animate bendForce from 0→π to see page curl in real-time
5. **Checkpoint**: A single page visibly bends/curls with realistic deformation

## Phase 3: Flip Animation
1. `FlipAnimation.js` — Spring-damper system: state machine (idle, dragging, flipping, settling)
2. Page states: flat-right, flipping, flat-left
3. Animation loop via requestAnimationFrame, update bend uniforms per frame
4. Queue system for rapid consecutive flips
5. **Checkpoint**: Click to flip pages with smooth physics animation

## Phase 4: Interaction (Mouse + Touch)
1. `Interaction.js` — Pointer events (pointerdown/move/up) on renderer canvas
2. Raycasting to detect which page edge is grabbed
3. Map pointer drag distance to foldAngle + bendForce
4. Swipe detection (velocity threshold) for quick flips
5. Drag from any point on page (not just edges)
6. **Checkpoint**: Drag a page corner to partially flip, release to complete or snap back

## Phase 5: HTML-to-Texture Pipeline
1. `TextureLoader.js` — Accept HTML string, render to hidden DOM element
2. Use html2canvas to capture as canvas
3. Create Three.js CanvasTexture, apply to page front/back
4. Texture atlas or individual textures per page (perf tuning)
5. Support image URLs directly (skip html2canvas)
6. **Checkpoint**: Menu HTML content visible on 3D pages

## Phase 6: Public API + UMD Build
1. `OpenFlip.js` — Constructor, options parsing, lifecycle (init/destroy)
2. Public methods: next(), prev(), flipTo(n), destroy()
3. Events: onFlip, onReady, onDragStart
4. Vite library mode build → `dist/openflip.min.js` (UMD) + `dist/openflip.es.js` (ESM)
5. **Checkpoint**: `new OpenFlip('#el', { pages: [...] })` works from script tag

## Phase 7: Controls UI
1. `Controls.js` — Optional overlay: prev/next arrows, page counter, fullscreen toggle
2. Thumbnail strip (optional)
3. Zoom (camera dolly)
4. Keyboard navigation (arrow keys)
5. CSS for controls — minimal, customizable via options

## Phase 8: Polish + Integration
1. Sound effects on flip (Web Audio API)
2. Paper grain texture (SVG noise as subtle overlay)
3. Cover material (different from interior pages — darker, slight gloss)
4. Page edge thickness (thin extrusion for realism)
5. Performance: frustum culling, texture dispose for off-screen pages, LOD
6. Mobile optimization: reduce geometry segments, simpler shadows
7. Integration demo with Ismano menu data

## Phase 9: Ismano Migration
1. Copy `dist/openflip.min.js` to `ismano/ui/static/www/js/`
2. Rewrite booklet section in `tour.go` to use OpenFlip
3. `pageCallback` fetches from `/api/menu` and builds HTML pages
4. Remove page-flip@2.0.7 CDN dependency
5. Test on desktop + mobile + tablet
6. Deploy to production
