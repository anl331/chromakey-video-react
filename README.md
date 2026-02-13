# chromakey-video-react

[![npm version](https://img.shields.io/npm/v/chromakey-video-react.svg)](https://www.npmjs.com/package/chromakey-video-react)
[![license](https://img.shields.io/npm/l/chromakey-video-react.svg)](https://github.com/anl331/chromakey-video-react/blob/main/LICENSE)
[![bundle size](https://img.shields.io/bundlephobia/minzip/chromakey-video-react)](https://bundlephobia.com/package/chromakey-video-react)

> **Real-time chroma key video in React. No FFmpeg, no pre-processing, runs on the GPU.**

Drop a green-screen video into your React app and get instant transparency — rendered frame-by-frame on the GPU with a single WebGL shader. No server, no WASM, no waiting.

## Install

```bash
npm install chromakey-video-react
```

## Quick Start

```tsx
import { ChromaKeyVideo } from 'chromakey-video-react';

function App() {
  return (
    <ChromaKeyVideo
      src="/presenter-greenscreen.mp4"
      className="w-[400px]"
    />
  );
}
```

That's it. The green background is gone.

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `src` | `string` | *required* | Video source URL |
| `color` | `string` | `"#00ff00"` | Hex color to key out (e.g. `"#00ff00"` for green, `"#0000ff"` for blue) |
| `similarity` | `number` | `0.35` | How aggressively to match the key color (0–1). Higher = more removal. |
| `blend` | `number` | `0.15` | Soft edge blending range (0–1). Higher = softer edges. |
| `despill` | `boolean` | `true` | Remove color spill/tint from edges of the subject |
| `loop` | `boolean` | `true` | Loop the video |
| `autoPlay` | `boolean` | `true` | Auto-play the video (always muted for browser autoplay policy) |
| `className` | `string` | — | CSS class applied to the `<canvas>` element |

## How It Works

The component renders a hidden `<video>` element and draws each frame to a WebGL `<canvas>` through a custom fragment shader. The shader uses a **dominance-based algorithm** rather than simple RGB distance:

1. **Identify the dominant channel** of the key color (green for `#00ff00`)
2. **Measure excess** — how much the dominant channel exceeds the average of the other two
3. **Calculate dominance ratio** — `excess / (dominant + ε)`
4. **Threshold with soft blend** — pixels above the similarity threshold become transparent, with smooth blending at the edges
5. **Despill** — reduce color tint on semi-transparent edge pixels so subjects don't glow green

This approach is far more accurate than Euclidean RGB distance because it measures *how green* a pixel is relative to itself, not how close it is to pure green. Dark greens, light greens, and shadowed greens all get caught.

## Why Not FFmpeg / WebM with Alpha?

| | This package | FFmpeg pre-processing | WebM alpha |
|---|---|---|---|
| **Setup** | `npm install`, done | Need FFmpeg pipeline, server or build step | Need to re-encode video |
| **File size** | Regular MP4 | Same or larger | 2–5× larger (alpha channel) |
| **Browser support** | All (WebGL) | N/A | Chrome/Firefox only (no Safari) |
| **Runtime cost** | GPU shader, ~0ms CPU | None (pre-processed) | Decode cost for larger file |
| **Flexibility** | Adjust threshold live | Re-encode to change | Re-encode to change |
| **Any key color** | ✅ Change via prop | Need different preset per color | Baked in |

The sweet spot: you keep your regular MP4 files (small, compatible everywhere), and the GPU does the keying at 60fps. No build step, no server, no Safari compatibility headaches.

## Advanced Usage

### Blue Screen

```tsx
<ChromaKeyVideo src="/blue-screen.mp4" color="#0000ff" />
```

### Fine-tuning

```tsx
<ChromaKeyVideo
  src="/tricky-footage.mp4"
  similarity={0.4}  // more aggressive removal
  blend={0.2}       // softer edges
  despill={true}    // clean up green fringing
/>
```

## Browser Support

Works in all browsers that support WebGL (97%+ global coverage). Video must be served with proper CORS headers if cross-origin.

## Read More

📖 [How I built real-time green screen removal in the browser](https://x.com/_itsanl/status/2022425470474625404)

## Credits

Built by [@_itsanl](https://x.com/_itsanl) (Alfredo Natal) while building [@useTCT](https://x.com/useTCT) — [Trading Confluence Tool](https://tradingconfluencetool.com).

## License

[MIT](./LICENSE)
