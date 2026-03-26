# HRsummit — Development Log & Current State

## Project Overview
- **App**: HRsummit (formerly Kardiomazov)
- **Purpose**: Heart rate monitor BLE app for high-altitude mountaineering training (5000m+)
- **User**: Iván — training for high-altitude peaks, uses Garmin HRM-600 + Samsung Galaxy S22 Ultra
- **GitHub**: https://github.com/NUForever/hrsummit
- **GitHub Pages**: https://nuforever.github.io/hrsummit/
- **Play Console**: https://play.google.com/console
- **Package**: com.nuforever.hrsummit

---

## Development Phases Completed

### Phase 1 — PWA Web App (from existing Kardiomazov code)
- Started from single-file HTML app (GitHub: NUForever/kardiomazov) connecting to Garmin HRM-600 via Web Bluetooth
- **Bug fixes**:
  - Screen-off data loss → Wake Lock API + timestamp-based elapsed timer (not setInterval increment)
  - Data persistence → IndexedDB saves session every 10s, restores on reload
  - BLE disconnection → auto-reconnect without losing session data
- **Performance metrics added for altitude training**:
  - TRIMP (Banister exponential training load, sex-weighted)
  - DFA α1 (Detrended Fluctuation Analysis — real-time aerobic threshold detection)
  - Baevsky Stress Index (sympathetic/parasympathetic balance from RR histogram)
  - EPOC / Training Effect (recovery demand estimation, 0-5 scale)
  - LF/HF ratio (Lomb-Scargle periodogram on RR intervals, acclimatization indicator)
  - Respiratory rate estimation (HF spectrum peak frequency × 60)
  - Cardiac drift (HR first-half vs second-half comparison, aerobic efficiency)
  - HRV trend slope (linear regression on RMSSD samples every 60s)
- Created proper PWA structure: manifest.json + sw.js (was inline blob URL before)

### Phase 2 — Capacitor Native Android App
- **Environment installed on Ubuntu 24.04**:
  - Node.js v20.20.2 via nvm
  - Android SDK Platform 34 + Build Tools 34.0.0 at ~/Android/Sdk
  - OpenJDK 21 (full JDK, not just JRE)
  - GitHub CLI (gh) authenticated as NUForever
- **Capacitor setup**:
  - @capacitor/core, @capacitor/cli, @capacitor/android
  - @capacitor-community/bluetooth-le (native BLE, replaces Web Bluetooth)
  - @capawesome/capacitor-background-task
- **Native Android code**:
  - `BLEForegroundService.java` — Android foreground service with PARTIAL_WAKE_LOCK (4hr), notification channel, keeps BLE alive with screen off
  - `MainActivity.java` — JS↔Native bridge via `@JavascriptInterface` for starting/stopping foreground service from JS, WebView cache disabled
- **Dual-mode BLE**: App detects if running in Capacitor (native) or browser (Web Bluetooth) and uses appropriate API

### Phase 3 — Rebranding (Kardiomazov → HRsummit)
- Renamed GitHub repo: NUForever/kardiomazov → NUForever/hrsummit
- Changed Android package: com.nuforever.kardiomazov → com.nuforever.hrsummit
- Moved Java files to new package directory
- Updated: all Java files, AndroidManifest.xml, build.gradle, strings.xml, capacitor.config.json, index.html, manifest.json, sw.js, all SVG/PNG assets
- Updated all native bridge references: KardiomazovNative → HRsummitNative

### Phase 4 — Play Store Preparation
- **Keystore**: `kardiomazov-release.keystore` (alias: kardiomazov, pass: kardiomazov2024, 10,000 days validity)
- **Store listing assets** (in `store-listing/`):
  - icon-512.svg/png — App icon with mountain silhouette + ECG line + heart
  - feature-graphic.svg/png — 1024x500 banner
  - screenshot-1-live.svg/png — Live tab mockup (1080x1920)
  - screenshot-2-rendimiento.svg/png — Performance tab mockup (1080x1920)
  - listing-texts.md — English + Spanish store descriptions
  - privacy-policy.html — Bilingual EN/ES with toggle, includes RevenueCat disclosure
  - data-deletion.html — Data deletion instructions (Play Store requirement)

### Phase 5 — Freemium Model + i18n (v2.1)
- **i18n system**: ~200+ translation keys in LANG object (en/es), `t(key)` function, English default (no auto-detect), user switchable in Config tab, persisted in localStorage
- **Freemium tiers**:
  - FREE: BLE + real-time HR, 5 zones, Min/Avg/Max, timer, basic RMSSD, 30-min session limit, last session only in history
  - PREMIUM ($4.99/mo or $34.99/yr): Unlimited sessions, all advanced metrics (DFA α1, TRIMP, EPOC, Baevsky SI, LF/HF, respiratory rate, cardiac drift, HRV trends), full session history, Recovery Score, VO2max estimation, CSV export
