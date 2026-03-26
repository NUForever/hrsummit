# HRsummit — Session Log

## Project Overview
- **App**: HRsummit (formerly Kardiomazov)
- **Purpose**: Heart rate monitor BLE app for high-altitude mountaineering training (5000m+)
- **User**: Iván — training for high-altitude peaks, uses Garmin HRM-600 + Samsung Galaxy S22 Ultra
- **GitHub**: https://github.com/NUForever/hrsummit
- **GitHub Pages**: https://nuforever.github.io/hrsummit/

---

## What Was Built

### Phase 1 — PWA Web App (v2.0)
- Started from a single-file HTML app connecting to Garmin HRM-600 via Web Bluetooth
- Fixed bugs: screen-off data loss (Wake Lock API + timestamp-based timer + IndexedDB persistence)
- Added auto-reconnect BLE on disconnect
- Added performance metrics for altitude training:
  - TRIMP (Banister training load)
  - DFA α1 (aerobic threshold detection via Detrended Fluctuation Analysis)
  - Baevsky Stress Index (autonomic balance)
  - EPOC / Training Effect (recovery demand)
  - LF/HF ratio (Lomb-Scargle frequency domain, acclimatization indicator)
  - Respiratory rate estimation (from HF spectrum peak)
  - Cardiac drift (first-half vs second-half HR comparison)
  - HRV trend slope (RMSSD over time)
- Created proper PWA: manifest.json + sw.js (separated from inline blob)

### Phase 2 — Capacitor Native App
- Installed Node.js (nvm v20), Android SDK (API 34), OpenJDK 21
- Migrated to Capacitor with native BLE plugin (@capacitor-community/bluetooth-le)
- App auto-detects platform: Capacitor native vs Web Bluetooth fallback
- Created BLEForegroundService.java — keeps BLE alive with screen off
- Created MainActivity.java — JS↔Native bridge for foreground service control
- Built signed APK and AAB

### Phase 3 — Rebranding
- Renamed from Kardiomazov → HRsummit
- Changed package: com.nuforever.kardiomazov → com.nuforever.hrsummit
- Renamed GitHub repo: NUForever/kardiomazov → NUForever/hrsummit
- Updated all Java files, AndroidManifest, build.gradle, strings.xml, index.html

### Phase 4 — Play Store Preparation
- Generated release keystore: `kardiomazov-release.keystore` (alias: kardiomazov, pass: kardiomazov2024)
- Built signed AAB: `hrsummit-release.aab` (3 MB)
- Created store listing assets in `store-listing/`:
  - icon-512.png (512x512)
  - feature-graphic.png (1024x500)
  - screenshot-1-live.png (1080x1920)
  - screenshot-2-rendimiento.png (1080x1920)
  - listing-texts.md (EN + ES)
  - privacy-policy.html (bilingual EN/ES, live on GitHub Pages)
  - data-deletion.html (data deletion instructions, live on GitHub Pages)

### Phase 5 — Freemium + i18n (v2.1)
- Added i18n system: English (default) + Spanish, auto-detected from navigator.language
- Added freemium model:
  - FREE: BLE + real-time HR, 5 zones, Min/Avg/Max, timer, basic RMSSD, 30-min session limit, last session only
  - PREMIUM ($4.99/mo or $34.99/yr): Unlimited sessions, DFA α1, TRIMP, EPOC, Baevsky SI, LF/HF, respiratory rate, cardiac drift, HRV trends, full history, recovery score, VO2max, CSV export
- Paywall UI: blur overlay on premium tabs, 30-min limit banner, PRO badges
- New features: Session History (IndexedDB), Recovery Score, VO2max estimation, Altitude Notes
- RevenueCat integration: placeholder `upgradePremium()` ready for real SDK
- Updated privacy policy with RevenueCat disclosure
- Updated store listing texts for freemium model

---

## Current State of Play Store Publishing

