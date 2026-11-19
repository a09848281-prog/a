# Remote Camera v4.5.1 Demo — セットアップ

## 1. 構成
- Web: GitHub Pages（このリポジトリの web/ を公開）
- Android: Kotlin / Foreground Service / WebRTC
- Signaling: GAS
- Database: Google Sheets
- Demo branch: demo-v4.5.1

## 2. GitHub Pages
初回はリポジトリの Settings → Pages → Build and deployment → Source を「GitHub Actions」に設定してください。GitHub ActionsのトークンにPagesサイト作成権限がない場合があるため、初回の有効化はリポジトリ設定画面で行います。ActionsでDeploy jobが成功してから次のURLを開きます。

https://a09848281-prog.github.io/a/

## 3. GAS
1. 現在のGAS Code.gsとスプレッドシートをバックアップします。
2. gas/Code.gs のAPI versionが4.5.1であることを確認します。
3. 既存DBのシート名・列が一致しているか確認します。
4. 既に動作しているGASを再利用する場合、何も変更せず既存/exec URLを利用できます。
5. GASを更新する必要がある場合は docs/GAS-REDEPLOY-v4.5.1.md を読み、新バージョンとしてデプロイします。
6. 初期化済みDBで setupDatabase() を安易に再実行しないでください。

Web URLは web/config.json にあります。Androidは android/.../Config.kt のRaw GitHub URLから、このブランチの web/config.json を取得します。

## 4. Android APK
1. Actionsから remote-camera-debug-apk-v4.5.1-demo を取得。
2. ZIPを展開し app-debug.apk をインストール。
3. アプリを開き、必要な権限を許可。
4. 「監視待機を開始」を押す。
5. Androidに出た8桁コードをWeb側へ入力。

applicationId: jp.a09848281.remotecamera.v451demo

## 5. Web / PWA
1. Safari / ChromeでデモURLを開く。
2. ペアリングコードを入力。
3. 接続を開始する。
4. 動体検知を利用する場合、映像上で検知範囲をドラッグして追加。
5. 通知を使う場合はボタンを押してOSの許可を選ぶ。

検知データは閲覧ブラウザー内に保存します。

## 6. 動作制約
- STUNのみ。TURNなし。
- iPadOSのバックグラウンド休止中は映像解析が止まる。
- 外部Pushなしのため休止中の通知を保証しない。
- 人物候補は簡易画像処理による推定。
- 実機テストは別途必要。
