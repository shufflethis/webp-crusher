# webp-crusher

> Browser-based batch image compressor. Convert up to 50 images to **WebP**, force every file **under a target size** (default 100 KB), and download the result as a ZIP — **without uploading a single byte**.

[![Live demo](https://img.shields.io/badge/live%20demo-webp--crusher.vercel.app-00e68a)](https://webp-crusher.vercel.app/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Build: none](https://img.shields.io/badge/build-zero--config-brightgreen)](#quick-start)
[![Privacy: client-side](https://img.shields.io/badge/privacy-100%25%20client--side-00e68a)](#privacy)
[![Deploy: Vercel](https://img.shields.io/badge/deploy-Vercel%20%2F%20VPS%20%2F%20local-black)](#deployment)
[![Ko-fi](https://img.shields.io/badge/support-Ko--fi-ff5e5b)](https://ko-fi.com/a2amcp)

**Live demo: [webp-crusher.vercel.app](https://webp-crusher.vercel.app/)** &nbsp;·&nbsp; UI is currently in German.

## What this is

webp-crusher is a single-file static web app that batch-converts images to WebP and compresses each one to a size budget you set. It is built for **SEO and web-performance work**, where image weight is one of the most common and most fixable problems.

SEO crawlers and performance tools — Screaming Frog, Lighthouse, PageSpeed Insights, GTmetrix — routinely flag images that are too heavy for mobile. The usual advice is "serve next-gen formats" and "keep images small." webp-crusher does exactly that, in bulk, in seconds, with no upload step and no per-image fiddling.

There is no backend. The entire app is one HTML file (`public/index.html`). All decoding, resizing, and WebP encoding happen in the visitor's browser via the Canvas API. You can run it on your laptop, drop it on a VPS, or deploy it to Vercel — the processing model is identical either way.

## Why use it

- **SEO / GEO bulk tasks** — convert a whole folder of product shots, blog images, or location-page photos in one pass and keep every file under the size threshold Screaming Frog and similar tools recommend.
- **Core Web Vitals** — smaller images mean faster LCP and less layout shift on mobile.
- **No upload, no account, no quota** — unlike online compressors, nothing leaves the browser. Safe for client material, unpublished assets, and anything under NDA.
- **Predictable output** — you set the size budget; the app guarantees it (or scales the image down until it fits).
- **Zero install for end users** — share a URL; the tool runs anywhere a modern browser does.

## Features

- **Batch conversion** — up to 50 images per run.
- **Input formats** — JPG, PNG, GIF, BMP, TIFF, and WebP.
- **Output** — WebP, every file forced under your target size.
- **Drag & drop or click** to add files; thumbnails and per-file stats shown live.
- **Configurable size budget** — default 100 KB, adjustable from 10 to 1000 KB.
- **Configurable max width** — default 1920 px, adjustable from 200 to 4000 px; aspect ratio preserved.
- **Optional filename prefix** — useful for naming conventions and cache-busting.
- **ZIP download** — all converted images bundled via [JSZip](https://stuk.github.io/jszip/).
- **Live savings report** — total original size, compressed size, and percentage saved.
- **Zero build, zero dependencies to install** — one HTML file; JSZip is loaded from a CDN.

## How it works

webp-crusher hits your size target with a two-stage strategy, all running on an HTML `<canvas>`:

1. **Resize first.** If an image is wider than the configured max width, it is scaled down (aspect ratio preserved) before any encoding. Most oversized images shrink enough at this step alone.
2. **Binary search on quality.** The app encodes to WebP and, if the result still exceeds the budget, runs a binary search across the WebP quality range (≈0.05–0.92, up to 12 iterations). It keeps the highest quality that still fits under the limit.
3. **Progressive downscale fallback.** If even the lowest practical quality is too large, the image is scaled down in 10% steps until it fits — so the size budget is honored even for difficult inputs.

This ordering matters: resizing before re-encoding preserves far more visual quality than crushing quality alone, and the binary search avoids the "guess a quality number and hope" problem of most converters.

## Quick start

The app is a static site. Clone it and serve the `public/` directory:

```bash
git clone https://github.com/shufflethis/webp-crusher.git
cd webp-crusher
npm run dev          # runs: npx serve public
```

Then open the printed URL (typically `http://localhost:3000`).

No build step exists — `npm run build` is intentionally a no-op. There is nothing to compile.

## Usage

1. Open the app in a browser.
2. (Optional) Adjust **Max file size (KB)**, **Max width (px)**, and **Prefix**.
3. Drag images onto the drop zone, or click to pick files (max 50).
4. Click **Konvertieren** (Convert). Each file shows a spinner, then ✅ with its new size and the percentage saved.
5. Click **ZIP Download** to get `webp-images-<timestamp>.zip`, containing a `webp-images/` folder with every converted file.

| Setting | Default | Range | Notes |
|---------|---------|-------|-------|
| Max file size (KB) | 100 | 10–1000 | Hard ceiling. Every output file is forced under this. |
| Max width (px) | 1920 | 200–4000 | Images wider than this are scaled down; aspect ratio kept. |
| Prefix | _(empty)_ | — | Optional. Output is named `prefix_originalname.webp`. |

> **Note:** The current UI is in German. Functionality is language-independent; an English UI is a welcome contribution (see [Getting involved](#getting-involved)).

## Privacy

webp-crusher does **not upload your images**. There is no server-side processing, no storage, and no analytics on the images themselves.

- Files are read with the browser `File` API and processed entirely in memory.
- Conversion happens on a local `<canvas>` element.
- The ZIP is assembled in the browser and offered as a direct download.
- The only external request is loading the JSZip library from a CDN. To run fully offline, vendor JSZip locally (see [Self-hosting JSZip](#self-hosting-jszip)).

This makes the tool safe for confidential, unpublished, or client-owned assets.

## Deployment

webp-crusher is a static site and runs anywhere static files can be served.

### Vercel

The repo ships a `vercel.json` configured for a zero-build static deploy (output directory `public/`, one-hour `Cache-Control`). The [live demo](https://webp-crusher.vercel.app/) runs exactly this setup. Import the repo in Vercel, or:

```bash
npm i -g vercel
vercel
```

### VPS or any static host

Copy `public/` behind any web server — nginx, Caddy, Apache, or a one-liner:

```bash
# Python
cd public && python3 -m http.server 8080

# Node
npx serve public
```

Serving over HTTPS is recommended so the `Cache-Control` header and modern browser features behave consistently.

### Self-hosting JSZip

To remove the CDN dependency (for offline use or stricter CSP), download [`jszip.min.js`](https://stuk.github.io/jszip/) into `public/` and change the script tag in `public/index.html`:

```html
<!-- from -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<!-- to -->
<script src="jszip.min.js"></script>
```

## Browser support

Requires a browser with `canvas.toBlob()` WebP encoding support — all current versions of Chrome, Edge, Firefox, and Safari (16.4+). WebP encoding via Canvas is the one hard requirement; older Safari releases will fail to encode.

## Limitations

- **50 images per batch.** Additional files beyond 50 are ignored with a notice. Run multiple batches for larger sets.
- **Memory-bound.** Very large images (e.g. 6000 px+ source files) are decoded fully into memory; extremely large batches on low-RAM devices may be slow.
- **No EXIF orientation handling.** Images relying on EXIF rotation may be encoded in their stored orientation.
- **No metadata preservation.** EXIF, ICC profiles, and color metadata are dropped during canvas re-encoding — expected behavior for web-delivery images, but not suitable for archival.
- **WebP only.** AVIF output is not supported.

## Repository map

```text
public/
  index.html        The entire application — UI, styles, and conversion logic.
vercel.json         Zero-build static deploy config (output dir + cache headers).
package.json        Dev script (npx serve) and no-op build script.
.gitignore          node_modules, .vercel, .DS_Store.
LICENSE             MIT.
```

## Keywords

WebP converter, batch image compressor, bulk WebP, image compression, client-side image converter, no-upload image compressor, SEO image optimization, GEO image optimization, Core Web Vitals, LCP optimization, mobile image optimization, Screaming Frog images, compress images under 100 KB, JPG to WebP, PNG to WebP, static web app, zero-build, browser image conversion.

## Getting involved

Useful contributions include:

- An English (or multilingual) UI.
- AVIF output support.
- EXIF orientation handling.
- A self-hosted JSZip build by default.
- Per-image quality overrides.

Open an [issue](https://github.com/shufflethis/webp-crusher/issues) or start a [discussion](https://github.com/shufflethis/webp-crusher/discussions).

Contributions and project support: [Ko-fi](https://ko-fi.com/a2amcp). Direct contact is via [X DM @wuebbe](https://x.com/wuebbe); no public email is published.

## License

[MIT](LICENSE) © 2026 Gorden Wuebbe / webp-crusher
