# PhantomLens 🔍

**Local image forensics toolkit — a combined FotoForensics + Hintfo replacement.**

PhantomLens gives you professional-grade image forensic analysis entirely on your own machine. No uploads to third-party servers. No rate limits. No accounts. Comes in two flavors: a Python CLI and a self-contained browser web app you can host on GitHub Pages.

---

## Features

| Analysis | CLI | Web App |
|---|---|---|
| Error Level Analysis (ELA) | ✅ | ✅ with comparison slider |
| EXIF / XMP / IPTC / GPS Metadata | ✅ | ✅ with sub-tabs |
| MD5 / SHA-1 / SHA-256 / SHA-512 digests | ✅ | ✅ with copy buttons |
| ASCII string extraction | ✅ | ✅ with search/filter |
| Hidden pixel detection (alpha channel) | ✅ | ✅ with visualization |
| JPEG quality estimation | ✅ | ✅ with heat-mapped tables |
| ICC color profile parsing | ✅ | ✅ |
| URL input | ✅ | ✅ (via CORS proxy) |
| File input | ✅ | ✅ drag & drop or picker |
| JSON export | ✅ `--json` flag | ✅ Export button |

---

## Python CLI (`phantomlens.py`)

### Requirements

```bash
pip install Pillow
```

**Optional (enable richer output):**

```bash
pip install piexif rich       # Better EXIF and styled terminal output
# Also: install exiftool system-wide for extended metadata
# macOS:  brew install exiftool
# Ubuntu: apt install libimage-exiftool-perl
```

### Usage

```bash
# Analyze a local file (all analyses)
python phantomlens.py image.jpg

# Analyze an image from a URL
python phantomlens.py https://example.com/photo.jpg

# Run specific analyses only
python phantomlens.py image.jpg --analyses ela metadata digest

# Save ELA output image to a specific path
python phantomlens.py image.jpg --output ela_result.png

# Adjust ELA parameters
python phantomlens.py image.jpg --ela-quality 90 --ela-scale 15

# Export all results as JSON
python phantomlens.py image.jpg --json > results.json

# Quiet mode (suppress progress messages)
python phantomlens.py image.jpg -q
```

### Available Analyses

| Flag name | Description |
|---|---|
| `ela` | Error Level Analysis — resaves at Q=95, amplifies diff |
| `metadata` | EXIF, XMP, GPS, ICC via PIL + optional ExifTool |
| `digest` | MD5, SHA-1, SHA-256, SHA-512 of raw file bytes |
| `strings` | ASCII string extraction, categorized (URLs, dates, paths…) |
| `hidden` | Alpha channel scan for hidden pixels |
| `quality` | JPEG quantization table quality estimation |
| `icc` | ICC color profile header + tag summary |

Pass multiple: `--analyses ela digest metadata`

### CLI Options

```
positional:
  image               Path to image file OR http(s):// URL

optional:
  -a, --analyses      Space-separated list of analyses to run (default: all)
  -o, --output        Output path for ELA PNG (default: <input>_ela.png)
  --ela-quality INT   JPEG quality for ELA resave (default: 95)
  --ela-scale INT     ELA amplification multiplier (default: 10)
  --min-string INT    Minimum string length for extraction (default: 6)
  --json              Output results as JSON to stdout
  -q, --quiet         Suppress progress/status output
```

---

## Browser Web App (`index.html`)

A fully self-contained single HTML file. No server required. Open it directly in your browser, or host it anywhere static files are served.

### Running Locally

```bash
# Just open it
open index.html           # macOS
xdg-open index.html       # Linux
start index.html          # Windows
```

Or serve it locally to avoid some browser file-access restrictions:

```bash
python -m http.server 8080
# then open http://localhost:8080
```

### GitHub Pages Deployment

1. Fork or clone this repo
2. Go to **Settings → Pages**
3. Set source to `main` branch, `/ (root)` folder
4. Your app will be live at `https://<username>.github.io/<repo>/`

No build step. No dependencies to install. The HTML file loads all libraries from CDN at runtime.

### URL Input / Privacy Note

When analyzing an image by URL, the web app routes the request through [corsproxy.io](https://corsproxy.io) to work around browser CORS restrictions. **The image URL is visible to that proxy service.** For sensitive work, download the image first and use file upload instead — all file-based analysis is 100% local/client-side.

### Supported Formats

JPEG, PNG, WebP, GIF, BMP, TIFF, HEIC (browser-dependent). ELA is most meaningful for JPEG images. JPEG quality estimation only applies to JPEG files.

---

## Understanding the Results

### Error Level Analysis (ELA)

ELA reveals areas of an image that have been re-saved at a different compression level than the surrounding regions, which can indicate editing or compositing.

- **Bright/high-contrast areas** in the ELA output → high error level → potentially edited or added content
- **Uniformly dark/flat areas** → low error level → consistent compression history, likely unmodified
- **Edges and fine details** always produce some ELA signal — this is normal
- **Solid colors and gradients** typically appear very dark in ELA

Caveats: ELA is most useful on JPEGs. Screenshots, images downloaded from social media, and files that have been re-compressed many times may produce misleading results.

### JPEG Quality Estimation

Quality is estimated by comparing the file's quantization tables against the IJG (Independent JPEG Group) standard reference tables.

| Estimated Quality | Interpretation |
|---|---|
| 95–100% | Near-lossless; original or very lightly compressed |
| 80–94% | Typical high-quality save; likely unmodified or once-resaved |
| 60–79% | Moderate compression; possibly processed or web-optimized |
| < 60% | Heavy compression; multiple resaves likely; ELA less reliable |

### Hidden Pixels

Detects non-zero RGB data in fully transparent pixels (alpha = 0). This is common in PNG files that have had their transparency masked but retain original pixel color values. May indicate cropping, compositing, or steganographic content.

### String Extraction

ASCII strings embedded in binary image data. Categories of interest:

- **Software** — reveals editing tools (Photoshop, GIMP, Lightroom, iPhone, etc.)
- **Dates** — may differ from EXIF dates if metadata was stripped/modified
- **URLs** — can reveal original source, CDN hosts, or linked resources
- **GPS** — raw coordinate strings that may survive metadata stripping
- **Paths** — file system paths from the original editing workstation

### Digest / Hashes

Use digests to verify file integrity, detect duplicates, or track a specific version of a file across sources. If two copies of an "original" image have different SHA-256 hashes, at least one has been modified.

---

## File Structure

```
phantomlens/
├── index.html        # Self-contained browser web app
├── phantomlens.py    # Python CLI tool
└── README.md         # This file
```

---

## Privacy

- **Python CLI:** Entirely local. URL input uses `urllib` (standard library) — no third-party routing.
- **Web app (file upload):** 100% client-side. Nothing leaves your browser.
- **Web app (URL input):** Routes through corsproxy.io. See note above.

---

## Credits & Inspiration

- [FotoForensics](https://fotoforensics.com) by Neal Krawetz — ELA methodology and forensic UI concepts
- [Hintfo](https://hintfo.com) — comprehensive metadata extraction approach
- [Jeffrey's Exif Viewer](https://exif.regex.info/exif.cgi) — metadata display inspiration
- [exifr](https://github.com/MikeKovarik/exifr) — client-side EXIF parsing library
- [SparkMD5](https://github.com/satazor/js-spark-md5) — client-side MD5

---

*PhantomLens — forensics that stays on your machine.*