- **Paywall UI**: blur overlay on premium tabs (Rend./HRV), 30-min session limit banner with dismiss, PRO badges, locked Recovery Score on home screen
- **New features**:
  - Session History — saves completed sessions to IndexedDB `sessions` store with summary stats
  - Recovery Score — combines avg RMSSD (last 5 sessions) + TRIMP (last 7 days), scale 0-100
  - VO2max estimation — Uth et al. 2004 formula: 15.3 × (maxHR / restHR)
  - Altitude Notes — free-text field saved with each session
- **Premium state**: `premiumState` defaults to `true` for testing (change to `false` for production + RevenueCat)
- **RevenueCat**: placeholder `upgradePremium()` toggle function, ready for real SDK integration

### Phase 6 — BLE Debugging (current)
- **Problem**: BLE connection fails in Capacitor native app
- **Root cause identified**: Dynamic ES module import from unpkg doesn't work in Capacitor WebView
- **Fix applied**: Use `Capacitor.registerPlugin('BluetoothLe')` directly with minimal wrapper
- **Additional fixes**:
  - Global try-catch wrapping entire `<script type="module">` to prevent black screen on JS errors
  - WebView cache disabled in MainActivity.java
  - BLE initialization deferred to first connect (lazy init) for permission flow
  - Detailed error messages showing plugin state for debugging
- **Current versionCode**: 8 (v2.1.7) uploaded to Play Store internal test, awaiting propagation
- **Last version tested on device**: v2.1.6 (showed BLE error with plugin list — confirmed plugin IS registered)
- **Play Store propagation**: Takes 10-30 min for internal test versions. User must: uninstall → force stop Play Store → clear Play Store cache → reinstall from internal test link

---

## Active Bug: BLE Not Working in Native App

### Symptoms (progressive debugging)
1. v2.0.0-2.1.3: Shows "Web Bluetooth not available" (was using dynamic import from unpkg which fails in WebView)
2. v2.1.4: Black screen (JS crash — unhandled top-level error in `<script type="module">`)
3. v2.1.5: Black screen (same — no global error handler yet)
4. v2.1.6: Shows error: "BLE plugin not available. Capacitor:true, Plugins:["systemBars","BluetoothLe","CapacitorCookies","WebView","CapacitorHttp","BackgroundTask"]"
   - **Key finding**: BluetoothLe IS in the plugins list, but BleClient is null after try-catch
   - This means `registerPlugin('BluetoothLe')` throws or the try block fails somewhere
5. v2.1.7: Added detailed error capture — will show the EXACT exception from the try-catch

### What's Been Tried
1. Dynamic import from unpkg CDN → fails in WebView (no network module loading)
2. `Capacitor.registerPlugin('BluetoothLe')` with wrapper → plugin is listed but BleClient ends up null
3. Lazy initialization (init on connect, not on load) → avoids permission timing issues
4. Global try-catch → prevents black screen, shows error
5. Dual access: try `registerPlugin()` first, fallback to `Capacitor.Plugins.BluetoothLe` direct → v2.1.7

### Critical Discovery from v2.1.6
The plugin **IS registered natively** (appears in Plugins list). The `registerPlugin()` call or something in the try block throws. v2.1.7 captures and displays the exact error.

### Next Steps to Fix
1. **IMMEDIATE**: Install v2.1.7 when it propagates on Play Store, read the exact error message
2. Based on the error:
   - If `registerPlugin is not a function`: Use `Capacitor.Plugins.BluetoothLe` directly (already added as fallback in v2.1.7)
   - If another error in the wrapper code: Fix the specific line
   - If the plugin proxy doesn't have `initialize`/`requestDevice` methods: May need to call native methods differently
3. Alternative approach: Use `Capacitor.Plugins.BluetoothLe` directly with `addListener` for notifications (skip registerPlugin entirely)

### Technical Notes on Capacitor BLE
- Plugin: `@capacitor-community/bluetooth-le` v8.1.3
- Native class: `com.capacitorjs.community.plugins.bluetoothle.BluetoothLe`
- Registered in: `android/app/src/main/assets/capacitor.plugins.json`
- The plugin's JS wrapper (`BleClient`) uses `registerPlugin('BluetoothLe')` from `@capacitor/core`
- On native, `registerPlugin` creates a proxy that calls native methods via the Capacitor bridge
- Notifications return hex strings on native (not DataView), need conversion: `hexToDataView()`
- Event listener keys: `notification|{deviceId}|{service}|{characteristic}`, `disconnected|{deviceId}`

---

## Play Store Status

### Completed ✅
1. Google Play Developer account created and verified ($25 paid)
2. App "HRsummit" created in Play Console
3. Store listing filled (English only)
4. Icon, feature graphic, screenshots uploaded
5. Content rating: Everyone (with in-app purchases)
6. Privacy policy URL: https://nuforever.github.io/hrsummit/store-listing/privacy-policy.html
7. Data safety: fitness data collected (not shared), purchase history collected+shared (RevenueCat)
8. Ads: No
9. App access: All functionality without special access
10. Government apps: No
11. Financial features: No
12. Health apps: Activity & fitness (not medical)
13. Advertising ID: Not used
14. Category: Health & Fitness
15. Foreground service permission declared (connectedDevice type)
16. Data deletion URL: https://nuforever.github.io/hrsummit/store-listing/data-deletion.html
17. Internal test track created with version uploaded

