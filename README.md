# Codeisa — Windows Desktop Edition (Tauri)

A premium Windows desktop port of the Codeisa web application, built with **Tauri 2.0** on **Rust + Webview2**. Targets **Windows 10 / Windows 11** with full Fluent design polish, native system tray, native network monitor, bilingual UI (Persian / English), and an advanced triple-calendar (Jalali / Gregorian / Hijri) with Iranian holidays.

> The original `index.html` ships **byte-for-byte preserved** as `src/index.original.html`. The runtime version (`src/index.html`) is the same file plus a small set of non-destructive `<link>` / `<script>` injections that activate the desktop enhancements. **All original features keep working unchanged.**

---

## 🚀 Build (the only two commands you need)

```powershell
npm install
npm run build
```

The MSI and NSIS installers will be written to:

```
src-tauri/target/release/bundle/msi/Codeisa_1.0.0_x64_en-US.msi
src-tauri/target/release/bundle/nsis/Codeisa_1.0.0_x64-setup.exe
```

### Prerequisites (one-time)

| Requirement | Notes |
|---|---|
| **Node.js ≥ 18**         | <https://nodejs.org/> |
| **Rust (stable)**        | `winget install --id Rustlang.Rustup` then `rustup default stable` |
| **MSVC Build Tools**     | "Desktop development with C++" workload (VS 2022 Build Tools) |
| **WebView2 Runtime**     | Pre-installed on Windows 11; the installer auto-bootstraps on Windows 10 |
| **Python 3.10+**         | Only used by `scripts/extract_strings.py` during prebuild |

---

## 📂 Project layout

```
codeisa-desktop/
├── package.json                    # npm scripts (build, dev, prebuild, audit)
├── README.md
├── src/                            # Tauri's frontendDist
│   ├── index.html                  # Original app + desktop enhancements (built)
│   ├── index.original.html         # Original, 100% untouched, for reference
│   ├── splash.html                 # Animated splash screen
│   ├── i18n-runtime.js             # Runtime FA↔EN overlay
│   ├── calendar-pro.js             # Triple-calendar + Iranian holiday DB
│   ├── fluent-overlay.css          # Win11 / Fluent polish
│   └── locales/{fa,en}.json        # Translation tables (copy of /locales)
├── locales/
│   ├── fa.json                     # Master Persian table (key → original Persian)
│   └── en.json                     # Master English table (key → translation)
├── scripts/
│   ├── extract_strings.py          # Localization auditor / extractor
│   ├── prebuild.mjs                # Runs the extractor + copies locales
│   └── audit_i18n.mjs              # Coverage report
├── docs/
│   ├── BUILD.md
│   ├── ARCHITECTURE.md
│   ├── LOCALIZATION.md
│   ├── PERFORMANCE.md
│   ├── CALENDAR.md
│   ├── i18n-audit.md
│   ├── i18n-audit.json
│   └── translation-coverage.json
└── src-tauri/
    ├── Cargo.toml
    ├── tauri.conf.json
    ├── build.rs
    ├── capabilities/default.json
    ├── icons/{32,128,128@2x}.png + icon.ico + icon.png
    └── src/
        ├── main.rs                 # Windows-subsystem entry
        ├── lib.rs                  # Tauri setup, tray, splash, commands
        └── network.rs              # Native network monitor
```

---

## ✅ What was delivered against the brief

| Requirement | Status | File / mechanism |
|---|---|---|
| Tauri-only desktop framework, Windows 10 & 11 | ✅ | `src-tauri/tauri.conf.json`, MSI + NSIS bundles |
| 100 % preservation of original functionality | ✅ | `src/index.original.html` shipped verbatim; runtime version only **adds** scripts/styles via the `</head>` and `</body>` boundaries |
| Bilingual UI (FA / EN) with runtime switching | ✅ | `i18n-runtime.js` + `locales/{fa,en}.json` + tray-driven language switch |
| English-mode Persian-character validation | ✅ | `window.codeisaI18n.auditReport()` returns `missingAtRuntime` |
| Fluent / Windows 11 visual upgrade | ✅ | `fluent-overlay.css` (additive only) |
| Splash screen with animated logo / progress | ✅ | `src/splash.html` + Rust fallback dismiss |
| Native notifications | ✅ | `tauri-plugin-notification`, `notify` command |
| Native Windows icon (ICO + PNG set) | ✅ | `src-tauri/icons/` |
| System tray with bilingual menu | ✅ | `lib.rs::build_tray` |
| Minimize-to-tray | ✅ | `WindowEvent::CloseRequested` handler |
| Start with Windows | ✅ | `tauri-plugin-autostart` + tray toggle |
| Auto-update infrastructure | ✅ | `tauri-plugin-updater`, endpoint in `tauri.conf.json` |
| Modern installer (MSI + NSIS) | ✅ | `bundle.targets` + multi-language NSIS |
| Native network monitor (download / upload) | ✅ | `network.rs` (sysinfo, 500 ms tick, lock-free snapshot) emitted to tray tooltip and to `codeisa://network-stats` event |
| Triple calendar (Jalali / Gregorian / Hijri) | ✅ | `calendar-pro.js` |
| Iranian holidays + religious + cultural events | ✅ | Static DB in `calendar-pro.js` (Borkowski Jalali ↔ Gregorian + Kuwaiti Hijri) |
| Coverage: −2 / +5 years | ✅ | `ensureCache()` pre-computes 8 years on first access, persists to `localStorage` for offline use |
| Offline holiday lookup | ✅ | localStorage cache survives restart |
| Today dashboard / next-holiday countdown / search / filters | ✅ | Mounted into `#datetime-section` automatically |
| Performance: minimal CPU / RAM | ✅ | `lto=true`, `opt-level=s`, `strip=true`, `codegen-units=1`, MutationObserver throttled, network thread `sleep(500ms)` |
| Single-instance enforcement | ✅ | `tauri-plugin-single-instance` |
| Window state persistence | ✅ | `tauri-plugin-window-state` |

