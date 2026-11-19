# Demo v4.5.1 — 初期状態レポート

- Base repository: https://github.com/a09848281-prog/web
- Base branch: rebuild-v4.5.1-smart-motion-pwa
- Base commit: 0b61527973d69e01fab87df16524d50dcc452506
- Demo repository: https://github.com/a09848281-prog/a
- Demo branch: demo-v4.5.1
- Demo Android applicationId: jp.a09848281.remotecamera.v451demo
- Demo Android versionCode: 451
- Demo Android versionName: 4.5.1
- GAS API: 4.5.1
- Firebase: not added
- External ML APIs: not added

This file describes the initial source transfer and does not claim that CI, APK build, GitHub Pages deployment, or real-device testing has succeeded. Check GitHub Actions for the actual run results.

## Demo-specific changes
1. Android applicationId is isolated from the upstream preview build.
2. Android Remote Config points to this repository's demo-v4.5.1/web/config.json.
3. Android app label and PWA name use Demo branding.
4. Service Worker cache namespace is separate.
5. Browser localStorage keys are namespaced to avoid conflicts with the original web app on the shared github.io origin.
6. GitHub Actions workflows are adapted for this repository and branch.
7. Pages deployment publishes web/ at this repository's Pages root.
