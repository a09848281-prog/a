# Remote Camera — Demo Repository

Android・iPad・Windows PC対応の遠隔監視カメラシステムのデモリポジトリです。

## 🚀 実際のデモ実装

実行可能なWeb・Android・GAS一式は、次のブランチにあります。

- **実装ブランチ：** [demo-v4.5.1](https://github.com/a09848281-prog/a/tree/demo-v4.5.1)
- **Webアプリ：** [web/](https://github.com/a09848281-prog/a/tree/demo-v4.5.1/web)
- **Androidアプリ：** [android/](https://github.com/a09848281-prog/a/tree/demo-v4.5.1/android)
- **GAS：** [gas/](https://github.com/a09848281-prog/a/tree/demo-v4.5.1/gas)
- **セットアップ：** [DEMO-SETUP-v4.5.1.md](https://github.com/a09848281-prog/a/blob/demo-v4.5.1/docs/DEMO-SETUP-v4.5.1.md)
- **GitHub Actions：** [ビルド・検証](https://github.com/a09848281-prog/a/actions)

GitHub Pagesの公開には、リポジトリの Settings → Pages → Build and deployment → Source を「GitHub Actions」に設定する必要があります。設定後、Deploy workflowが成功すると https://a09848281-prog.github.io/a/ でWebデモを開けます。

## 📘 統合仕様書

[開発引き継ぎ・統合仕様書 v3.8〜v4.5.1](docs/REMOTE-CAMERA-HANDOVER-v3.8-v4.5.1.md)

## 📌 ブランチの役割

- main：統合仕様書・リポジトリ案内を保持。
- demo-v4.5.1：v4.5.1を元にした独立デモ実装。
- 元の実装リポジトリ [a09848281-prog/web](https://github.com/a09848281-prog/web) とPR #12は変更していません。

## ⚠️ 注意

GitHub Actionsのソース検査・APKビルド成功は、Galaxy A22 5G / Android 13・iPad Safariの実機確認済みを意味しません。TURNなしのWebRTC接続、PWAバックグラウンド制限、人物候補検知の誤判定などの制限があります。カメラ・マイクはOSの権限・Foreground Service通知に従って利用します。
