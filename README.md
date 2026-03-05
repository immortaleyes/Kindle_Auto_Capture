<div align="center">

# 📚 Kindle AutoCapture

### Chrome Extension — Auto-screenshot every page of your Kindle ebook

[![Version](https://img.shields.io/badge/version-6.0.0-6c63ff?style=for-the-badge&logo=googlechrome)](https://github.com)
[![Manifest V3](https://img.shields.io/badge/Manifest-V3-22c55e?style=for-the-badge)](https://developer.chrome.com/docs/extensions/mv3/intro/)
[![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](LICENSE)
[![Browser](https://img.shields.io/badge/Chrome%20%7C%20Edge-Supported-orange?style=for-the-badge&logo=googlechrome)](https://chrome.google.com)

---

> **One click. 205 pages. All saved.**
> Opens your Kindle book, automatically advances every page, captures a clean screenshot, and stops when the last page is reached — no manual effort.

---

[Installation](#-installation) · [Usage](#-how-to-use) · [Features](#-features) · [Version History](#-version-history) · [Author](#-author)

</div>

---

## 🎯 What It Does

Kindle AutoCapture is a Chrome extension that **fully automates** capturing every page of a Kindle Cloud Reader ebook:

1. You open your Kindle book to **Page 1**
2. Click **▶ Start Auto-Capture** in the popup
3. The extension automatically:
   - Clicks the **Next Page** button (real mouse click simulation — not fake keyboard events)
   - Waits for Kindle to fully render the new page
   - Hides Kindle's own UI overlays ("Slide saved" badge etc.)
   - Takes a clean screenshot
   - Saves to `Downloads/AutoSave/`
   - Repeats for every page
4. **Stops automatically** when end of book is detected (3 consecutive identical frames)
5. Desktop notification confirms: *"📚 Book Complete! 205 pages saved"*

---

## ✨ Features

| Feature | Details |
|---------|---------|
| 🤖 **Full Auto Mode** | Advances pages and captures without any user interaction |
| 🖱️ **Real Mouse Click** | Uses native `MouseEvent` chain — works where keyboard events fail |
| 🎯 **4-Strategy Page Turn** | Button selector → Rightmost button → Iframe → Tap-to-advance |
| 🛑 **Auto End Detection** | Stops after 3 identical frames (end of book) |
| 🚫 **Overlay Hiding** | Hides Kindle UI badges before each screenshot |
| ⏱️ **Adjustable Speed** | 1s–6s delay per page slider |
| 📸 **Manual Capture** | Press **P** key or scroll to save anytime |
| 💾 **Auto-Save** | Always saves to `Downloads/AutoSave/` — no prompts |
| 🔁 **Duplicate Skip** | Perceptual hash comparison skips identical frames |
| ⚡ **Tab Injection** | Injects into all open tabs on install, startup, and navigation |

---

## 🔧 Installation

### Prerequisites
> ⚠️ **Critical first step:** Go to `chrome://settings/downloads` → turn **OFF** *"Ask where to save each file"*. Without this, every download is blocked.

### Chrome / Microsoft Edge

```
1. Download and extract the ZIP file
2. Open Chrome → chrome://extensions/
3. Enable Developer Mode (toggle, top-right)
4. Click "Load unpacked"
5. Select the kindle-autocapture/ folder
6. 📚 icon appears in toolbar
```

### Firefox
> Firefox is not recommended — Kindle Cloud Reader has known issues in Firefox.
> Use Chrome or Edge for best results.

---

## 📖 How to Use

### Auto-Capture an Entire Book

```
1. Open read.amazon.in or read.amazon.com in Chrome
2. Navigate to Page 1 of your book
3. Click the 📚 extension icon
4. Set the delay slider to 3–4 seconds (recommended for Kindle)
5. Click "▶ Start Auto-Capture"
6. Sit back — every page is captured automatically
7. Extension stops and notifies you when the last page is reached
```

### Manual Controls

| Action | How |
|--------|-----|
| Save current page | Press **`P`** key anywhere on the page |
| Auto-save on scroll | Scroll 80px+ then stop — auto-captures |
| Slide navigation | Press `←` `→` `PageDown` — captures after transition |
| Capture now | Click **📷 Capture Now** in popup |
| Stop auto mode | Click **⏹ Stop Auto-Capture** |

---

## 📁 Output Files

All screenshots save to:
```
Downloads/
└── AutoSave/
    ├── capture_0001_auto_p1_20250305_143022.png
    ├── capture_0002_auto_20250305_143045.png
    ├── capture_0003_auto_20250305_143101.png
    ├── ...
    └── capture_0205_auto_20250305_152847.png
```

**Filename format:** `capture_[INDEX]_[TRIGGER]_[TIMESTAMP].png`

---

## 🏗️ Architecture

```
kindle-autocapture/
├── manifest.json      ← Extension config (Manifest V3, v6.0.0)
├── background.js      ← Service Worker: capture logic, download, dedup, auto-stop
├── content.js         ← Injected per tab: P key, scroll, pressNextPage(), auto loop
├── popup.html         ← Extension UI
├── popup.js           ← Popup controller: auto mode controls, progress, log
└── icons/
    ├── icon16.png
    ├── icon48.png
    └── icon128.png
```

### How Tab Injection Works
Static `content_scripts` in `manifest.json` only inject into **new** tabs. This extension uses **programmatic injection** at 4 events to ensure every tab has the content script:

```
onInstalled  → inject into all currently open tabs
onStartup    → inject into all tabs on browser start
tabs.onUpdated (status=complete) → inject into newly loaded tab
tabs.onActivated → re-inject when user switches tab
```

### How Page-Turning Works (Kindle-specific)
Kindle Cloud Reader ignores `dispatchEvent(KeyboardEvent)` — it's React-based and listens internally. The extension fires a **full native MouseEvent chain** instead:

```
pointerover → mouseenter → mousemove → pointerdown → mousedown
→ pointerup → mouseup → click
```

Four strategies are tried in order:
1. **Known selectors** — `[class*="kg-next"]`, `[aria-label="Next page"]`, etc.
2. **Rightmost button** — scans all buttons, picks furthest right on screen
3. **Iframe click** — Kindle renders book in iframe; clicks inside at 80% width
4. **Tap-to-advance** — clicks at 82% viewport width (Kindle's tap zone)

---

## 🐛 Version History

### v6.0.0 — Kindle-Compatible Auto-Advance *(Current)*
> The page-turn finally works. Complete rebuild of navigation logic.

**🔴 Critical Fixes:**
- **`pressNextPage()` complete rewrite** — previous versions used `dispatchEvent(KeyboardEvent)` which Kindle's React renderer completely ignores. Now fires full native `MouseEvent` sequence (`pointerdown → mousedown → mouseup → click`) which Kindle responds to
- **4-strategy fallback chain** — tries button selectors, rightmost button, iframe click, tap-to-advance in sequence until one succeeds
- **Iframe penetration** — Kindle renders book content inside an `<iframe>`; now detects and clicks inside it

**🟡 Improvements:**
- Default delay increased to 3s (optimal for Kindle page transitions)
- `step()` loop uses `slideDelay` as the full interval between page turns
- Content script guard changed to `window.__apc_v6` (clears old v5 guard conflicts)
- Added `PING`/`PONG` message pair for tab health checks

---

### v5.0.0 — Auto-Capture Mode
> First version with auto-advance. Page turning didn't work but auto mode UI and capture pipeline were solid.

**✅ New Features:**
- **Auto-Capture mode** — `▶ Start Auto-Capture` button with stop/start
- **End-of-book detection** — stops after `MAX_DUPES=3` consecutive identical frames
- **Page delay slider** — 1s–6s configurable
- **Progress bar** in popup showing pages saved
- **`START_AUTO` / `STOP_AUTO`** message protocol between popup ↔ background ↔ content

**🔴 Known Bugs (fixed in v6):**
- `pressRight()` used `dispatchEvent(KeyboardEvent)` — ignored by Kindle
- Page never actually advanced → extension captured same page 205 times

---

### v4.0.0 — Programmatic Tab Injection
> Fixed the root cause of P key and scroll not working on existing tabs.

**🔴 Critical Fixes:**
- **Static `content_scripts` removed from manifest** — they only inject into tabs opened *after* install; already-open tabs had no listener
- **4-point programmatic injection** added: `onInstalled`, `onStartup`, `tabs.onUpdated`, `tabs.onActivated`
- **`window.__apc_v3` guard** prevents double-injection
- **`window.addEventListener` used** (not `document`) for broader event capture

**🟡 Improvements:**
- `⚡ Re-inject` button in popup for manual fix
- Content script sends `CONTENT_READY` confirmation on injection
- `TOGGLE` message now broadcasts to all content scripts
- Slide key delay: 1200ms (was 600ms)

---

### v3.0.0 — Overlay Hiding + Screenshot Quality
> Screenshots were showing Kindle's "Slide saved" green badge and white flash.

**🔴 Critical Fixes:**
- **Overlay hiding before capture** — injects temporary CSS `opacity:0` on all `position:fixed`, `[role="status"]`, `[class*="toast"]`, `[class*="saved"]` elements before screenshotting; removes immediately after
- **120ms settle delay** added after hiding overlays before `captureVisibleTab()`
- **Slide delay 600ms → 1200ms** — eliminated white flash from mid-transition captures

**🟡 Improvements:**
- Desktop notification on every save (with title, filename, trigger type)
- Screenshot format changed to PNG for lossless quality
- Added `notifications` permission to manifest (was missing; caused silent crash)

---

### v2.0.0 — Always-On Architecture
> Rebuilt from scratch. No more Start button needed.

**🔴 Critical Fixes:**
- **`setInterval` → `chrome.alarms` API** — MV3 Service Workers suspend after ~30s; `setInterval` stops firing; `chrome.alarms` survives sleep
- **State persistence** — all capture state now saved to `chrome.storage.local` on every update; SW can wake and resume correctly
- **`notifications.create('')`** — empty notification ID caused Chrome crash; fixed to `cap_0001` style unique IDs

**✅ New Features:**
- Extension is **ON by default** — no Start button needed
- **P key** triggers immediate capture
- **Scroll stop** (80px+, 1.2s debounce) triggers capture
- **Arrow / PageDown / PageUp** triggers capture after transition
- Green/red badge shows enabled/disabled state
- `isEnabled` toggle in popup

**🟡 Improvements:**
- `alarms` permission added to manifest
- Slide capture delay: 600ms
- Visual toast on page (later removed in v3 per user request)

---

### v1.3.0 — Fixed Save Location
> All saves now always go to `Downloads/AutoSave/` — folder name input removed.

**🔴 Critical Fixes:**
- Hardcoded `FOLDER = 'AutoSave'` — removes user-configurable folder name that caused confusion
- Flat filename fallback: `AutoSave_capture_XXXX.png` when subfolder creation fails
- `sanitize()` fallback changed from `'PageCaptures'` to `'AutoSave'`

**🟡 Improvements:**
- Folder name input removed from popup (no longer needed)
- Footer shows fixed save path
- Trigger label added to filename (e.g. `capture_0001_scroll_...`)

---

### v1.2.0 — Tab Resolution Fix
> Manual capture and auto-capture both completely broken — fixed root cause.

**🔴 Critical Fixes:**
- **`currentWindow: true` in SW context** — in a Service Worker, `currentWindow` is `undefined`, so `chrome.tabs.query({ active:true, currentWindow:true })` always returned `[]`; replaced with `chrome.windows.getAll({ windowTypes:['normal'] })` + find focused window
- **Popup window focus stealing** — when popup opens, it becomes the "focused" window; `lastFocusedWindow` returned the popup, not the browser tab; fixed by filtering `windowType === 'normal'` only
- **`windows` permission** added to manifest (was missing)
- **Diagnostic button** added — tests screenshot → download → subfolder step by step with full log output

**🟡 Improvements:**
- Every capture step logs explicitly to popup
- `sender.tab` used directly from content script messages (always correct)
- `captureIndex` state persisted to storage

---

### v1.1.0 — Service Worker Stability
> Timer mode stopped working after ~30 seconds.

**🔴 Critical Fixes:**
- `setInterval()` replaced with `chrome.alarms` API
- Capture state lost on SW sleep → added `chrome.storage.local` persistence
- `notifications.create('')` empty ID crash → unique IDs
- `content.js` iframe errors → added `chrome.runtime.id` guard + top-frame check
- `notifications` and `alarms` permissions added to `manifest.json`

---

### v1.0.0 — Initial Release
> Basic extension with scroll, slide, URL, timer modes and popup UI.

**✅ Features:**
- 4 capture modes: Scroll Stop, Slide Change, URL Change, Auto Timer
- PNG / JPEG format selection
- Duplicate skip toggle
- Custom folder name
- Activity log in popup
- Desktop notifications

**Known bugs** (all fixed in later versions):
- Timer stopped after ~30s (SW sleep)
- Manual capture broken (`currentWindow` bug)
- P key didn't work on existing tabs (static injection)
- Screenshots showed UI overlays
- Page turning didn't work in Kindle

---

## 🔐 Permissions

| Permission | Reason |
|-----------|--------|
| `tabs` | Read tab URL and capture visible tab |
| `activeTab` | Access focused tab |
| `downloads` | Save screenshots to Downloads/AutoSave/ |
| `scripting` | Inject content.js into pages programmatically |
| `storage` | Persist settings and capture index across sessions |
| `notifications` | Desktop notification on each save / book complete |
| `windows` | Get real browser window (not popup window) for capture |
| `host_permissions: <all_urls>` | Allow content script injection on all sites |

> 🔒 **Privacy:** No data leaves your device. No analytics. No servers. All screenshots stay in your Downloads folder.

---

## ⚠️ Known Limitations

- Works only inside **Chrome / Edge browser tabs** — cannot capture other Windows apps
- `chrome://` pages cannot be captured (browser security restriction)
- Kindle DRM may cause blank screenshots on some books — if so, try zooming in Chrome (Ctrl+scroll) before starting
- Firefox not recommended for Kindle

---

## 👨‍💻 Author

**Prof. (Dr.) Ajay Shriram Kushwaha**
Department of Computer Science & Applications
Sharda School of Computing Science & Engineering, Sharda University, Greater Noida

[![Website](https://img.shields.io/badge/Website-ajaykushwaha.in-6c63ff?style=flat-square)](https://ajaykushwaha.in)
[![Research](https://img.shields.io/badge/Research-Cyber%20Security%20%7C%20Blockchain%20%7C%20AI%2FML-48cae4?style=flat-square)](https://ajaykushwaha.in)

---

## 📄 License

MIT License — free to use, modify, and distribute with attribution.

---

<div align="center">

*Made for researchers, students, and knowledge-seekers who read too many ebooks.*

**⭐ Star this repo if it saved you hours of manual work!**

</div>
