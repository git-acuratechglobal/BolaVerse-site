# bolaverse.com — invite links (universal / app links)

Serves the two verification files that let `https://bolaverse.com/join/<CODE>`
open the app, plus a fallback page for people who don't have it installed.

```
hosting/
  .well-known/apple-app-site-association   iOS  — no file extension, served as application/json
  .well-known/assetlinks.json              Android
  join.html                                fallback page (/join/** rewrites here)
  index.html, Help_Center.html,            static landing site (copied from BolaVerse-site-main)
  privacy-policy.html, terms-of-service.html
  images/                                  competition flags, league logos, avatars (copied from assets/)
```

## Before this works — two things are still missing

1. **DNS.** Add `bolaverse.com` as a custom domain:
   `firebase hosting:sites` → Firebase console → Hosting → Add custom domain,
   then swap the GitHub Pages A records (185.199.108-111.153) at the registrar
   for the records it gives you. Keep the existing MX/SPF/TXT records.
2. **Android fingerprint.** Replace `REPLACE_WITH_PLAY_APP_SIGNING_SHA256` in
   `assetlinks.json` with the **Play App Signing** SHA-256 from
   Play Console → Release → Setup → App signing. Not the local keystore's:
   Google re-signs the app. (Note `android/app/build.gradle.kts` still signs
   release builds with the debug key — fix that before the first upload.)

The iOS `appID` is already filled in: `UG4L8VGUD5.com.bolaverse.app`
(`DEVELOPMENT_TEAM` from the Xcode project + the bundle id). Confirm that team
is the one the App Store build ships under.

## Deploy

```bash
firebase deploy --only hosting
```

## Verify

```bash
curl -i https://bolaverse.com/.well-known/apple-app-site-association   # 200, application/json
curl -i https://bolaverse.com/.well-known/assetlinks.json              # 200, real fingerprint
curl -i https://bolaverse.com/join/ABC123                              # the fallback page
```

Android also checks verification on the device:

```bash
adb shell pm get-app-links com.bolaverse.app     # bolaverse.com: verified
```

On iOS, reinstall the app after the files are live — the system fetches the
association file at install time.
