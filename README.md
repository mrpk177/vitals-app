# PhotoPulse Lab

An experimental, browser-based camera vitals reader (remote photoplethysmography / rPPG) with per-person profiles, session comparison, a printable report, Bluetooth heart-rate device support, and GitHub-repo-based cloud sync.

**⚠️ Not a medical device.** This is a hobbyist signal-processing project. It estimates heart rate, breathing rate, and pulse variability from ordinary webcam video. It does **not** measure blood oxygen or blood pressure, does not diagnose anything, and should never be used to make a health decision. See the in-app disclaimer and footer for details.

Live app: `https://raw.githack.com/<your-username>/<your-repo>/main/photopulse-lab-3.html`
(replace with your own fork's path — see **Setup from scratch** below)

---

## Features

- Camera-based heart rate, breathing rate, and pulse-variability (HRV) estimation via rPPG
- Named profiles (not biometric face recognition — see *Why no face-ID* in the app's own footer)
- Session-to-session comparison with plain-language, non-diagnostic observations
- Published reference ranges (AHA / Mayo Clinic / Cleveland Clinic, cited in-app) for context
- Bluetooth heart-rate device support (standard BLE Heart Rate service — Polar, Garmin, etc.)
- Manual equipment/doctor-notes log (pulse oximeter, BP cuff, thermometer readings you enter yourself)
- Camera controls: fullscreen, front/back flip, flashlight toggle, zone/timer/distance guidance
- Skin-tone plausibility filter (prevents non-face objects from producing a fake reading)
- Printable report: letterhead, triple-line bordered A4-style layout, Preview-before-print
- Share report via native share sheet / WhatsApp / email
- CSV export (Excel-compatible) and JSON backup export/import
- GitHub-repo-based cloud sync (push/pull your data as JSON using your own token)
- Two themes: "Lab" (dark/amber) and "Clinical" (white/blue)
- Optional local PIN lock for the device itself (not a real multi-user login — see Limitations)

## Limitations (please read)

- **Not clinically validated.** Treat every reading as an estimate, not a fact.
- **No blood pressure or SpO2.** Plain RGB webcams cannot reliably measure either; use a real pulse oximeter / BP cuff and log the reading in the Equipment panel instead.
- **No real user accounts.** There is no backend server here, so there's no password-based login. GitHub-repo sync is a practical workaround, not bank-grade security — see the in-app warning before using it.
- **No automatic face recognition.** People are told apart by the name you type, not biometrics.
- Web Bluetooth (device connection) works on Chrome/Edge for Android and desktop only — not supported in Safari/iOS.
- Camera access requires a secure context (HTTPS or `localhost`). It will not work opened directly as a local `file://` page — see Setup below.

---

## Setup from scratch (for a new contributor, or moving to production)

### 1. Get the code
Fork or clone this repository, or start a new one and copy `photopulse-lab-3.html` into it. It's a single self-contained HTML file — no build step, no dependencies to install.

### 2. Quick hosting for testing (fastest, good for personal use)
GitHub Pages doesn't strictly need to be enabled for a *quick* live link — you can use a raw-file CDN proxy instead:

```
https://raw.githack.com/<owner>/<repo>/<branch>/photopulse-lab-3.html
```

This works immediately once the file is committed to the repo, no configuration needed. Good for personal use and testing, but note raw.githack is a third-party convenience CDN, not designed for sustained high-traffic production use (see their own site for current usage guidance).

### 3. Proper hosting for production
For anything beyond personal/testing use, host it as a real static site instead:

- **GitHub Pages** (free): repo → Settings → Pages → Source: "Deploy from a branch" → branch `main`, folder `/(root)` → Save. Your app will be at `https://<owner>.github.io/<repo>/photopulse-lab-3.html`.
- **Netlify / Vercel / Cloudflare Pages** (free tiers, more robust than GitHub Pages for production traffic): connect the repo and deploy — each auto-detects a static HTML file with no config needed.
- Any of these give you free HTTPS automatically, which is required for camera access to work at all.

If GitHub Pages' settings UI doesn't render fully on a mobile browser (a known mobile-web rendering quirk), the direct settings URL is `https://github.com/<owner>/<repo>/settings/pages`, or use a desktop browser for that one-time setup step.

### 4. (Optional) Enable GitHub-repo cloud sync
1. Create a **fine-grained personal access token**: `https://github.com/settings/personal-access-tokens/new`
2. Scope it to **only this repository**, with **Contents: Read and write** permission — nothing else.
3. In the app's Cloud Sync panel, enter `owner/repo` and paste the token.
4. Use Push/Pull to sync a profile's data as `data/<profile-name>.json` in the repo.
5. Revoke or rotate the token anytime from the same GitHub settings page.

### 5. (Optional) Automating deploys for contributors
If you want changes pushed programmatically (e.g., by an AI assistant or a script) rather than manual upload:
- Create a token as above (Contents: Read/write, scoped to one repo).
- Use the GitHub Contents API: `GET /repos/{owner}/{repo}/contents/{path}` to fetch the current file SHA, then `PUT /repos/{owner}/{repo}/contents/{path}` with the base64-encoded new content and that SHA to update it.
- Treat the token like a password — don't commit it into the repo itself.

### 6. Local development
There's no build process. To test locally with camera access working (camera requires a secure context, so opening the file directly via `file://` will not work):
```bash
python3 -m http.server 8000
# then open http://localhost:8000/photopulse-lab-3.html
```

---

## Changelog

### v2.2
- Added a "Clear reading" button next to Start/Stop — discards the current in-progress (unsaved) reading and restarts capture without stopping the camera.

### v2.1 — Critical fix
- Fixed a bug where the entire app was invisible on screen. The new printable-report frame wrapper had inadvertently inherited `display:none` by default (it's only meant to appear during printing/preview), hiding the whole page. Found via automated headless-browser testing, not just code review.

### v2.0 — Major update
- **Camera controls:** fullscreen toggle, front/back camera flip, flashlight/torch toggle (back camera, where supported by hardware).
- **Live guidance:** on-screen "forehead" zone label, session timer against a research-based ~30s minimum reading duration, and distance guidance (~50cm / arm's length, based on published rPPG studies).
- **Face plausibility filter:** basic skin-tone heuristic to stop non-face objects (e.g. a TV remote) from producing a fake pulse reading; paired with a live face-lock stability indicator.
- **Report redesign:** professional letterhead (app name, report ID, patient name, generated date), triple-line bordered A4-style print layout, and terse clinical-register "Observations" in the printed report (casual explanatory language stays in-app only).
- **Preview Report:** see the exact print/export layout before committing to print or share.
- **Cloud sync:** push/pull profile data to/from your own GitHub repo as a lightweight alternative to a real login system (which would require a backend this project doesn't have).
- **Theme & readability overhaul:** theme-aware colors throughout (including canvas-drawn elements), plain-language subtitles under technical readouts (Signal Quality, HRV) aimed at a wide age range.
- **Performance:** cached expensive style lookups that were previously running 60×/sec, cached the static oscilloscope grid instead of redrawing it every frame, throttled the waveform animation to 24fps, and moved a per-frame DOM write down to a 1Hz update.

### v1.x — Foundation
- Initial rPPG heart-rate/breathing/HRV estimation from webcam video, with a live oscilloscope-style waveform display.
- Named profiles, session history, and session-to-session comparison with plain-language (non-diagnostic) observations.
- Published reference ranges for heart rate and breathing rate, cited to AHA / Mayo Clinic / Cleveland Clinic.
- Printable PDF report via the browser's print dialog.
- Camera device selector and resolution presets (for external/USB webcams).
- Improved camera-access error handling: distinguishes permission-denied, no-device-found, in-use-elsewhere, and — notably — pages opened as a local `file://` file, which browsers silently block from camera access regardless of in-page permission prompts.
- Platform-aware fix instructions (Android / iPhone / Desktop) for getting camera access working outside a local file.
- **Storage bug fix:** switched from a Claude-artifact-only storage API to real `localStorage`, since the artifact-only API silently doesn't exist once the app is hosted externally (e.g. on GitHub Pages) — session data wasn't actually persisting before this fix.
- Bluetooth heart-rate device connection (standard BLE Heart Rate service).
- Manual equipment/doctor-notes log for readings from real medical devices.
- Share report via native share sheet, WhatsApp, and email links.
- CSV (Excel-compatible) and JSON backup export/import.
- Clinical (white/blue) theme option alongside the original dark/amber "Lab" theme.

---

## Explicitly out of scope (and why)

A few features were requested during development and deliberately not built:

- **Visual disease/pattern detection from face or body camera images**, cross-referenced against "world medical databases." Not attempted — this is an unresolved, actively-researched clinical imaging problem, and a hobbyist script claiming to do it (especially framed as checked against medical literature) would make a wrong guess *more* convincing, not less.
- **Blood pressure or SpO2 estimation from the webcam.** Not reliably possible with a standard RGB camera; a real pulse oximeter or BP cuff (logged in the Equipment panel) is the honest answer here.
- **A real password-based login system with cross-device sync.** This is a static file with no backend server — GitHub-repo sync is the practical alternative that was built instead.
