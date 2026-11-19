# Demo v4.5.1 — ビルド・検証状況

## 基本情報
- Base repository: https://github.com/a09848281-prog/web
- Base branch: rebuild-v4.5.1-smart-motion-pwa
- Base commit: 0b61527973d69e01fab87df16524d50dcc452506
- Demo repository: https://github.com/a09848281-prog/a
- Demo branch: demo-v4.5.1
- Verified applicationId: jp.a09848281.remotecamera.v451demo
- versionCode: 451
- versionName: 4.5.1
- GAS API: 4.5.1
- Firebase: not added
- External ML APIs: not added

## Automated validation and APK build — SUCCESS
- Workflow run: https://github.com/a09848281-prog/a/actions/runs/44539321832
- Run ID: 44539321832
- Tested commit: 2ad85425aacfed9a55bf9e6965ca43529407898c
- Status: completed / success
- Passed: JavaScript syntax, Service Worker syntax, GAS syntax, JSON/manifest parse, HTML ID uniqueness, DOM references, left/right layout, motion-zone/person-candidate checks, API version checks, demo applicationId and remote config URL checks.
- Android Gradle debug APK build: success
- Artifact upload: success

### APK Artifact
- Name: remote-camera-debug-apk-v4.5.1-demo
- Artifact ID: 14538335408
- Size: 26,099,030 bytes (about 26 MB)
- SHA-256: 69825a1f2e2ef8443fbb583cb19a166a5758e3c963ebed23a6c0cd1e0cd6474a
- Created: 2026-11-19 14:03:47 UTC
- Expires: 2026-12-19 14:03:45 UTC
- Download: open the Actions run above, then download the artifact named remote-camera-debug-apk-v4.5.1-demo. Extract the ZIP and install app-debug.apk.

## GitHub Pages — PENDING INITIAL REPOSITORY SETUP
- Initial deployment run: https://github.com/a09848281-prog/a/actions/runs/44538825891
- The deployment stopped in Configure GitHub Pages because the GitHub Actions integration cannot create the Pages site for this repository.
- Required user action: repository Settings → Pages → Build and deployment → Source → GitHub Actions.
- After enabling Pages, run the workflow “Deploy Remote Camera v4.5.1 Demo to GitHub Pages” from Actions (workflow_dispatch) or push a change under web/.
- Expected site URL after successful deployment: https://a09848281-prog.github.io/a/

## Real-device verification — NOT YET CONFIRMED
CI success does not prove that camera capture, microphone routing, flashlight, foreground waiting, network traversal, iPad Safari playback, PWA behavior, or motion detection work on physical devices. Verify on Galaxy A22 5G / Android 13, iPad Safari, and Windows PC.

## Demo-specific separation
1. Android applicationId is isolated from the upstream preview build.
2. Android Remote Config points to this repository's demo-v4.5.1/web/config.json.
3. Android app label and PWA name use Demo branding.
4. Service Worker cache namespace is separate.
5. Browser localStorage keys use a dedicated namespace to avoid conflicts with the original web app on the shared github.io origin.
6. The demo Pages workflow deploys this repository's web/ directory at the Pages root.
7. The source repository web and its Draft PR #12 were not modified.
