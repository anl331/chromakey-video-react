# chromakey-video-react

[![npm version](https://img.shields.io/npm/v/chromakey-video-react.svg)](https://www.npmjs.com/package/chromakey-video-react)
[![license](https://img.shields.io/npm/l/chromakey-video-react.svg)](https://github.com/anl331/chromakey-video-react/blob/main/LICENSE)
[![bundle size](https://img.shields.io/bundlephobia/minzip/chromakey-video-react)](https://bundlephobia.com/package/chromakey-video-react)

> **Real-time chroma key video in React. No FFmpeg, no pre-processing, runs on the GPU.**

Drop a green-screen video into your React app and get instant transparency, rendered frame-by-frame on the GPU with a single WebGL shader. No server, no WASM, no waiting.

**[Try the playground →](https://anl331.github.io/chromakey-playground/)**

<p align="center">
  <img src="assets/demo.gif" alt="Real-time green screen removal demo" width="600" />
</p>

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

## The Story Behind This

I needed a 3D animated mascot for [my product's website](https://usetct.io). No budget for a designer, no patience for Blender. So I generated the character image with AI ([Higgsfield.ai](https://higgsfield.ai)), then animated it into a 5-second looping video using Kling 2.5. The trick was rendering it on a solid green background so I could remove it later.

This is what the raw video looks like:

<p align="center">
  <img src="assets/mascot-greenscreen.gif" alt="Animated mascot with green screen background" width="400" />
</p>

Now I needed to remove that green background. The obvious route: run FFmpeg with a chroma key filter, export a WebM with alpha transparency. Tried it. The results were rough. Green bleeding around the edges of the character, and WebM with alpha doesn't even work on Safari. File sizes balloon when you add a transparency channel.

So I wrote a WebGL fragment shader that does it in real-time, in the browser, on the GPU. It plays the original MP4 through a hidden video element, checks every pixel for "is this green?", and sets matching pixels to transparent. The result renders on a canvas with a transparent background. Cleaner output, works everywhere, and you can tweak the thresholds without re-encoding anything.

Here's what the final result looks like, green background completely gone, running live in the browser:

<p align="center">
  <img src="assets/demo.gif" alt="Real-time green screen removal result" width="600" />
</p>

## Why Not FFmpeg / WebM with Alpha?

| | This package | FFmpeg pre-processing | WebM alpha |
|---|---|---|---|
| **Setup** | `npm install`, done | Need FFmpeg pipeline, server or build step | Need to re-encode video |
| **File size** | Regular MP4 | Same or larger | 2-5x larger (alpha channel) |
| **Browser support** | All (WebGL) | N/A | Chrome/Firefox only (no Safari) |
| **Runtime cost** | GPU shader, ~0ms CPU | None (pre-processed) | Decode cost for larger file |
| **Flexibility** | Adjust threshold live | Re-encode to change | Re-encode to change |
| **Any key color** | Change via prop | Different preset per color | Baked in |

The sweet spot: keep your regular MP4 files (small, compatible everywhere), and the GPU does the keying at 60fps.

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `src` | `string` | *required* | Video source URL |
| `color` | `string` | `"#00ff00"` | Hex color to key out (e.g. `"#00ff00"` for green, `"#0000ff"` for blue) |
| `similarity` | `number` | `0.35` | How aggressively to match the key color (0-1). Higher = more removal. |
| `blend` | `number` | `0.15` | Soft edge blending range (0-1). Higher = softer edges. |
| `despill` | `boolean` | `true` | Remove color spill/tint from edges of the subject |
| `loop` | `boolean` | `true` | Loop the video |
| `autoPlay` | `boolean` | `true` | Auto-play the video (always muted for browser autoplay policy) |
| `className` | `string` | — | CSS class applied to the `<canvas>` element |

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
  despill={true}    // clean up color fringing
/>
```

## Tips

- **Background color matters.** Use a color far from anything in your subject. Green (`#00ff00`) is standard. If your subject has green, use magenta (`#FF00FF`).
- **Keep everything opaque.** Semi-transparent elements in the video (holographic effects, glass, etc.) will cause issues with chroma removal.
- **CORS headers.** If your video is cross-origin, make sure it's served with proper CORS headers or WebGL can't read the frames.

## Browser Support

Works in all browsers that support WebGL (97%+ global coverage).

## Read More

📖 [How I built real-time green screen removal in the browser](https://x.com/_itsanl/status/2022425470474625404)

## Credits

Built by [@_itsanl](https://x.com/_itsanl) (Alfredo Natal) while building [TCT](https://usetct.io), a Chrome extension that helps traders grade their setups before entering.

## License

[MIT](./LICENSE)
