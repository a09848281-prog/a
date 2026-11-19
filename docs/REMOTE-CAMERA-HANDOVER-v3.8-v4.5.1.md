# Android・iPad・PC対応 遠隔監視カメラシステム
## 開発引き継ぎ・統合仕様書 v3.8〜v4.5.1

- 作成日：2026年11月19日
- 文書種別：開発引き継ぎ・統合仕様書
- 実装リポジトリ：[`a09848281-prog/web`](https://github.com/a09848281-prog/web)
- 仕様書・デモ管理リポジトリ：[`a09848281-prog/a`](https://github.com/a09848281-prog/a)
- 対象バージョン：v3.8 / v3.9 / v4.0 / v4.1 / v4.2 / v4.2.1 / v4.2.2 / v4.5 / v4.5.1
- 用途：個人利用を中心とした遠隔監視カメラシステム
- 引き継ぎ時の最重要方針：**既存版を壊さない・mainへ勝手にマージしない・実機未検証を動作確認済みと表現しない**

---

## 0. この文書の読み方・情報の確度

本書では情報を次の3区分で扱う。

| 区分 | 意味 |
|---|---|
| **コード確認済み** | GitHub上の対象ブランチにある実ファイルを取得し、設定値・主要処理を確認したもの |
| **仕様・開発記録** | 既存仕様書、PR説明、ビルドレポート、ユーザーから引き継いだ履歴に記載されたもの |
| **実機未検証** | ソースやCI上の準備は確認できるが、実端末で正常動作したとは確認できていないもの |

GitHub Actionsの成功は、APK生成や静的検査が成功したことを示すものであり、Galaxy A22 5G / Android 13、iPad Safari、Windows PCでの実機動作を保証しない。

v4.3・v4.4の開発資料・ブランチは確認できていない。欠けているバージョンの仕様を推測して埋めてはならない。

---

# 1. プロジェクト概要

## 1-1. 目的

Androidスマートフォンを監視カメラとして利用し、iPad・Windows PCなどのWebブラウザーから遠隔操作・映像確認・音声確認を行う。

単なるカメラ配信ではなく、次の機能を統合する。

- 遠隔ライブ映像の確認
- Androidカメラの遠隔操作
- Androidマイクの音声受信
- iPad・PCからAndroidへの音声送信（PTT）
- 複数ブラウザーからの同時監視
- 動体検知・人物候補検知
- 位置情報・地図・履歴
- カメラライト・ズーム・画角・FPS操作
- Android Foreground Serviceによる待機
- 通信量・接続状態・端末情報の表示
- ブックマークレットからの利用
- PWAとしてのホーム画面追加・アプリシェルキャッシュ

## 1-2. 対象端末

| 役割 | 端末・環境 |
|---|---|
| カメラ側 | Galaxy A22 5G |
| カメラ側OS | Android 13 |
| 操作・監視側 | iPad |
| 操作・監視側 | Windows PC |
| Webブラウザー | Safari / Chromeなど |

## 1-3. 基本方針

- Android側はKotlinによるネイティブアプリ。
- Web側はHTML / CSS / JavaScript。
- Web公開はGitHub Pages。
- シグナリング・管理通信はGoogle Apps Script（GAS）。
- データ管理はGoogle Sheets。
- 映像・音声通信はWebRTC。
- Firebaseは使用しない。
- アカウント登録・ログインを基本機能にしない。数字によるペアリングを使う。
- 有料外部サービスや追加APIへの依存をできるだけ避ける。
- コードの安定性を優先し、既存機能を壊さない。
- 既存バージョンを保持し、新しい版は独立ブランチで検証する。
- カメラ・マイクの利用はユーザーが許可・開始した状態で行い、OSの権限・通知・Foreground Service制約に従う。
- OSの強制停止や権限状態を回避して、隠れてカメラ・マイクを起動する設計にはしない。

---

# 2. システム構成

## 2-1. 全体構成

### Androidアプリ

- カメラ取得
- マイク取得
- ライト制御
- 位置情報取得
- Foreground Service
- WebRTC映像・音声送信
- Webからの遠隔操作受信
- 接続状態・電池・通信量などの情報提供

↓

### WebRTC

↓

### iPad / PC Webアプリ

- ライブ映像表示
- 音声再生
- PTT送話
- カメラ操作
- 動体検知
- 人物候補検知
- 地図・位置情報表示
- 検知履歴・画像表示
- 複数閲覧端末の管理

### 別経路：管理・シグナリング通信

Android / Web
↓
GAS
↓
Google Sheets

GASの主な役割：

- Android端末の初期登録
- ペアリング情報管理
- 閲覧端末トークン管理
- セッション情報管理
- WebRTCシグナリング
- 接続状態管理
- 遠隔操作コマンドの受け渡し
- 設定情報の保存・取得
- 位置情報・履歴関連データ管理

## 2-2. 技術構成

| 部分 | 技術 |
|---|---|
| Android | Kotlin |
| Androidビルド | Gradle / Android Studio |
| Web | HTML / CSS / JavaScript |
| Web公開 | GitHub Pages |
| 映像・音声 | WebRTC |
| シグナリング | GAS |
| データベース | Google Sheets |
| 地図 | Leaflet / OpenStreetMap |
| PWA | Web App Manifest / Service Worker |
| 動体検知 | Canvas画像差分処理 |
| 人物候補検知 | 簡易画像処理・形状推定 |

v4.5.1の新規検知機能には、Firebase、TensorFlow、MediaPipe、外部AI人物認識APIなどを追加しない。既存v4.2.2から継承した地図機能はLeaflet CDN（unpkg）とOpenStreetMapタイルを引き続き利用するが、今回の動体・人物候補検知処理には使用しない。

---

# 3. バージョンごとの開発履歴

## 3-1. Remote Camera v3.8.0

ブランチ：`rebuild-v3.8-realtime-stability-no-firebase`

### 主な目的
WebRTCの接続安定性とWebからの遠隔操作を改善する。

### 実装・変更内容
1. WebRTC DataChannelによるAndroidへの直接操作。
2. カメラ切替。
3. 画質設定。
4. 位置情報設定。
5. 遠隔切断。
6. ICEの一時的な切断に対する8秒間の復旧待機。
7. WebRTC接続中のGASポーリング停止。
8. DataChannel切断時のGAS経由フォールバック。
9. 映像フレームの受信監視。
10. 映像停止・フリーズ時の再接続。
11. Web Audioによる受信音量調整。
12. Android位置情報のキャッシュ利用。
13. GAS読み取り処理の軽量化。
14. GASエラー時の再試行。
15. 30秒・6秒・2秒の待機モード。
16. GAS APIバージョン照合。

### 重要な設計
WebRTCが正常に接続されている間はGASへの不要なポーリングを停止する。接続が切れた場合はGAS経由の通信へ戻す。

### 制限
- STUNのみ。TURNは未構成。
- 異なるネットワーク間の接続は保証されない。
- Android 13実機とiPad Safariでの動作確認が必要。

## 3-2. Remote Camera v3.9.0

ブランチ：`rebuild-v3.9-camera-controls-stability`

### 主な目的
カメラ操作・画角・FPS・音声・バックグラウンド動作を改善する。

### カメラ機能
- 画角：16:9 / 4:3 / 1:1 / 9:16
- FPS：15 / 20 / 24 / 30
- 画質：360p / 480p / 720p / 1080p
- 通信量モード
- 自動画質調整
- 広角カメラ優先

### 操作機能
- Androidライトの遠隔操作
- PNG静止画保存
- 画面スリープ防止
- 電池残量表示
- 端末温度表示

### 音声関連
- AndroidのJavaAudioDeviceModule設定
- 音声入力経路の調整
- AEC / NS設定
- 音声モードの明示

### バックグラウンド関連
- Android ActivityとForeground Serviceを分離。
- アプリ設定画面を開いてもサービスが止まらない構成。
- バックグラウンド通信量モードを追加。

### 注意
配信中に変更可能な設定と、次回接続時に適用される設定を分ける。FPS・ビットレートは接続中の変更を優先し、解像度・画角などは変更に失敗した場合に既存映像を維持する。

## 3-3. Remote Camera v4.0

ブランチ：`rebuild-v4.0-ambient-audio-ptt`

### 主な目的
環境音の受信と双方向音声操作を強化する。

### 追加機能
1. Androidの環境音優先マイク。
2. Web受信音量0〜200%。
3. Androidマイク受信レベルメーター。
4. 大型プッシュトークボタン。
5. 送話ロック。
6. Androidからの音声ミュート。
7. 音声レベル・受信状態の表示。

### 音声仕様
Android側の環境音を優先して取得する。Web側で音声を増幅できるが、200%設定はAndroidマイクの物理的な感度を上げるものではない。環境音優先モードでは周囲の雑音やエコーも入りやすくなる。

## 3-4. Remote Camera v4.1.0

ブランチ：`rebuild-v4.1-complete-stable`

### 主な目的
v3.9とv4.0の機能を統合する。

### 継承機能
- カメラ画角切替
- FPS切替
- 画質切替
- ライト
- ズーム
- PNG静止画
- 位置情報
- 地図
- 履歴
- Foreground Service
- 自動再接続
- 通信量モード
- 接続統計
- GASリトライ

### 音声機能
Androidマイクモード：
- 環境音重視
- バランス
- 声重視

Web側：
- 受信音量0〜200%
- ミュート
- 音声メーター
- 大型PTT
- 送話ロック
- 3分安全自動解除
- Androidスピーカー音量0〜100%

### 重要事項
PTTの送話ロックには安全タイムアウトを設ける。Android側の強制停止・再起動直後・権限取消などからの遠隔起動は保証しない。

## 3-5. Remote Camera v4.2.0

ブランチ：`rebuild-v4.2-multi-viewer-camera`

### 主な目的
1台のAndroidカメラを複数のWebブラウザーから同時に監視できるようにする。

### 複数端末監視
- Android 1台に対して最大3台の閲覧端末。
- 各閲覧端末は独立したPeerConnection。
- 閲覧端末ごとに登録・認証・切断。
- 1台を切断しても他の閲覧端末は維持する。
- 閲覧端末一覧を表示する。
- 端末名・オンライン状態・最終応答時刻を表示。

### 動体検知
- OFF / 低 / 標準 / 高。
- ブラウザー内で映像を解析。
- 動体検知イベントを保存。
- 検知画像を保存。
- CSV出力。
- 利用者が許可した場合の通知。

### 画像ビューアー
- 受信画像を全画面表示。
- ライブ映像の静止画表示。
- 動体検知履歴画像の全画面表示。
- 画像保存。
- 閉じる操作。
- ブックマークレット利用時の親パネル拡張・復帰。

### 接続安定性
- ICE候補収集のタイムアウト対策。
- PeerConnection単位のエラー処理。
- 4台目の接続拒否。
- 不要な再試行の防止。
- LocalStorageの例外対策。
- 画像Object URLの解放。

## 3-6. Remote Camera v4.2.1

ブランチ：`rebuild-v4.2-ui-layout-stability`

### UI変更
画面を左右に分離する。

左側：操作・設定
- 接続
- 切断
- PTT
- 音量
- 全画面操作
- 動体検知
- 地図操作
- 各種設定

右側：表示
- ライブ映像
- 地図
- 受信画像

右側は映像・地図などの表示を中心とする。Android端末情報など、一部の情報表示は右側にも配置可能。

### 重要事項
画像ビューアーの全画面表示を維持する。v4.2本体の機能・API互換性を壊さない独立ブランチとして検証する。

## 3-7. Remote Camera v4.2.2

ブランチ：`rebuild-v4.2.2-ultra-stable-controls`

### 主な目的
操作性・画質・音声・通信情報を改善する。

### 追加機能
1. 左側操作・右側表示レイアウトの安定化。
2. 右上にAndroid端末情報をコンパクト表示。
3. 監視経過時間表示。
4. 通信量表示。
5. 接続中の画角変更を試行。
6. 60fps選択。
7. 受信音声フィルター。
8. ライト操作の多重実行防止。
9. ライト操作失敗時の再試行。
10. 位置情報キャッシュの改善。

### FPS
選択可能なFPS：15 / 20 / 24 / 30 / 60

60fps非対応時は30 / 24 / 20 / 15fpsへのフォールバックを試みる。実際に選択・配信できるFPSは端末・カメラ・エンコーダーに依存する。

### 音声フィルター
Web Audio APIによるEQ・コンプレッサー・ソフトゲートを利用する。AI音声分離ではない。

### 通信量
AndroidのUID単位TrafficStatsから概算値を取得し、Web側でもWebRTC統計を表示する。完全な通信量計測ではなく概算として扱う。

## 3-8. Remote Camera v4.5.0

ブランチ：`rebuild-v4.5-motion-zones-white-ui`

### 主な目的
監視カメラ向けの動体検知機能を拡張する。

### 動体検知範囲
- 映像上をドラッグして検知範囲を作成。
- 最大8範囲。
- 範囲ごとの有効・無効。
- 範囲の削除。
- 範囲の相対座標保存。
- 範囲内のみを検知対象にする。

### 検知方式
ブラウザー内のCanvasによる画素差分処理。AI・機械学習・外部画像解析APIは使用しない。

### 履歴・通知
- 動体検知イベント保存。
- 検知画像保存。
- 画面内通知。
- 対応環境でのOS通知。

### UI
白基調のデザイン。左側設定・右側映像の構成を維持する。

### プライバシー
検知範囲・イベント画像・履歴は閲覧ブラウザーのローカル領域に保存する。GAS・Google Sheetsへ検知画像を送信しない。

## 3-9. Remote Camera v4.5.1

ブランチ：`rebuild-v4.5.1-smart-motion-pwa`

PR：[Draft PR #12](https://github.com/a09848281-prog/web/pull/12)

### 位置づけ
v4.2.2を基礎にした独立プレビュー版。v4.2.2の既存機能を維持しつつ、動体検知・人物候補検知・PWA機能を追加する。

### 動体検知範囲
- 最大8範囲。
- 映像上をドラッグして範囲指定。
- 範囲ごとの名前設定。
- 範囲ごとの感度設定。
- 範囲ごとの有効・停止。
- 範囲の削除。
- 初回の手動範囲作成時に初期全画面範囲を置き換える。
- ズーム状態を考慮した範囲座標変換。
- 座標は映像に対する0〜1の正規化座標として保存する。

### 動体検知処理
- 受信映像を縮小してCanvas上で画素差分を比較する。
- 既存仕様書では処理用映像を160×90へ縮小。
- 指定範囲内だけを検知対象にする。
- 大きな全画面変化は照明変化などとして抑制する。
- 検知時刻・範囲名・変化率・イベント種別・縮小JPEGをブラウザー内に保存する。
- 既存仕様書に記載された履歴上限は最大20件。
- CSV出力に対応する。

### 人物候補検知
機械学習ではなく、簡易的な画像処理による推定。
- 縦長の動き領域候補を検出。
- 人物らしい形状を候補として扱う。
- 候補数の概算。
- 移動方向の推定。
- 検知範囲への進入イベント。
- 検知範囲からの退出イベント。

判定の補助要素：
- 差分領域の連結成分
- 面積
- 縦横比
- 動きの連続性
- 移動方向
- 検知範囲との位置関係
- 元映像の縦横比を考慮した形状補正

本人特定・顔認識・属性推定・個人追跡は行わない。影・荷物・家具・扉などを人物候補と誤判定する可能性がある。候補数は実際の人数を保証しない。

### 検知履歴
- 検知時刻
- 検知画像
- 検知範囲名
- 検知イベント種別
- 人物候補数の概算
- 移動方向
- 感度情報
- CSV出力

### 保存先
検知画像・検知範囲・履歴は閲覧ブラウザー内に保存する。GAS・Sheetsへ送信しない。

### PWA
- Web App Manifest。
- Service Worker。
- 同一オリジンのアプリシェルキャッシュ。
- ホーム画面追加への対応。
- ページ動作中のローカル通知。
- Service WorkerのPushイベントは使用しない。

### UI
- 白基調。
- 左側に設定・操作。
- 右側に映像。
- 映像上に検知範囲を表示。
- PCでは左右2列、狭い画面ではレスポンシブ表示を維持する。

### バックグラウンド制限
iPadOSがPWAをバックグラウンド休止すると映像解析が停止する。外部Pushサービスを使用しないため、PWA休止中・終了中の即時通知は保証しない。復帰後にローカル履歴を確認する。

---

# 4. v4.5.1の機能一覧

| 分類 | 機能 | 状態・備考 |
|---|---|---|
| 映像 | ライブ映像受信 | 既存WebRTC実装を継承 |
| 映像 | 画質設定 | 360p / 480p / 720p / 1080pの設定系統 |
| 映像 | 画角設定 | 16:9 / 4:3 / 1:1 / 9:16 |
| 映像 | FPS設定 | 15 / 20 / 24 / 30 / 60の選択系統。実効値は端末依存 |
| 映像 | ズーム | 既存実装を継承 |
| 映像 | 全画面表示 | iPad Safariの復帰処理を含む |
| 映像 | 静止画保存 | PNG等の画像保存 |
| カメラ | 前面・背面切替 | Android側で切替 |
| カメラ | ライト操作 | Android側で実行 |
| 音声 | Androidマイク受信 | 環境音 / バランス / 声重視 |
| 音声 | 受信音量 | Web側の音量調整 |
| 音声 | ミュート | 対応 |
| 音声 | 音声フィルター | Web Audio API。AI分離ではない |
| 音声 | PTT | WebからAndroidへの送話 |
| 音声 | 送話ロック | 3分安全自動解除 |
| 音声 | Androidスピーカー音量 | 対応する制御範囲で操作 |
| 接続 | 8桁ペアリング | 10分期限の一回使用コード |
| 接続 | WebRTC | 映像・音声の主通信 |
| 接続 | 自動再接続 | 既存実装を継承 |
| 接続 | 複数端末監視 | 最大3閲覧端末を想定 |
| 接続 | 接続状態表示 | 対応 |
| 監視 | 動体検知 | Canvas画素差分 |
| 監視 | 最大8検知範囲 | 範囲ごとの管理 |
| 監視 | 範囲別感度 | 低 / 標準 / 高 |
| 監視 | 人物候補検知 | 簡易形状推定 |
| 監視 | 進入・退出イベント | 範囲との関係で推定 |
| 監視 | 移動方向推定 | 概算 |
| 監視 | 検知履歴 | ブラウザー内保存 |
| 監視 | 検知画像保存 | ブラウザー内保存 |
| 監視 | CSV出力 | 対応 |
| 位置情報 | Android位置情報 | 権限・設定に依存 |
| 位置情報 | 地図 | Leaflet / OpenStreetMap |
| 位置情報 | 履歴 | GAS / Sheets側に保存する既存機能 |
| 端末情報 | 電池・温度 | Androidからの取得可否に依存 |
| 端末情報 | 通信量 | 概算表示 |
| 端末情報 | 監視経過時間 | 表示 |
| Web | PWA | Manifest / Service Worker |
| Web | Service Worker | アプリシェルキャッシュ |
| Web | ブックマークレット | パネル利用に対応する既存機能 |

※バージョン間で実装状況が異なる。表の全機能がv4.5.1で実機動作確認済みという意味ではない。

---

# 5. ペアリング・接続仕様

## 5-1. 基本方式

- ログイン・新規登録を基本機能にしない。
- Android側で8桁のペアリングコードを発行する。
- Web側で数字を入力してペアリングする。
- ペアリングコードの有効期限は10分。
- Web側はブラウザー内にcontrollerTokenを保持する。
- 再ペアリング・解除時は既存トークンを無効化する。
- ペアリングコードは一回使用すると消費する。
- GAS側にはペアリング試行回数制限がある。

## 5-2. 実コードで確認した主なGAS定数

`gas/Code.gs` で次の設定を確認した。

| 項目 | 値 |
|---|---:|
| 最大閲覧端末数 | 3 |
| ペアリングコード有効時間 | 10分 |
| セッション保持期間の定数 | 7日 |
| コマンド有効時間 | 3分 |
| シグナリング有効時間 | 3分 |
| ペアリング失敗上限 | 10分間に25回 |
| Android初回登録上限 | 1時間に5回 |

## 5-3. 通信方式

通常の映像・音声：WebRTC

制御・シグナリング：GAS + Google Sheets

WebRTC DataChannelが利用できる場合は遠隔操作を優先して直接送信する。DataChannelが利用できない場合はGAS経由へフォールバックする。

## 5-4. 複数端末監視

v4.2以降は最大3台の閲覧端末を想定する。
- 閲覧端末ごとにPeerConnectionを管理する。
- Android側では閲覧端末ごとのDataChannelを管理する。
- 1台の閲覧端末の切断で、他の閲覧端末の接続を終了させない。
- 4台目の接続要求は拒否する。
- 閲覧端末ごとに認証・解除を行う。

---

# 6. Androidバックグラウンド動作

Android側はForeground Serviceを利用して監視待機を行う。

## 6-1. 目的

- アプリ画面を閉じても、OSが許可する範囲で待機を維持する。
- 画面OFF中の待機を可能な範囲で維持する。
- Web側からの接続要求に応答する。
- カメラ・マイクを使う際はOSの権限と制約に従う。
- 通知領域にサービス稼働状態を表示する。

## 6-2. Android側の実装確認項目

v4.5.1ブランチで次の実装を確認した。

- `MainActivity.kt`：待機モード、遠隔カメラ・マイク許可、CPU維持、音声取得モード等の設定UI。
- `MonitorService.kt`：Foreground Service、サービス通知、監視待機、GASポーリング、遠隔操作の処理。
- `RtcEngine.kt`：WebRTC、複数PeerConnection、映像・音声トラック、DataChannel、カメラ切替、ライト操作。
- `ApiClient.kt`：GAS HTTP通信、タイムアウト、限定的な再試行、リダイレクト先ホストの検査。
- Android Manifest：camera / microphone / location / foreground service関連の権限宣言。

MainActivityには待機頻度として、省電力30秒・標準6秒・応答優先2秒の選択肢がある。

Android側には「Webからカメラ・マイクの起動を許可」の設定があり、既定値は有効。画面OFF中のCPU維持は別設定で、既定値は無効。これらの設定をユーザーが確認して利用する。

## 6-3. 必要な権限

- INTERNET
- ACCESS_NETWORK_STATE
- CAMERA
- RECORD_AUDIO
- WAKE_LOCK
- ACCESS_FINE_LOCATION
- ACCESS_COARSE_LOCATION
- ACCESS_BACKGROUND_LOCATION（利用する場合）
- POST_NOTIFICATIONS
- FOREGROUND_SERVICE
- FOREGROUND_SERVICE_CAMERA
- FOREGROUND_SERVICE_MICROPHONE
- FOREGROUND_SERVICE_LOCATION

実際に要求する権限はOSバージョン・機能・ユーザー設定によって異なる。宣言されていることと、ユーザーが許可済みであることを混同しない。

## 6-4. 保証できないこと

次の状態では遠隔からカメラを必ず起動できるとは限らない。

- Androidの強制停止
- 端末再起動直後
- 権限取消
- バッテリー最適化
- メーカー独自のバックグラウンド制限
- OSによるカメラ・マイク利用制限
- Foreground Service開始条件を満たさない場合

Androidの仕様に反する常時起動や、ユーザーに分からないカメラ・マイク利用を前提にしない。

---

# 7. 動体検知・人物候補検知の詳細方針

## 7-1. 動体検知

Canvasで受信映像を縮小したフレームを取得し、前フレームとの差分を比較する。画素の変化量を計算し、設定された感度を超えた場合に動体イベントとする。

- 範囲は最大8個。
- 範囲座標は映像に対する正規化座標で保存する。
- ズーム・全画面・画面サイズ変更を考慮して描画座標を変換する。
- 初回の手動範囲を作ると初期全画面範囲を置き換える。
- 範囲ごとに名前・感度・有効/停止を保持する。
- 照明変化などによる誤検知を抑制する処理を含む。

## 7-2. 人物候補検知

機械学習モデルを導入せず、形状や動きの特徴から人物らしい領域を推定する。

判定の例：
- 縦横比
- 動き領域の大きさ
- 領域の連続性
- 移動方向
- 検知範囲との位置関係

この処理は人物の確定判定ではなく、人物候補である。本人特定・顔認識・個人識別・属性推定・個人追跡は行わない。

影・荷物・家具・扉などを人物と誤判定する可能性がある。候補数は実際の人数を保証しない。

## 7-3. 履歴・プライバシー

- 検知画像・検知範囲・検知履歴は閲覧ブラウザー内に保存する。
- 検知画像をGAS / Google Sheetsへ送信しない。
- ローカル保存の容量・ブラウザー削除・プライベートブラウズ等の影響を受ける。
- CSVには検知時刻・範囲名・イベント種別・候補数概算・方向等を含める。
- 保持件数を増やす場合はLocalStorage容量と画像サイズを確認する。

## 7-4. 将来の拡張方針

機械学習を導入する場合は、現在の簡易処理と別機能として設計する。外部APIを使用しない方針を維持する場合は、ブラウザー内・端末内で動作するモデルを検討する。ただし、端末負荷・メモリ・処理速度・iPad Safariとの互換性を検証してから導入する。

---

# 8. UI設計方針

## 8-1. 左側：操作・設定

- 接続
- 切断
- ペアリング
- カメラ設定
- FPS
- 画質
- 画角
- 音声設定
- PTT
- ライト
- 動体検知
- 人物候補検知
- 検知履歴
- 地図操作
- 詳細設定

## 8-2. 右側：映像・情報表示

- ライブ映像
- 地図
- Android端末情報
- 監視経過時間
- 通信量
- 接続状態
- 検知範囲オーバーレイ
- 受信画像

## 8-3. 重要なUIルール

- 左側は設定・操作を中心にする。
- 右側は映像・地図を中心にする。
- 映像の表示領域をできるだけ広くする。
- 全画面から戻った後に映像が停止しないようにする。
- iPadとPCの両方で操作できるようにする。
- モバイル表示ではレスポンシブレイアウトを維持する。
- 設定項目を増やす場合は詳細設定へ整理する。
- 既存の操作位置を勝手に変更しない。
- 白基調のデザインを維持する。

---

# 9. ブックマークレット対応

Webアプリをブックマークレット経由でも利用できるようにする。

- Webアプリの機能を可能な範囲で維持する。
- パネル表示に対応する。
- 画像ビューアーを全画面表示できるようにする。
- 全画面終了時は元のパネルサイズに戻す。
- 通常のWeb利用に悪影響を与えない。
- 他サイトの既存機能と干渉しないようにする。
- 利用できないブラウザーAPI・権限制限は通常のWebアプリと区別する。
- 親パネルへの通知・復帰処理を行う場合は、既存ページのグローバル変数やCSSと衝突しないようにする。

---

# 10. GAS・Google Sheets

## 10-1. GASの役割

- 端末初期登録
- ペアリング
- 閲覧端末トークン管理
- WebRTCシグナリング
- セッション管理
- 接続状態
- 遠隔操作コマンド
- 設定保存・取得
- 位置情報関連処理

## 10-2. v4.5.1 GASで確認したシート構成

`gas/Code.gs` の `TABLES` 定義：

| シート | 列 |
|---|---|
| Devices | deviceId, ownerUserId, deviceName, deviceSecretHash, codeHash, codeExpiresAt, createdAt, registeredAt, lastSeen, status, controllerSecretHash |
| Commands | commandId, deviceId, ownerUserId, command, payloadJson, createdAt, expiresAt, status |
| Signals | seq, sessionId, deviceId, sender, recipient, kind, sdp, candidateJson, createdAt, expiresAt |
| Settings | deviceId, settingsJson, updatedAt |
| LocationHistory | locationId, userId, deviceId, lat, lng, accuracy, timestamp, mode |
| SecurityLog | timestamp, userId, deviceId, eventType, success, detail |
| Controllers | controllerId, deviceId, controllerName, tokenHash, createdAt, lastSeen, status |

既存シート名・列・データは重要な互換性要素である。新機能でDBを変更する場合は、移行手順・後方互換性・バックアップを先に設計する。

## 10-3. v4.5.1で確認したAPIアクション

- health
- device_begin
- device_refresh_code
- claim_device
- unpair_device
- get_controllers
- viewer_ping
- device_poll
- send_signal
- poll_signals
- send_command
- ack_command
- get_settings
- save_settings
- add_location
- get_locations
- delete_locations

既存アクション名を変更・削除する場合はAndroid・Webの両側を同時に確認し、APIバージョン整合性を検査する。

## 10-4. データ保護

- スプレッドシートを公開しない。
- 管理情報や秘密情報をフロントエンドに直接置かない。
- トークンはハッシュ化してGAS側で照合する。
- 不要な位置情報を監査ログへ記録しない。
- 検知画像はGASへ送信しない。
- 位置情報とセキュリティ監査ログを混同しない。
- GASのログやエラー表示に秘密情報・トークンを含めない。

## 10-5. setupDatabase() の扱い

v4.5.1のGASコードでは、既存行を消去せず、必要なシートを作成し、末尾の不足列を追加し、既存ヘッダーの不一致があればデータ保護のため中止する実装を確認した。旧Users / Sessionsシートが存在する場合も削除しない。

ただし、既に利用中のDBに対しては、引き継ぎ時点の運用ルールに従い、**バックアップとシート構成の確認なしにsetupDatabase()を再実行しない**。GASを更新する場合は以下を守る。

1. 現在のCode.gsをバックアップ。
2. v4.5.1ブランチの `gas/Code.gs` を確認。
3. 既存シート名・列・データを確認。
4. 初期化済みDBではsetupDatabase()の再実行を避ける。
5. デプロイ管理から既存Webアプリを編集。
6. 新バージョンとしてデプロイ。
7. 既存の/exec URLを維持。
8. Web側・Android側の接続とペアリングを確認。

---

# 11. 現在のGitHub開発状態

## 11-1. 実装リポジトリ

[`https://github.com/a09848281-prog/web`](https://github.com/a09848281-prog/web)

主要ブランチ：

| バージョン | ブランチ |
|---|---|
| v3.8 | rebuild-v3.8-realtime-stability-no-firebase |
| v3.9 | rebuild-v3.9-camera-controls-stability |
| v4.0 | rebuild-v4.0-ambient-audio-ptt |
| v4.1 | rebuild-v4.1-complete-stable |
| v4.2 | rebuild-v4.2-multi-viewer-camera |
| v4.2.1 | rebuild-v4.2-ui-layout-stability |
| v4.2.2 | rebuild-v4.2.2-ultra-stable-controls |
| v4.5 | rebuild-v4.5-motion-zones-white-ui |
| v4.5.1 | rebuild-v4.5.1-smart-motion-pwa |

v4.3のブランチ・仕様書・ビルドレポートは確認できていない。v4.4についても確認できる開発記録はない。これらの仕様を推測して補完しないこと。

## 11-2. v4.5.1 Draft PR #12

- PR：[Remote Camera v4.5.1 Preview](https://github.com/a09848281-prog/web/pull/12)
- ブランチ：`rebuild-v4.5.1-smart-motion-pwa`
- ベース：`rebuild-v4.2.2-ultra-stable-controls`
- Head commit：`0b61527973d69e01fab87df16524d50dcc452506`
- Base commit：`616c3819115883b53b308e588f79c159acb3620d`
- PR状態：Open / Draft / 未マージ
- 確認時の変更ファイル数：19
- 確認時のコミット数：10

**重要：v4.5.1をmainへマージしない。** 既存main・v4.2・v4.2.1・v4.2.2を上書きしない。

## 11-3. v4.5.1の主な変更ファイル

PR #12で確認した変更ファイル：

- `.github/workflows/deploy-pages-v451-preview.yml`
- `.github/workflows/validate-v451.yml`
- `android/app/build.gradle.kts`
- `android/app/src/main/AndroidManifest.xml`
- `android/app/src/main/java/jp/a09848281/remotecamera/Config.kt`
- `docs/BUILD-REPORT-v4.5.1.md`
- `docs/GAS-REDEPLOY-v4.5.1.md`
- `docs/QUICKSTART-v4.5.1.md`
- `docs/REMOTE-CAMERA-v4.5.1-SPEC.md`
- `docs/TEST-PLAN-v4.5.1.md`
- `gas/Code.gs`
- `web/app.css`
- `web/app.js`
- `web/config.json`
- `web/icon-192.svg`
- `web/icon-512.svg`
- `web/index.html`
- `web/manifest.webmanifest`
- `web/sw.js`

Androidの撮影・WebRTC・Foreground Serviceの主要ロジックはv4.2.2からの継承部分を含む。v4.5.1ではアプリ識別子とリモート設定取得先などをプレビュー用に分離している。

## 11-4. v4.5.1 Webプレビュー

[https://a09848281-prog.github.io/web/v4.5.1-preview/](https://a09848281-prog.github.io/web/v4.5.1-preview/)

## 11-5. v4.5.1 GAS URL・Web設定

ファイル：`web/config.json`

```json
{
  "gasUrl": "https://script.google.com/macros/s/AKfycbz2B2DzvtiM9WugX_ZdKulLbbm9M5sSkoE0C3vPzqR0C7IoYm5LEritRccFwuExyX7i/exec",
  "appVersion": "4.5.1",
  "pollIntervalMs": 2500,
  "requestTimeoutMs": 12000
}
```

Android側の `Config.kt` は、v4.5.1ブランチの `web/config.json` をRaw GitHub URLで参照する。

GAS URLを変更する場合は、まずWeb側の `web/config.json` を変更し、Android側が参照するリモート設定との整合性を確認する。設定URLを変更しない範囲では、GAS URLの変更だけを理由にAPKの再ビルドが必要とは限らない。実際の取得タイミング・キャッシュ動作はAndroidコードで確認する。

## 11-6. Android APK識別情報

v4.5.1の `android/app/build.gradle.kts` で確認した値：

| 項目 | 値 |
|---|---|
| namespace | jp.a09848281.remotecamera |
| applicationId | jp.a09848281.remotecamera.v451preview |
| versionCode | 451 |
| versionName | 4.5.1 |
| minSdk | 26 |
| targetSdk | 35 |
| compileSdk | 35 |
| Java / Kotlin target | 17 |

applicationIdが別のため、端末環境が許せば旧版と並行インストールできる。v4.5.1では新しい8桁コードで再ペアリングする。

## 11-7. APK・Actions情報の不一致に関する注意

GitHub上で取得した情報には、ビルド実行ID・Artifact IDの記載に不一致がある。

| 情報源 | 記載されたActions実行ID | Artifact ID |
|---|---:|---:|
| ユーザーから引き継いだ情報 | 44527994518 | 記載なし |
| PR #12本文 | 44527241766 | 14532870645 |
| `docs/QUICKSTART-v4.5.1.md` | 44527714275 | 14531357847 |
| `docs/BUILD-REPORT-v4.5.1.md` | 44526944518 | 14533225223 |

これらが同一ビルドを指すとは限らない。APKをダウンロードする際は、GitHub Actions画面で実際に成功した実行を開き、対象ブランチ・実行日時・Artifact名・Artifact IDを確認すること。仕様書内の番号だけを根拠にArtifactを決めない。

Build ReportとPR本文のArtifact期限記載は2026年12月19日となっている。文書作成日（2026年11月19日）時点では未来の日付であるため、実際のダウンロード可否はGitHub Actions画面で確認する。

---

# 12. v4.5.1の検証状況

## 12-1. GitHub Actions / ソース検査

開発記録とワークフローには次の検査・ビルド項目がある。

- JavaScript構文検査
- GAS構文検査
- JSON構文検査
- HTML ID重複検査
- JavaScript DOM参照検査
- 左右レイアウト検査
- APIバージョン整合性
- Android Gradleビルド
- APK成果物生成
- PWA関連ファイル検査
- Service Worker関連検査
- 動体検知・範囲関連のソース検査
- Android設定URL・applicationId・version情報の整合性

Actions成功は、これらの自動検査・ビルドが通ったという意味である。すべてのUI操作・カメラ・マイク・通知が実機で正常動作した証明ではない。

## 12-2. 実機試験が必要な項目

### Galaxy A22 5G / Android 13
- APK並行インストール
- カメラ権限・マイク権限
- カメラ映像
- マイク音声
- ライト
- 前面・背面カメラ切替
- 60fps選択時の実効FPS
- 長時間待機
- Foreground Serviceの通知・継続
- 画面OFF時の待機
- バッテリー消費・発熱
- 遠隔操作の応答
- モバイル通信での接続

### iPad Safari / PWA
- ホーム画面追加・standalone起動
- 8桁ペアリング
- WebRTC接続
- 映像・音声受信
- PTT送話
- ライト操作
- 全画面表示
- 全画面終了後の映像復帰
- 検知範囲の作成・編集・削除
- ズーム時の範囲位置合わせ
- 範囲内だけでの検知
- 人物候補の誤検知
- 進入・退出イベント
- 通知許可・拒否時の動作
- 履歴・画像保存・CSV出力
- PWAのバックグラウンド休止・復帰

### 複数端末
- 2台・3台同時閲覧
- 1台の閲覧端末を切断しても残りの映像が継続すること
- 4台目が拒否されること
- 各閲覧端末の登録解除
- 端末名・最終応答時刻の更新

---

# 13. 既知の制限・注意事項

1. TURNサーバーがないため、ネットワーク環境によってWebRTC接続が失敗する。
2. iPadOSのバックグラウンド制限により、PWAの映像解析が停止する場合がある。
3. Androidの強制停止後に遠隔起動できる保証はない。
4. 60fpsは端末・カメラ・エンコーダーの対応状況に依存する。
5. 音声フィルターはAIによるノイズ除去ではない。
6. 動体検知は照明変化や影でも反応する可能性がある。
7. 人物候補検知は本人特定ではない。
8. OS通知はブラウザー・OSの対応条件に依存する。
9. PWAが停止中の即時通知は保証しない。
10. 通信量表示は概算である。
11. ブックマークレットでは通常のWebページと同一の権限・API利用ができない場合がある。
12. AndroidのForeground ServiceはOS・メーカーの省電力設定等に左右される。
13. ローカル検知履歴はブラウザーのデータ消去・容量制限の影響を受ける。
14. モバイル通信・異なるWi-Fi間の接続成功はTURN未構成のため保証しない。

---

# 14. セキュリティ・プライバシー方針

- カメラ・マイク利用はAndroidの権限・Foreground Service通知に従う。
- ユーザーが監視待機・遠隔撮影許可を確認できるUIを維持する。
- Androidの強制停止やOSのプライバシー制御を迂回しない。
- スプレッドシートを一般公開しない。
- controllerTokenやdeviceSecretをWebソース・ログへ平文で出さない。
- GASではトークンをハッシュで照合する。
- 不要な正確な位置情報を監査ログに書き込まない。
- 検知画像・検知範囲・検知イベントはブラウザー内に保持し、GASへ送信しない。
- 位置情報履歴とセキュリティ監査ログを分離する。
- 遠隔操作要求には有効なペアリング・コントローラ認証を要求する。
- PTT送話ロックには3分の安全自動解除を設ける。

---

# 15. 今後の開発ルール

## 最重要ルール

- 既存機能を勝手に削除しない。
- 既存バージョンを勝手に上書きしない。
- mainへ勝手にマージしない。
- 既存GASのDB構造を勝手に変更しない。
- Firebaseを導入しない。
- 外部AI APIを勝手に導入しない。
- 人物認識と人物候補検知を混同しない。
- AndroidとWebのAPIバージョンを一致させる。
- Web側の変更だけでAndroid側の動作が壊れないか確認する。
- GASの変更が既存データに影響しないか確認する。
- APKビルド成功と実機試験成功を混同しない。
- 仕様が不明な箇所を推測で実装しない。
- v4.3 / v4.4の未確認仕様を作らない。
- v4.5.1 PR #12はDraftのまま維持する。

## 開発手順

1. 既存コード・仕様書・ブランチを確認。
2. 変更対象を明確にする。
3. 既存機能への影響を調査。
4. 新しい開発ブランチを作る。
5. 実装する。
6. Web・GAS・Androidのコードを検査する。
7. GitHub Actionsでビルドする。
8. エラーを修正する。
9. 実機テスト項目を整理する。
10. Draft PRとして保持する。
11. 実機確認が完了するまではmainへ統合しない。

## 推奨するコード点検

- JavaScript：構文、DOM参照、イベント多重登録、非同期競合、タイマー解放、Object URL解放、WebRTC PeerConnection単位の後始末
- GAS：APIアクション互換性、シート列互換性、認証、期限切れデータ、ロック、再試行、エラー応答、秘密情報のログ出力
- Android：権限、Foreground Service起動条件、通知、カメラ・マイクの解放、PeerConnection単位の管理、画面OFF・復帰、HTTPタイムアウト
- PWA：キャッシュ更新、同一オリジン、Service Worker更新、iPad Safari互換性、バックグラウンド停止時の説明
- UI：左操作・右表示、全画面復帰、狭い画面、誤操作防止、読み上げ・フォーカス

---

# 16. 現時点で確認したソース・未確認事項

## 16-1. v4.5.1で取得・確認した主なファイル

| ファイル | 確認した内容 |
|---|---|
| `web/config.json` | GAS URL・APIバージョン・ポーリング間隔・タイムアウト |
| `web/app.js` | WebRTC制御、再接続、全画面、画像ビューアー、ライト、PTT、動体検知、人物候補、履歴、複数閲覧端末関連の処理 |
| `web/index.html` | 左側の設定、PTT安全解除、動体検知範囲UI、人物候補UI、通知・履歴・CSV、映像オーバーレイ |
| `web/sw.js` | PWAアプリシェルキャッシュ、通知クリック処理。Pushイベントは使用しない |
| `web/manifest.webmanifest` | 日本語PWA名、standalone表示、アイコン、白基調 |
| `gas/Code.gs` | API v4.5.1、DBテーブル、認証、ペアリング、Viewer上限、WebRTCシグナリング、位置情報 |
| `android/app/build.gradle.kts` | applicationId、versionCode、versionName、SDK、依存関係 |
| `android/app/src/main/AndroidManifest.xml` | CAMERA / RECORD_AUDIO / Foreground Service / 位置情報等の権限 |
| `android/.../Config.kt` | Remote Configの参照先 |
| `android/.../MainActivity.kt` | 待機モード、遠隔カメラ・マイク許可、画面OFF時のCPU維持、音声モードUI |
| `android/.../MonitorService.kt` | Foreground Service、通知、監視待機、GASポーリング |
| `android/.../RtcEngine.kt` | WebRTC、最大3 PeerConnection、映像・音声トラック、DataChannel、カメラ切替、ライト |
| `android/.../ApiClient.kt` | GAS HTTP通信、タイムアウト、再試行、リダイレクト先の検査 |
| `docs/REMOTE-CAMERA-v4.5.1-SPEC.md` | v4.5.1検知仕様 |
| `docs/QUICKSTART-v4.5.1.md` | GAS / Android / Webセットアップ |
| `docs/BUILD-REPORT-v4.5.1.md` | ビルド記録 |
| `docs/TEST-PLAN-v4.5.1.md` | 実機試験計画 |

## 16-2. まだ実機で確認できていないこと

- Galaxy A22 5G / Android 13での長時間の安定性
- iPad Safariにおける全画面復帰
- 実際のネットワークを変えたWebRTC接続
- 60fpsの実効性能
- PTTの双方向音声品質
- 監視範囲とズーム位置の一致精度
- 人物候補検知の誤検知率
- 長期保存時のローカル履歴容量
- iPadOSバックグラウンド復帰後の再開
- 3台同時監視時の端末負荷・発熱

## 16-3. ソース点検の範囲

この統合仕様書の作成時には、PR #12のメタデータ・変更ファイル一覧、Web主要ファイル、GAS Code.gs、Androidの設定・Manifest・主要Kotlinファイル、v4.5.1の仕様書・クイックスタート・ビルドレポート・テスト計画を確認した。

ただし、全ソースの全行について形式検証・実機デバッグ・ネットワーク負荷試験を完了したという意味ではない。新しい開発を始める前に、作業対象ブランチを改めて取得し、全体を点検すること。

---

# 17. 参考リンク

## 実装リポジトリ
- [GitHub repository: a09848281-prog/web](https://github.com/a09848281-prog/web)

## v4.5.1
- [Draft PR #12](https://github.com/a09848281-prog/web/pull/12)
- [v4.5.1 Webプレビュー](https://a09848281-prog.github.io/web/v4.5.1-preview/)
- [v4.5.1 GAS Code.gs](https://github.com/a09848281-prog/web/blob/rebuild-v4.5.1-smart-motion-pwa/gas/Code.gs)
- [v4.5.1 Android説明](https://github.com/a09848281-prog/web/blob/rebuild-v4.5.1-smart-motion-pwa/android/README.md)
- [v4.5.1 セットアップ](https://github.com/a09848281-prog/web/blob/rebuild-v4.5.1-smart-motion-pwa/docs/QUICKSTART-v4.5.1.md)
- [v4.5.1 仕様書](https://github.com/a09848281-prog/web/blob/rebuild-v4.5.1-smart-motion-pwa/docs/REMOTE-CAMERA-v4.5.1-SPEC.md)
- [v4.5.1 ビルドレポート](https://github.com/a09848281-prog/web/blob/rebuild-v4.5.1-smart-motion-pwa/docs/BUILD-REPORT-v4.5.1.md)
- [v4.5.1 テスト計画](https://github.com/a09848281-prog/web/blob/rebuild-v4.5.1-smart-motion-pwa/docs/TEST-PLAN-v4.5.1.md)

## 仕様書・デモ管理
- [デモリポジトリ a09848281-prog/a](https://github.com/a09848281-prog/a)
- [本統合仕様書](https://github.com/a09848281-prog/a/blob/main/docs/REMOTE-CAMERA-HANDOVER-v3.8-v4.5.1.md)

---

# 18. 引き継ぎ先AIへの指示

あなたは、このRemote Cameraプロジェクトの継続開発を担当するAIである。

最初に以下を行うこと。

1. 実装リポジトリ `a09848281-prog/web` を確認する。
2. v4.5.1のPR #12がDraft・未マージであることを確認する。
3. v4.2.2を基準とした変更内容を把握する。
4. Web・GAS・Androidの対象ブランチのコードを取得する。
5. 本書と実際のコードの差分を整理する。
6. 実装済み・未実装・未検証を区別する。
7. Actionsの実行番号・Artifact IDの不一致を確認してからAPKを案内する。
8. 既存機能に影響する変更は、新しいブランチで実装・検証する。

ユーザーが新機能を希望した場合は、既存機能との干渉を避けながら設計・実装する。最優先は安定性・既存機能の維持・安全性である。不明な仕様を勝手に推測して実装しない。

**このリポジトリ `a09848281-prog/a` は仕様書・デモ管理用であり、実際の動作コードの正本は `a09848281-prog/web` である。** 本書を保存しただけでカメラアプリやAPKが新規作成されたと判断してはならない。
