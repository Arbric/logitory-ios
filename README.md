# Logitory for iPhone and iPad

A [Capacitor 8](https://capacitorjs.com) shell around the Logitory web app, the iOS twin of
`logitory-android`. The app opens **https://logitory.io/login/** full screen, so every web deploy reaches
phones straight away; a new App Store build is only needed for changes to the shell itself (icon, name,
link rules, permissions, iOS versions).

## What the shell does
- **Loads** `https://logitory.io/login/` (`capacitor.config.json` → `server.url`).
- **Keeps in the app:** logitory.io and Logitory's Supabase project (sign-in links). **Everything else**
  (other websites, `mailto:`, `tel:`) opens in Safari, Mail or Phone.
- **No billing in the app:** the user agent ends in `LogitoryiOS`, and the web app then hides Billing,
  Add Licenses and Pay Invoice (App Store Review Guideline 3.1.1), as it does for Android. Administrators
  manage billing on the web. Export to .xlsx and the sample template are hidden too (the web view
  doesn't save downloads).
- **No connection:** shows a local "Can't reach Logitory · Try Again" page (`www/offline.html`).
- **Location:** asked for only when someone taps Copy Current Location (`NSLocationWhenInUseUsageDescription`).
- **Export compliance:** `ITSAppUsesNonExemptEncryption` is false (HTTPS only), so App Store Connect
  doesn't ask each upload.
- iPhone and iPad, iOS 15 or later. App id / bundle id: `io.logitory.app` (same as Android).

## Build
iOS apps can only be built on a Mac with Xcode. Two ways:

**On GitHub (no Mac needed).** Every push to `main` runs *iOS build* (Actions) on a macOS runner:
a Simulator build always (proves it compiles), and, once the App Store Connect secrets below are set, a
signed App Store build uploaded to App Store Connect, where it appears under TestFlight.

**On a Mac.**
```sh
npm install
npm run sync      # copies capacitor.config.json and www/ into ios/
npm run open      # opens Xcode: pick your Team under Signing & Capabilities, then Run ▶ or Product → Archive
```

## Release (App Store)
1. **Apple Developer Program** (developer.apple.com, US$99 a year). As an organization you need a D-U-N-S
   number. Note the **Team ID** (Membership details).
2. **App Store Connect → Apps → +** New App: platform iOS, name Logitory, bundle id `io.logitory.app`
   (register it first under Certificates, Identifiers & Profiles → Identifiers if it isn't listed),
   SKU e.g. `logitory-ios`.
3. **API key** (App Store Connect → Users and Access → Integrations → App Store Connect API → Team
   Keys → +): role **Admin** (needed for automatic, cloud-managed signing). Download the `.p8` once and
   note its **Key ID** and the **Issuer ID**.
4. **GitHub secrets** (this repo → Settings → Secrets and variables → Actions):
   - `APPLE_TEAM_ID`, `ASC_KEY_ID`, `ASC_ISSUER_ID`
   - `ASC_KEY_P8_BASE64`: the `.p8` file base64-encoded. In PowerShell:
     `[Convert]::ToBase64String([IO.File]::ReadAllBytes("AuthKey_XXXX.p8")) | Set-Clipboard`
5. **Actions → iOS build → Run workflow.** The build uploads to App Store Connect; after processing
   (10–30 min) it's under **TestFlight**. Install it with the TestFlight app on an iPhone to check it.
6. **App Store tab:** screenshots (6.9" iPhone and 13" iPad), description, keywords, support URL,
   privacy policy URL, **App Privacy** answers, age rating, and **App Review Information** with a demo
   login (Logitory needs a sign-in, and review is rejected without one). Pick the TestFlight build,
   then **Submit for Review**.

The build number is the workflow run number (each upload must be higher); change `MARKETING_VERSION`
in Xcode (or `ios/App/App.xcodeproj/project.pbxproj`) for the version people see.

## Icons and splash
Generated from `assets/` (the Logitory mark, 1024 × 1024, no transparency) with `npm run icons`.

## Changing the shell
Edit `capacitor.config.json`, then `npm run sync`.