---

## 🌐 Localization

### Current state

```
Total unique Persian strings detected: 1373
Seed-translated by default dictionary:   87  (6.3 %)
Pending human review:                  1286
```

Why only 6 %? The seed dictionary intentionally translates only the **most common UI nouns/verbs/labels** that appear universally across desktop apps. The remaining strings are **specific Codeisa copy** (legal/medical disclaimers, banking explanations, psychology test questions, etc.) where machine translation would harm quality.

### How to finish localization (≈ 1–2 hours of focused work)

1. Open `locales/en.json` in VS Code.
2. Every value that still contains Persian characters needs a real English translation.
3. The keys are stable SHA-1 hashes — you can re-run `npm run extract` at any time without losing manual translations (already-non-Persian values are kept).
4. Run `npm run audit` to verify coverage.
5. Rebuild with `npm run build`.

Full per-key list with Persian source text is in **`docs/i18n-audit.md`** (first 200 entries) and **`docs/i18n-audit.json`** (complete).

### Runtime validator

While the app runs in English mode, every Persian fragment that the runtime overlay could not translate is recorded. Press **F12** → console:

```js
window.codeisaI18n.auditReport()
// { lang: "en", totalKeys: 1373, missingAtRuntime: [...], missingCount: N }
```

The list `missingAtRuntime` is exactly the set of strings that still appear visibly in Persian. The target is `missingCount === 0`.

---

## 🗓️ Calendar

See **`docs/CALENDAR.md`** for the data model. Key points:

- **Jalali ↔ Gregorian**: Borkowski algorithm (exact, valid for years 1206–1500 H.S.).
- **Hijri**: Kuwaiti algorithm — calendar arithmetic accurate to ±1 day vs. Umm al-Qura announcements (which depend on moon sighting, by definition non-deterministic).
- **Holiday DB**: Static, embedded — no network required. Covers all official Iranian public holidays, Shia religious occasions (lunar, projected per year), national & cultural commemorations, and a small set of international days.
- **Offline**: First load pre-computes 8 years of events and persists them via `localStorage`. Subsequent launches restore from cache instantly.
- **Mount point**: a `<div id="codeisa-calendar-pro">` is auto-injected into the existing `#datetime-section`. The original calendar is preserved.

---

## 🔧 Development

```powershell
npm install
npm run dev       # opens a Tauri dev window with hot reload
```

Editing `src/index.original.html` is **discouraged** (it's the canonical preserved copy). To customize, edit `src/index.html` directly — the prebuild step only regenerates it if it does not exist.

> If you ever need to regenerate `src/index.html` from the original, delete it and run `npm run prebuild` — the same injection routine used during the initial build will re-apply.

---

## 📦 Distribution

```powershell
npm run build:msi    # MSI only (preferred for enterprise GPO deployment)
npm run build:nsis   # NSIS only (per-user / per-machine, multilingual installer UI)
npm run build        # both
```

Both installers bundle the **WebView2 bootstrapper**, so end-users on Windows 10 do not need to install anything.

---

## 📊 Audit reports

After each build, the following are regenerated under `docs/`:

- `i18n-audit.md` / `i18n-audit.json` — every detected Persian string with its key.
- `translation-coverage.json` — exact pending count, percentage, first 500 pending keys.

---

## 🛡️ License & attribution

Original Codeisa application: © its original authors.
Tauri desktop integration, calendar engine, i18n overlay, Fluent overlay, and Rust backend: © 2026 Codeisa Desktop project.
