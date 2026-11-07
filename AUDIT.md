# Video Filters 1.4.1 — source audit

Reviewed 2026-11-07. Fork: https://github.com/facundosterzerforino/VideoFilters

## Verdict and scope

**No obvious malicious behavior was found in the reviewed source.** This is an AI-assisted source review and limited functional check, not an independent security certification or a guarantee that no vulnerability exists.

Pinned upstream commit: `e8f311e3ffe5047860ab8b3f8becbba8857f2ec1`.

All 19 tracked files under `Extension/` were accounted for: six JavaScript files, two HTML files, one stylesheet, one manifest, and nine PNG icons. Every text/runtime file was read; icon signatures, resource paths, working-copy hashes, JavaScript syntax, and package contents were checked. PNG files were not independently fuzzed or analyzed for decoder vulnerabilities. Runtime code and permissions were **not modified**. The project is MIT-licensed.[9]

This review covers the source snapshot and the packaged files whose hashes are recorded here. It does not cover a Chrome Web Store binary, future commits, every possible video platform, or the browser's own security.

## Data-flow and execution review

- No fetch/XHR/WebSocket/EventSource/beacon request code, analytics SDK, remote JavaScript loader, `eval`/`new Function`, or obfuscated executable payload was found in the runtime. The WebGL shader editor compiles user-provided **GLSL**, not JavaScript.[13]
- No cookie, password, clipboard, native-messaging, browser-history, download, or proxy access was found. The manifest requests only `storage`, `activeTab`, and `scripting`, plus broad host access.[8]
- Page scripts are injected when the extension popup is used. There is no declared always-running content script that automatically inspects every loaded site. The background service worker initializes/migrates saved settings on extension installation/update.[8][15]
- The popup reads video elements, their inline filters, playback rate, dimensions, and frame URLs; it changes CSS filter properties, playback rate, picture-in-picture, and optional shader canvases.[13]
- The options page has ordinary user-clicked links for GitHub feedback, PayPal donations, and icon attribution. It does not automatically load a remote font or remote script.

## Findings and caveats

### F1 — Broad host permissions: privacy risk by design

`host_permissions` contains `<all_urls>`.[8] The reviewed code uses injection into the active tab and its frames, but the grant is broader than a Netflix-only use case requires.

**Recommendation:** start with Vivaldi's site access set to **On click**, or limit it to streaming sites you actually use. Cross-origin embedded players may need additional site permission. Do not enable incognito or local-file access unless needed. The manifest itself was not narrowed.

### F2 — Preferences are not strictly local-only

Defaults, presets, shader source, and the advanced-PiP setting use `chrome.storage.sync`.[13][15] They may therefore be handled by the browser's configured synchronization service; no developer-owned collection endpoint was found. Browser Sync behavior is not equivalent to guaranteed local-only storage.

The last targeted frame URL, video index, serialized CSS filter, and playback-rate values are stored through `chrome.storage.local`.[13] The frame URL may include its query string and is not explicitly cleared by the extension. This is not a browsing-history database, but it is retained page metadata.

### F3 — Low-severity hardening issue: custom labels interpreted as HTML

The popup assigns a saved preset name through `newOption.innerHTML = name` and a custom shader label through `label.innerHTML = props.label + ':'`.[13]

A harmless isolated-browser test confirmed that markup in those settings creates HTML child elements instead of being displayed literally. No arbitrary JavaScript execution, credential access, or external request was demonstrated. The inputs come from user-entered/synchronized extension settings, not directly from a site's video title. Treat this as an HTML-injection/hardening concern, not evidence of malware.

**Recommendation:** use trusted preset names and shader snippets. A future hardening patch should replace these text-only sinks with `textContent` and add regression tests. This audit did not silently modify the runtime to fix them.

### F4 — Functional/robustness limitations

- Presets are applied manually. Automatic per-site application is not implemented in this snapshot.
- Shader effects require reading video pixels into WebGL and are explicitly unsupported for DRM-protected content such as Netflix. Normal CSS filters are a separate mechanism.[3][13]
- Shader cleanup, cross-frame targeting, page navigation/video replacement, and long-session behavior were not comprehensively tested. The code uses shared local-storage keys for frame/video targeting and installs shader-related observers/listeners; concurrent popups or repeated shader cycles deserve additional testing.[13]
- There is no upstream test suite, package manifest/lockfile, bundled third-party JavaScript library, npm dependency tree, or build system in this repository. An `npm audit` result is therefore not applicable; no npm install was performed for this extension. This should not be described as a proof of zero vulnerabilities.

## Verification performed

| Check | Result |
|---|---|
| Real GitHub fork and correct upstream parent | Verified |
| Six JavaScript files: `node --check` | 6 passed |
| All runtime source and manifest unchanged relative to pinned commit | Verified |
| Manifest, HTML and CSS local resource paths | Verified |
| PNG signatures and dimensions | 9 checked |
| Picture-filter code in a separate headless Chromium real DOM | 11 checks passed, 0 failed |
| Custom preset/shader-label markup interpretation | Confirmed, reported as F3 |
| ZIP CRC and every runtime entry against the canonical files | Verified |
| Vivaldi installation, full popup/Chrome API integration, preset persistence, live Netflix/DRM, PiP and shader rendering | **Not tested** |

The 11 real-DOM checks exercised the actual extracted `parseFilter` and `pfToString` source functions: default values, browser acceptance/round-trip of each of the nine CSS filters, and percent/unit normalization. They did **not** run the entire extension or measure image quality, playback performance, or rendered Netflix video pixels.

The two label observations were separate from those passing filter checks; they remain documented findings, not passing security tests. Raw results and per-file SHA-256 values are in `audit/audit-record.json`.

## Package provenance

- Canonical load-unpacked folder: `C:\Users\fsterzer\Downloads\VideoFilters\Extension`
- Extension version: **1.4.1**; manifest version: **3**.[8]
- Runtime source commit: `e8f311e3ffe5047860ab8b3f8becbba8857f2ec1`
- Runtime modifications: **none**; working-copy line endings may differ from Git blob line endings.
- Build/install scripts executed: **none**; the extension is already plain browser JavaScript.
- ZIP: `VideoFilters-1.4.1-audited.zip`
- ZIP size: **31,346 bytes**
- ZIP entries: **21** — 19 runtime assets plus MIT `LICENSE` and `SOURCE.txt` provenance.
- ZIP SHA-256: `18711cc5e705010ea9a2bece143b1f41c4dc804bfb6ed3d4d8c814a93e0dc834`

Only documentation and audit evidence were added to this fork. No runtime code, permissions, browser settings, subscription, or scheduled update mechanism was added or changed. The user chose self-installation; **this extension was not installed in Vivaldi by the agent**. Installation and safe update steps are in `INSTALL-VIVALDI.md`.

## Sources

[3] https://github.com/FWeynschenk/VideoFilters/blob/master/readme.md
[8] https://github.com/FWeynschenk/VideoFilters/blob/e8f311e3ffe5047860ab8b3f8becbba8857f2ec1/Extension/manifest.json — Video Filters Manifest V3 and permissions
[9] https://github.com/FWeynschenk/VideoFilters/blob/e8f311e3ffe5047860ab8b3f8becbba8857f2ec1/LICENSE
[13] https://github.com/FWeynschenk/VideoFilters/blob/e8f311e3ffe5047860ab8b3f8becbba8857f2ec1/Extension/popup.js
[15] https://github.com/FWeynschenk/VideoFilters/blob/e8f311e3ffe5047860ab8b3f8becbba8857f2ec1/Extension/background.js — Pinned Video Filters settings/background source
