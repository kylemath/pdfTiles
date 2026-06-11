# Local PDF Grid Tiler

A single-file browser tool that tiles one page of a PDF into a 2-, 4-, 9-, or 16-up grid layout and downloads the result — entirely on your device, with no server involved at any step.

![PDF Grid Tiler UI](screenshot.png)

---

## What it does

Open `index.html` in any modern browser. Select a PDF, choose a grid layout (e.g. 4-Up prints four copies of the page on one sheet), adjust cell margins, toggle borders, and click **Tile PDF & Download**. The tiled PDF lands in your downloads folder in seconds.

**Supported layouts**

| Option | Arrangement |
|--------|-------------|
| 2-Up   | 1 row × 2 columns |
| 4-Up   | 2 rows × 2 columns |
| 9-Up   | 3 rows × 3 columns |
| 16-Up  | 4 rows × 4 columns |

---

## Zero-trust local processing

This tool is designed so that your documents never leave your machine under any circumstances.

### How processing works

1. The browser reads the file you select using the standard [File API](https://developer.mozilla.org/en-US/docs/Web/API/File_API) — the bytes go directly into JavaScript memory, not to any server.
2. [pdf-lib](https://pdf-lib.js.org/) (bundled locally, see below) tiles the page entirely in-process.
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
After the download is triggered, `URL.revokeObjectURL()` is called to release the in-memory blob and prevent memory leaks.

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
index.html       — the entire application (single file, no build)
pdf-lib.min.js   — pdf-lib v1.17.1 (bundled locally, no CDN dependency)
screenshot.png   — UI screenshot used in this README
```

---

## Dependencies

| Library | Version | Source |
|---------|---------|--------|
| [pdf-lib](https://github.com/Hopding/pdf-lib) | 1.17.1 | Bundled locally (`pdf-lib.min.js`) |

No npm, no bundler, no build toolchain needed.
