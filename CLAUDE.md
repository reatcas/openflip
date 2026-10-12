# OpenFlip — WebGL 3D Flipbook

## Project Overview
Open-source WebGL flipbook library built from scratch with Three.js. Physics-based page turns with realistic cylindrical bend deformation via custom GLSL vertex shaders. Zero jQuery dependency, pure ES modules.

## Target Quality
Premium restaurant menu booklet — must look as good as commercial solutions (Real 3D FlipBook level). The primary consumer is Ismano Gastrobar (reatcas/ismano) but the library should be generic and reusable.

## Architecture

```
openflip/
├── src/
│   ├── OpenFlip.js           # Main entry — public API, creates Scene+Book+Controls
│   ├── Scene.js              # Three.js scene, WebGLRenderer, camera, lights, shadows
│   ├── Book.js               # Book model: manages pages array, spine, covers, flip queue
│   ├── Page.js               # Single page: PlaneGeometry (32×48 segments), double-sided
│   ├── PageMaterial.js       # ShaderMaterial with custom vertex (bend) + fragment (paper)
│   ├── shaders/
│   │   ├── page.vert.glsl    # Cylindrical bend deformation — variable radius arc
│   │   └── page.frag.glsl    # Paper material — texture + grain + soft shadow
│   ├── FlipAnimation.js      # Spring-damper physics for page flip motion
│   ├── Interaction.js        # Mouse/touch → drag-to-flip, swipe detection
│   ├── TextureLoader.js      # HTML→Canvas→Texture pipeline (html2canvas or OffscreenCanvas)
│   └── Controls.js           # Optional UI: prev/next, thumbnails, fullscreen, zoom
├── dist/
│   └── openflip.min.js       # UMD bundle for CDN/script-tag use
├── demo/
│   ├── index.html            # Basic demo with sample pages
│   └── menu-demo.html        # Ismano menu integration demo
├── package.json
├── vite.config.js
├── LICENSE                   # MIT
└── README.md
```

## Key Technical Decisions
- **Three.js**: Latest stable (r168+), imported as ES module
- **Bundler**: Vite for dev, Vite library mode for production UMD build
- **No jQuery**: Pure vanilla JS, no external dependencies except Three.js
- **Page geometry**: PlaneGeometry with 32×48 subdivisions for smooth bend
- **Vertex shader**: Cylindrical bend with variable radius (radius grows from spine to free edge for natural paper feel). Standard CG technique — NOT copied from any commercial product.
- **Animation**: Spring-damper system (configurable stiffness + damping) for physically plausible page momentum
- **Shadows**: Three.js shadow maps (SpotLight casting onto pages)
- **HTML-to-texture**: html2canvas v1.4+ renders DOM elements to canvas, used as Three.js CanvasTexture
- **Interaction**: Pointer events (unified mouse+touch), drag from page edge or center to flip

## Public API (target)
```javascript
const flip = new OpenFlip('#container', {
  width: 800,
  height: 1131,           // A4 ratio
  pages: [
    { type: 'html', content: '<div>...</div>' },
    { type: 'image', src: 'page1.jpg' },
    { type: 'callback', render: (canvas, pageNum) => { ... } }
  ],
  // OR
  pageCallback: (n) => ({ type: 'html', content: htmlString }),
  pageCount: 20,

  // Visual
  lighting: 'soft',        // 'soft' | 'dramatic' | 'flat'
  paperColor: '#FAFAF7',
  coverColor: '#3D3845',
  shadows: true,
  paperGrain: true,

  // Physics
  flipDuration: 800,       // ms
  stiffness: 0.08,
  damping: 0.85,

  // Sounds
  sounds: { flip: 'flip.mp3' },

  // Events
  onFlip: (page) => {},
  onReady: () => {}
});

flip.flipTo(5);
flip.next();
flip.prev();
flip.destroy();
```

## Vertex Shader Approach (from-scratch, standard CG)
The page bend uses a well-known cylindrical deformation technique:
1. Define a fold axis (line where the page bends)
2. For each vertex, compute distance from fold axis
3. Map that distance onto a cylindrical arc with configurable radius
4. The radius can grow with distance (tighter at spine, looser at free edge)
5. Transform vertex position and normal accordingly
6. Uniforms: bendForce, foldAngle, bendRadius, pageWidth, pageHeight

This is standard deformable surface mathematics, published in multiple CG papers and tutorials.

## Build Commands
- `npm run dev` — Vite dev server with hot reload on :5173
- `npm run build` — Production build to dist/
- `npm run preview` — Preview production build

## Integration with Ismano
Once complete, the built `dist/openflip.min.js` gets copied to `ismano/ui/static/www/js/openflip.min.js` and replaces the page-flip@2.0.7 usage in `tour.go`. The menu overlay booklet initialization changes from `new St.PageFlip(...)` to `new OpenFlip(...)` with pageCallback fetching from `/api/menu`.
