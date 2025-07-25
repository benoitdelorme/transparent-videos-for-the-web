# Why This HTML Script Exists – Extracting PNG Frames with Alpha from a WebM Video

Transparent WebM videos — especially those encoded with **VP8 or VP9** — store transparency (alpha) **within the video stream itself**, rather than as a separate alpha channel. Because of this:

- **VP8** does **not** support a proper alpha channel at all
- Even **VP9** encodes alpha in a compressed, embedded format
- Tools like **FFmpeg cannot reliably extract frames with transparency intact** from such `.webm` files

## 🎯 The Solution

This HTML/JavaScript script uses a combination of the `<video>` and `<canvas>` elements to:

1. Load a `.webm` video with transparency
2. Read each frame at a defined frame rate (e.g., 30 fps)
3. Draw the frame on a `<canvas>` element
4. Export the result as a **PNG file with preserved alpha transparency**

## 🔄 Purpose

This workflow allows you to:
- **Reconstruct a full image sequence** from a transparent WebM
- Preserve the **original alpha channel**
- Post-process or crop the sequence using **FFmpeg** or other tools
- Export to formats like **ProRes `.mov` (Safari-compatible)** or **VP9 `.webm` (optimized for web)**

## 📦 When to Use

Use this script if:
- You have a `.webm` file with transparency
- You want to **crop, convert, or edit** it while preserving alpha
- You need **high-quality per-frame export** or cross-browser compatibility


# FFmpeg Guide: From PNG Sequence to MOV and WebM (with Cropping)

This guide outlines how to use FFmpeg to:

1. Convert a sequence of transparent PNG images to a `.mov` file with ProRes 4444
2. Crop the resulting `.mov` with transparency preserved
3. Convert a `.webm` to a cropped `.webm` with CRF 20 and alpha transparency

---

## 🖼️ Step 1 – Create a `.mov` (ProRes 4444) from PNG sequence

Assumes files like `frame_0000.png`, `frame_0001.png`, etc.

```bash
ffmpeg -framerate 30 -i frame_%04d.png \
  -c:v prores_ks -profile:v 4 -pix_fmt yuva444p10le \
  output.mov
```

- `prores_ks -profile:v 4`: ProRes 4444 (supports alpha)
- `yuva444p10le`: pixel format with alpha and 10-bit color

---

## ✂️ Step 2 – Crop the `.mov` file (remove 850px left/right, 20px bottom)

Assuming input is 3840×2160:

```bash
ffmpeg -i output.mov \
  -vf "crop=2140:2140:850:0,format=yuva444p10le" \
  -c:v prores_ks -profile:v 4 -pix_fmt yuva444p10le \
  output_cropped.mov
```

- Crops 850px from both sides and 20px from the bottom (height = 2160 - 20)
- Final dimensions = **2140×2140**

---

## 🌐 Step 4  - Convert .mov to HEVC
- Open video .mov with Quicktime
- Export as 1080p
- Keep transparency
- Select HEVC

## 🌐 Step 3 – Convert `.webm` to cropped `.webm` with CRF 20 (VP9 + alpha)

This example applies the same crop as above (850px left/right, 20px bottom), with light compression.

```bash
ffmpeg -i input.webm \
  -vf "crop=2140:2140:850:0,format=yuva420p" \
  -c:v libvpx-vp9 -pix_fmt yuva420p -crf 20 -b:v 0 \
  output_cropped.webm
```

- `libvpx-vp9`: required for alpha support in WebM
- `yuva420p`: pixel format supporting alpha
- `-crf 20`: visually high quality, much smaller file than CRF 0
- `-b:v 0`: disables default bitrate constraints

---

## ✅ Notes

- **Safari only supports `.mov` (ProRes 4444)** for transparency
- **Chrome/Firefox support `.webm` (VP9 + alpha)** — but not Safari
- Use both formats for full browser compatibility
- Resource : https://www.youtube.com/watch?v=Xg6aYfuvRHk&t=51s&ab_channel=YuriArtiukh


