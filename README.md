# Local Grid Tiler

🚀 **[Live Demo](https://kylemath.github.io/pdfTiles)** 🚀

A single-file browser tool with two tabs, both running **entirely on your device** with no server involved at any step:

- **Tile** — tiles a PDF or image into any rows × columns grid layout and downloads the result as a PDF.
- **Weekly Flyer** — builds a printable Shabbat flyer (parasha, candle lighting, Havdalah and full davening zmanim) for any location, then exports it as PNG/PDF or hands it straight to the Tile tab.

![Grid Tiler UI](screenshot.png)

---

## What it does

Open `index.html` in any modern browser. Select a PDF or image file, set the number of rows and columns, adjust cell margins, toggle borders, choose orientation, and click **Tile & Download**. The tiled PDF lands in your downloads folder in seconds.

### Input formats

| Type | Formats |
|------|---------|
| PDF | `.pdf` — first page is tiled |
| Images | `.jpg`, `.png`, `.gif`, `.webp`, `.bmp`, `.tiff` |

JPEG files are embedded directly. All other image formats are converted to PNG in-browser before embedding.

### Grid layout

Instead of fixed presets, you set **rows** and **columns** independently — any value from 1 to 10 each, up to 100 copies per page. The default is 2 rows × 2 columns (4 copies). A live copy count updates as you type.

### Options

| Option | Default | Description |
|--------|---------|-------------|
| Rows | 2 | Number of grid rows (1–10) |
| Cols | 2 | Number of grid columns (1–10) |
| Cell inner margin | 10 px | Padding inside each grid cell |
| Draw borders | On | Draws a light gray border around each cell |
| Landscape output | Off | Rotates the output page to horizontal (792 × 612 pt) |

### Live preview

A small canvas preview appears as soon as a file is loaded and updates instantly whenever any setting changes. It shows the actual image content for image inputs and a document placeholder for PDFs.

---

## Weekly Flyer tab

Create a print-ready weekly Shabbat flyer without any internet connection or API key. All halachic times and the Torah portion are calculated on-device by [KosherZmanim](https://github.com/BehindTheMath/KosherZmanim) (the JavaScript port of [KosherJava](https://kosherjava.com/)), which is bundled locally.

### Inputs

| Field | Description |
|-------|-------------|
| Location preset | A list of common cities, or **Custom** to enter your own coordinates |
| Latitude / Longitude / Elevation | Used for the solar calculations; elevation affects sunrise/sunset |
| Time zone | IANA zone (e.g. `America/Toronto`); autocompletes where the browser supports it |
| 📍 Use my GPS location | Fills coordinates + time zone from the device (browser geolocation, no network) |
| Shabbat date | Defaults to the upcoming Saturday; candle lighting is the Friday before |
| Tradition / minhag | **Orthodox** (GRA zmanim) or **Chabad** (Baal HaTanya zmanim), or **Custom** |
| Candle lighting | **Plag HaMincha** (early Shabbat) or a set number of minutes before sunset |
| Shabbat Ends (main time) | 72 minutes / Rabbeinu Tam (default), 3 stars / Geonim 8.5°, 50 or 42 minutes, or Baal HaTanya (6°). A secondary **Chabad** time (8.5°, the Motzei-Shabbos *lechumra*) is always shown beside it |
| Use elevation | Off by default (sea-level), matching most Chabad luachs; on uses the entered elevation |
| Manual time overrides | A collapsible section with a box for every printed time (Candle Lighting, Say Shema, Sunrise, Latest Shema, Mincha, Shabbat Ends, Chabad) — leave blank to use the computed value, or type your shul's exact time verbatim |
| Israel | One-day yom tov + Israeli parasha schedule |
| Title override | Blank → auto-builds `Shabbat Parshat <name>` (or the holiday name) |
| Subtitle | Blank → auto-fills special parasha, *Shabbat Mevorchim*, or the Hebrew month |
| Shacharit time | Your shul's davening time (free text, default `9:00 AM`) |
| Lesson of the Parsha | Auto-fills a one-line, Chabad-style summary for the week's parasha (distilled from Chabad.org's *Parshah in a Nutshell*); the **✨ Use suggested summary** button copies it into the box so you can edit it, or type your own |
| Closing blessing | Default `Shabbat Shalom U'Mevorach!` |
| Show B"H | Toggles the B"H mark in the top corner |

### Layout & output

The flyer follows a classic Chabad-style template — a navy header band (B"H, parasha title, subtitle, city/date) over two gold-underlined sections:

- **Friday Evening** — Mincha (*Before Candlelighting*), Candle Lighting, Kabbalat Shabbat (*Following Candles*), Say Shema (nightfall).
- **Shabbat Day** — Sunrise, Latest Shema, Shacharit, Mincha (*Mincha Gedola → Shkia*), and **Shabbat Ends** showing the main time **plus the Chabad time** (8.5° / 3 medium stars — the documented Motzei-Shabbos *lechumra*, per R' Feigelstock the 6° weekday tzeis should **not** be used for the end of Shabbos).

A live **canvas preview** updates as you type. Then:

- **Download PNG** / **Download PDF** save the flyer to disk.
- **Send flyer to Tile tab →** loads the rendered flyer directly into the Tile tab so you can print multiple copies per page.
- A **verification table** lists every computed time with a link to [MyZmanim](https://www.myzmanim.com/) so you can cross-check before printing.
- Open the flyer directly with the `index.html#flyer` deep link.

> Times are computed from a solar model and standard minhag options. Always confirm against your community's published times before relying on them.

---

## Zero-trust local processing

This tool is designed so that your documents never leave your machine under any circumstances.

### How processing works

1. The browser reads the file you select using the standard [File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API) — the bytes go directly into JavaScript memory, not to any server.
2. [pdf-lib](https://pdf-lib.js.org/) (bundled locally, see below) tiles the content entirely in-process. For non-JPEG images, an off-screen `<canvas>` handles the format conversion before embedding.
3. The finished PDF is handed back to you via a `blob:` URL and a simulated download link — a browser-native mechanism that opens no network socket.

At no point does the tool make an upload request, POST form data, or call any external API with your file.

### Security measures in the code

**No outbound network calls from JavaScript**
A [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) meta tag is embedded in the page:
```
connect-src 'none'; object-src 'none'; base-uri 'none';
```
This instructs the browser to block any `fetch()`, `XMLHttpRequest`, `WebSocket`, or `sendBeacon()` call made by any script on the page — even if a dependency were somehow compromised.

**pdf-lib bundled locally — zero CDN calls**
The PDF processing library (`pdf-lib.min.js`, v1.17.1) is included directly in this repository. The page loads it with a relative path (`src="pdf-lib.min.js"`), so opening `index.html` makes **no outbound network requests whatsoever** — it works completely offline.

> Previously the library was loaded from unpkg.com. Before removing that dependency, a Subresource Integrity (SRI) hash was verified (`sha384-weMABwrltA6...`) to confirm the local copy is byte-for-byte identical to the published package.

**Blob URL cleanup**
After the download is triggered, `URL.revokeObjectURL()` is called to release the in-memory blob and prevent memory leaks. Preview image blob URLs are also revoked immediately after the image loads.

**No analytics or telemetry**
There are no third-party scripts, tracking pixels, analytics SDKs, or remote logging of any kind.

---

## Usage

No build step, no server, no installation required.

```bash
# Open directly in your default browser
open index.html        # macOS
start index.html       # Windows
xdg-open index.html    # Linux
```

Or just double-click `index.html` in Finder / File Explorer.

---

## Platform compatibility

| Platform | Browser | Support |
|----------|---------|---------|
| Windows | Chrome, Edge, Firefox | Full |
| macOS | Safari, Chrome, Firefox | Full |
| Android | Chrome | Full |
| iOS 13+ | Safari | Partial — PDF opens in viewer or share sheet rather than auto-saving (browser limitation, not a security issue) |
| iOS (any) | Chrome / Firefox | Partial — same WebKit behavior as Safari |

---

## Files

```
index.html            — the entire application (single file, no build)
pdf-lib.min.js        — pdf-lib v1.17.1 (bundled locally, no CDN dependency)
kosher-zmanim.min.js  — KosherZmanim v0.9.0 (bundled locally, no CDN dependency)
screenshot.png        — UI screenshot used in this README
```

---

## Dependencies

| Library | Version | Source |
|---------|---------|--------|
| [pdf-lib](https://github.com/Hopding/pdf-lib) | 1.17.1 | Bundled locally (`pdf-lib.min.js`) |
| [KosherZmanim](https://github.com/BehindTheMath/KosherZmanim) | 0.9.0 | Bundled locally (`kosher-zmanim.min.js`) |

No npm, no bundler, no build toolchain needed.
