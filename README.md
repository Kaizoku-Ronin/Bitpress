# Bitpress

**Turn any PDF into a tiny, searchable, 1-bit (CCITT G4) PDF — entirely in your browser.**

**https://kaizoku-ronin.github.io/Bitpress/**

Bitpress renders each page, thresholds it to pure black-and-white, encodes it with CCITT Group 4 (the fax codec), and optionally lays an invisible OCR text layer behind it so the result stays selectable and searchable. A 25 MB "Print to PDF" scan can drop to a couple of MB while coming out *sharper*.

No install, no server, no upload. One HTML file.

---

## Why

Documents that come out of "Microsoft Print to PDF" drivers and scanners are often enormous — each page is megabytes of traced vector paths or full-color raster, frequently with no real text. That's a problem when you need to fit under a portal's file-size limit, the pages aren't searchable, and a normal "compress PDF" tool finds no embedded images to shrink.

For clean black-on-white documents — invoices, forms, shipping paperwork, scanned packets — 1-bit **CCITT Group 4** is the right tool. It's typically **5–10× smaller than grayscale JPEG and sharper**, because text and line art are inherently two-tone. Bitpress makes that one-click, and adds an OCR layer so the shrunk file is still searchable.

## Features

- **True CCITT G4 encoding** — a from-scratch ITU-T T.6 encoder, validated pixel-perfect against two independent decoders (libtiff and poppler).
- **Searchable output** — optional OCR (Tesseract) builds an invisible text layer aligned to the image, so you can select and search the text.
- **1-bit Flate fallback** — one click to a guaranteed-valid 1-bit `FlateDecode` PDF if you ever want it.
- **Resolution, threshold, and language controls** — 150–400 dpi, Otsu auto-threshold or manual, English/Italian/German/French/Spanish (and combos).
- **100% client-side** — your file never leaves the machine. Good for confidential documents.
- **Single file, no build step** — just `bitpress.html`.

## Quick start

Open `bitpress.html` in a modern browser, drop a PDF on the drop zone, pick your settings, and click **Compress PDF**. Download the result.

Or host it: commit `bitpress.html` to a GitHub repo, enable **Settings → Pages**, and open the published URL.

## Usage

| Control | What it does |
| --- | --- |
| **Resolution** | Rasterization dpi. 300 is a good default for dense tables; 150–200 for lighter docs or smaller files. |
| **Image encoding** | `CCITT G4` (default, smallest) or `1-bit Flate` (always-valid fallback). |
| **Threshold** | `Auto (Otsu)` for clean black-on-white; `Manual` for faint or uneven scans. |
| **OCR text layer** | `On` adds searchable text (slower — a few seconds per page). `Off` is a pure image-only shrink. Pick the language(s) to match the document. |

The output file is named `<original>_1bit_g4_ocr.pdf` (suffixes reflect the options you chose).

## How it works

1. **Render** — pdf.js draws each page to a canvas at the chosen dpi.
2. **Threshold** — luminance is reduced to 1 bit per pixel (Otsu or manual cutoff).
3. **OCR** *(optional)* — Tesseract reads the page and returns word boxes.
4. **Encode** — each bitmap is compressed with the hand-rolled CCITT G4 encoder (or Flate).
5. **Assemble** — a PDF is built at the byte level: a `CCITTFaxDecode` image XObject per page, plus an invisible (`Tr 3`) text layer positioned from the OCR boxes.

The CCITT G4 encoder and PDF writer are dependency-free; only rendering and OCR use libraries.

## Self-hosting (offline / locked-down networks)

By default the app loads three libraries from public CDNs at runtime — pdf.js, Tesseract.js (plus its worker, WASM core, and `*.traineddata` language files), and pako. If your network blocks CDNs:

1. Download the assets and host them alongside `bitpress.html`.
2. Edit the `CDN` block at the top of the `<script>` in `bitpress.html` to point at your copies:

```js
const CDN = {
  pdfjs:       "...pdf.min.js",
  pdfjsWorker: "...pdf.worker.min.js",
  pako:        "...pako.min.js",
  tesseract:   "...tesseract.min.js",
  tessWorker:  "",   // path to tesseract worker.min.js
  tessCore:    "",   // folder containing tesseract-core*.wasm.js
  tessLang:    ""    // folder containing eng.traineddata.gz, etc.
};
```

If the OCR assets are unavailable, Bitpress still performs the 1-bit / G4 shrink — it just skips the text layer.

## Privacy

Everything runs locally in the browser. No PDF, image, or text is uploaded anywhere. The only network requests are for the libraries above (which you can self-host to make the tool fully offline).

## Browser support

Any current Chromium (Chrome/Edge), Firefox, or Safari. Large or high-dpi documents need a desktop browser with enough memory; OCR is the slow step and scales with page count and resolution.

## Limitations

- Best for **black-on-white** content (text, tables, line art). Photographs and heavy gradients are not a good fit for 1-bit output.
- OCR accuracy depends on scan quality and the selected language; the text layer is for search/selection, not a perfect transcript.
- Output is image-based (plus a text layer); the original vector text, if any, is not preserved.

## License

[MIT](LICENSE).
