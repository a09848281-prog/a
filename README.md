# Remote Camera v4.5.1 Demo

Android・iPad・Windows PC向けの遠隔監視カメラシステムです。v4.5.1の実装を元に、元リポジトリ a09848281-prog/web から独立したデモ環境として展開しています。

## 🌐 デモ
GitHub Pages: https://a09848281-prog.github.io/a/

デプロイ完了後にアクセスしてください。GitHub Pagesの初回有効化やActions実行が完了するまでは公開URLが利用できない場合があります。

## 📦 含まれるもの
- web/ — iPad / PC用Webアプリ（PWA）
- android/ — Kotlin / WebRTC Androidアプリ
- gas/ — Google Apps Script API
- docs/ — バージョン別仕様書・セットアップ・統合引き継ぎ仕様書
- .github/workflows/ — デモ専用の検証・APKビルド・Pages公開

## 📱 対応構成
- カメラ側：Android（Galaxy A22 5G / Android 13を主な検証対象）
- 操作側：iPad Safari / Windows PC Chrome等
- 映像・音声：WebRTC
- シグナリング・設定：GAS
- データ：Google Sheets
- 動体検知・人物候補：ブラウザー内Canvas処理
- Firebase / 外部AI人物認識API：新規導入なし

## 🚀 セットアップ

### Web
1. GitHub Pagesのデプロイが成功した後、上記デモURLを開く。
2. iPadはSafari共有メニューからホーム画面に追加できる。
3. 左側の「Androidをペアリング」から8桁コードを入力する。

### Android APK
1. GitHubのActionsを開く。
2. 「Validate and build Remote Camera v4.5.1 Demo」の最新成功実行を開く。
3. Artifact remote-camera-debug-apk-v4.5.1-demo をダウンロード。
4. ZIPを展開し、app-debug.apkをAndroidにインストール。
5. カメラ・マイク・通知など必要な権限を許可し、「監視待機を開始」を押す。
6. 表示された8桁コードをWeb側へ入力する。

Android applicationIdは jp.a09848281.remotecamera.v451demo です。元のv4.5.1 Previewとは別アプリとしてインストールできます。

### GAS
Web設定 web/config.json は既存のv4.5.1 GAS /exec URLを参照しています。別のGASを利用する場合は、ここを変更し、同じURLを読み込むAndroid設定を確認してください。既に初期化済みのSheetsに対し、確認なしでsetupDatabase()を再実行しないでください。

詳細は docs/DEMO-SETUP-v4.5.1.md を参照してください。

## 🔎 開発・検証
- ブランチ：demo-v4.5.1
- 基準：a09848281-prog/web の rebuild-v4.5.1-smart-motion-pwa
- 元PR #12：https://github.com/a09848281-prog/web/pull/12
- デモ用APKは専用applicationIdでビルドする。
- Pagesはこのリポジトリの web/ を公開する。
- 実機未検証項目は docs/TEST-PLAN-v4.5.1.md を確認する。

## ⚠️ 制限
- TURNサーバー未構成のため、ネットワークによってWebRTC接続が失敗することがある。
- iPadOSがPWAをバックグラウンド休止すると映像解析は停止する。
- PWA休止中・終了中の即時通知は保証しない。
- 人物候補検知は形状に基づく概算で、人物の本人特定や正確な人数判定ではない。
- Androidの強制停止・再起動直後・権限取消後の遠隔起動は保証しない。
- GitHub Actionsのビルド成功は実機動作確認済みを意味しない。

## 🔐 安全・プライバシー
カメラ・マイクはAndroid OSの権限とForeground Service通知に従って利用します。ユーザーの操作・許可なしの隠れた撮影を目的としません。検知画像・検知範囲・検知履歴は閲覧ブラウザー内に保存され、GASへ送信しません。