### Completed in Play Console ✅
1. ✅ Google Play Developer account created and verified
2. ✅ App "HRsummit" created in Play Console
3. ✅ Store listing filled (English only — user decided no Spanish listing)
4. ✅ Icon, feature graphic, and screenshots uploaded
5. ✅ Content rating completed — rated "Everyone"
6. ✅ Privacy policy URL set: https://nuforever.github.io/hrsummit/store-listing/privacy-policy.html
7. ✅ Data safety section completed (fitness data collected, purchase history shared via RevenueCat)
8. ✅ Ads declaration: No ads
9. ✅ App access: All functionality available without special access
10. ✅ Government apps: No
11. ✅ Financial features: No
12. ✅ Health apps: Fitness & activity tracking (not medical)
13. ✅ Advertising ID: Not used
14. ✅ Store settings: Category "Health & Fitness", contact email set

### Pending — Closed Testing ⏳
Google requires a **closed test with 12 testers for 14 days** before production access.

**Next steps:**
1. Select countries/regions for closed test
2. Create a tester list with **12 email addresses** (friends, family, climbing partners with Android)
3. Upload `hrsummit-release.aab` to closed test track
4. Release name: `2.1.0`
5. Release notes: `Initial release — Heart rate monitor for high-altitude training`
6. Each tester must accept the invitation and install the app
7. Wait 14 days
8. Then request production access

### Pending — RevenueCat Integration ⏳
- The app has a placeholder `upgradePremium()` function
- Need to:
  1. Create RevenueCat account (free tier, up to $2.5K MRR)
  2. Connect RevenueCat to Google Play Console
  3. Create products in Play Console: `hrsummit_monthly` ($4.99) and `hrsummit_yearly` ($34.99)
  4. Install `@revenuecat/purchases-capacitor` plugin
  5. Initialize RevenueCat SDK in app
  6. Replace placeholder with real purchase flow
  7. Rebuild AAB and upload new version

---

## Key Files

| File | Description |
|------|-------------|
| `index.html` | Main app (i18n + freemium + all features) |
| `manifest.json` | PWA manifest |
| `sw.js` | Service worker (browser only) |
| `capacitor.config.json` | Capacitor config (appId: com.nuforever.hrsummit) |
| `package.json` | npm dependencies |
| `kardiomazov-release.keystore` | **CRITICAL** — Release signing key. DO NOT LOSE. |
| `hrsummit-release.aab` | Signed release bundle (3 MB) — upload to Play Console |
| `android/` | Android native project |
| `android/app/src/main/java/.../MainActivity.java` | JS↔Native bridge |
| `android/app/src/main/java/.../BLEForegroundService.java` | Background BLE service |
| `android/app/src/main/AndroidManifest.xml` | Permissions (BLE, foreground service, wake lock) |
| `android/app/build.gradle` | Signing config + build settings |
| `store-listing/icon-512.png` | App icon for Play Store |
| `store-listing/feature-graphic.png` | Feature graphic for Play Store |
| `store-listing/screenshot-*.png` | Screenshots for Play Store |
| `store-listing/listing-texts.md` | Store listing texts EN + ES |
| `store-listing/privacy-policy.html` | Privacy policy (live on GitHub Pages) |
| `store-listing/data-deletion.html` | Data deletion page (live on GitHub Pages) |

---

## Build Commands

```bash
# Load environment
export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$JAVA_HOME/bin:$PATH:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools

# Copy web files to android
cp index.html www/index.html
cp manifest.json www/manifest.json
cp index.html android/app/src/main/assets/public/index.html
cp manifest.json android/app/src/main/assets/public/manifest.json

# Build release AAB
cd android && ./gradlew clean bundleRelease --no-daemon

# Output at: android/app/build/outputs/bundle/release/app-release.aab
```

---

## Keystore Info (DO NOT SHARE)
- File: `kardiomazov-release.keystore`
- Alias: `kardiomazov`
- Store password: `kardiomazov2024`
- Key password: `kardiomazov2024`
- Validity: 10,000 days
- **If you lose this keystore, you cannot update the app on Play Store**

---

## URLs
- GitHub repo: https://github.com/NUForever/hrsummit
- Privacy policy: https://nuforever.github.io/hrsummit/store-listing/privacy-policy.html
- Data deletion: https://nuforever.github.io/hrsummit/store-listing/data-deletion.html
- Play Console: https://play.google.com/console