### Pending ⏳

#### Closed Testing (required for production access)
- Google requires **12 testers** who accept invitation + **14 days** of closed testing
- Need to: select countries, create tester email list, upload AAB to closed test track
- Internal testing is for developer testing (current), closed testing is the gate to production

#### RevenueCat Integration
1. Create RevenueCat account at https://www.revenuecat.com (free tier up to $2.5K MRR)
2. Connect RevenueCat to Google Play Console (service account + API key)
3. Create subscription products in Play Console:
   - `hrsummit_monthly` — $4.99/month auto-renewing
   - `hrsummit_yearly` — $34.99/year auto-renewing
4. Install `@revenuecat/purchases-capacitor` plugin
5. Initialize SDK in app with API key
6. Replace `window.upgradePremium()` placeholder with real purchase flow
7. Add subscription restore functionality
8. Set `premiumState` default to `false` for production
9. Rebuild and upload new AAB

#### iOS Version (future phase)
- Capacitor supports iOS — code is 90% cross-platform
- Requires: Mac with Xcode, Apple Developer Program ($99/yr)
- Changes needed: `npx cap add ios`, Info.plist BLE permissions, background mode config
- Same HTML/JS/CSS, different native shell

---

## Key Files

| File | Description |
|------|-------------|
| `index.html` | Main app (~2090 lines): i18n + freemium + all features + BLE |
| `manifest.json` | PWA manifest |
| `sw.js` | Service worker (browser-only, not used in Capacitor) |
| `capacitor.config.json` | Capacitor config (appId: com.nuforever.hrsummit, webDir: www) |
| `package.json` | npm dependencies |
| `kardiomazov-release.keystore` | **⚠️ CRITICAL** — Release signing key. NEVER LOSE THIS. |
| `hrsummit-release.aab` | Current signed release bundle (3 MB) |
| `android/app/build.gradle` | Signing config, versionCode (currently 7), dependencies |
| `android/app/src/main/java/.../MainActivity.java` | JS↔Native bridge + WebView config |
| `android/app/src/main/java/.../BLEForegroundService.java` | Background BLE service |
| `android/app/src/main/AndroidManifest.xml` | Permissions: BLE, foreground service, wake lock, billing |
| `store-listing/` | All Play Store assets (PNGs, texts, privacy policy, data deletion) |
| `SESSION-LOG.md` | This file |

---

## Build & Deploy Commands

```bash
# 1. Load environment
export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$JAVA_HOME/bin:$PATH:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools

# 2. Copy web files to android assets
cp index.html www/index.html
cp index.html android/app/src/main/assets/public/index.html
cp manifest.json www/manifest.json
cp manifest.json android/app/src/main/assets/public/manifest.json

# 3. Increment versionCode in android/app/build.gradle BEFORE building!

# 4. Build release AAB
cd android && ./gradlew clean bundleRelease --no-daemon
# Output: android/app/build/outputs/bundle/release/app-release.aab

# 5. Copy to project root
cp app/build/outputs/bundle/release/app-release.aab ../hrsummit-release.aab

# 6. Upload to Play Console internal/closed test

# 7. Push to GitHub
cd .. && git add -A && git commit -m "message" && git push origin main
```

---

## Keystore Info (⚠️ DO NOT SHARE)
- **File**: `kardiomazov-release.keystore`
- **Alias**: `kardiomazov`
- **Store password**: `kardiomazov2024`
- **Key password**: `kardiomazov2024`
- **Validity**: 10,000 days
- **WARNING**: If you lose this keystore, you CANNOT publish updates to the app on Play Store. Back it up.

---

## URLs
- **GitHub**: https://github.com/NUForever/hrsummit
- **Privacy Policy**: https://nuforever.github.io/hrsummit/store-listing/privacy-policy.html
- **Data Deletion**: https://nuforever.github.io/hrsummit/store-listing/data-deletion.html
- **Play Console**: https://play.google.com/console
- **Internal Test Link**: https://play.google.com/apps/internaltest/4701252787958829765

---

## Version History

| versionCode | versionName | Changes | Status |
|-------------|-------------|---------|--------|
| 1 | 2.0.0 | Initial Capacitor build, Web Bluetooth import from unpkg | On Play Store (broken BLE) |
| 2 | 2.1.0 | Freemium + i18n + session history + recovery score | Uploaded |
| 3 | 2.1.2 | BLE fix attempt (Capacitor.Plugins direct access) | Uploaded |
| 4 | 2.1.3 | BLE fix (registerPlugin + lazy init) | Confirmed on device |
| 5 | 2.1.4 | BLE fix + improved error handling | Confirmed on device (black screen) |
| 6 | 2.1.5 | English default, WebView cache disabled, debug error info | Uploaded |
| 7 | 2.1.6 | Global try-catch (prevents black screen), BLE registration in try-catch | Tested on device — shows "BLE plugin not available" with plugin list |
| 8 | 2.1.7 | Detailed BLE registration error capture, dual plugin access (registerPlugin + Plugins direct) | Built, uploaded to Play Store internal test, awaiting propagation |
