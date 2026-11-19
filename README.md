# Remote Camera — Demo / Handover

Android・iPad・PC対応の遠隔監視カメラシステムについて、開発引き継ぎ資料とデモ計画を管理するリポジトリです。

## 現在の位置づけ

- このリポジトリは**仕様書・引き継ぎ資料用のデモリポジトリ**です。
- 実際のv3.8〜v4.5.1実装の正本は、別リポジトリ `a09848281-prog/web` にあります。
- 既存実装をこのリポジトリへ自動コピーしたり、元リポジトリを変更したりしません。
- v4.5.1はDraft PR #12として保持され、mainへマージしません。

## 仕様書

- [開発引き継ぎ・統合仕様書 v3.8〜v4.5.1](docs/REMOTE-CAMERA-HANDOVER-v3.8-v4.5.1.md)

## 元リポジトリ・参照先

- [実装リポジトリ](https://github.com/a09848281-prog/web)
- [v4.5.1 Draft PR #12](https://github.com/a09848281-prog/web/pull/12)
- [v4.5.1 Webプレビュー](https://a09848281-prog.github.io/web/v4.5.1-preview/)
- [v4.5.1 GAS Code.gs](https://github.com/a09848281-prog/web/blob/rebuild-v4.5.1-smart-motion-pwa/gas/Code.gs)
- [v4.5.1 クイックスタート](https://github.com/a09848281-prog/web/blob/rebuild-v4.5.1-smart-motion-pwa/docs/QUICKSTART-v4.5.1.md)

## 開発上の注意

- main、v4.2、v4.2.1、v4.2.2、v4.5.1の既存ブランチ・PRを勝手に変更しない。
- Firebase、外部AI人物認識API、追加の有料サービスを勝手に導入しない。
- 機械学習による人物認識と、画像処理による人物候補検知を混同しない。
- GitHub Actionsの成功を実機動作確認済みと表現しない。
- カメラ・マイクはユーザーが明示的に許可・開始した状態で利用し、OSの制約を回避しない。
