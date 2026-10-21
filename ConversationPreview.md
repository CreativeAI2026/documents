# 会話プレビュー手順

`events.json` に書いた会話を、フィールドを通さずに `UI_ConversationPreview` シーンで再生して確認する手順。

## 手順

1. `game/Assets/_Project/Features/Event/events.json` に置く
2. Unity で `Tools > CreativeAI > Import Events` → `events.json` を選択(エラーは Console に出る)
3. `Features/Event/Data/` にイベントごとの `.asset` ができたことを確認
4. `Scenes/UI/UI_ConversationPreview.unity` を開き、`ConversationPreviewDriver` が付いた GameObject を選択
5. Inspector の **Event** 欄に見たいイベントの `.asset` をドラッグ
6. Play

Event 欄を空にすると組み込みデモが流れる。

## このシーンでの制限

見た目(台詞・立ち絵・選択肢・演出コマンド)の確認用。次のものは本番どおりには動かない。

- `giveItem` / `giveWeapon`: 入手演出だけ出て、所持品には入らない
- 選択肢のフラグ: プレビュー内の一時状態に記録し、後続branchで評価する。本番フラグ・終了時進行度は変更しない
- `battle`: 敵がいないので警告を出してスキップ

## 分岐の確認

ConversationPreviewDriverのInspectorでInitial Flagsにフラグ名と値、Important Item Keysに大事なもののitemKeyを設定してPlayする。選択肢の結果は同じ再生内でbranchに反映される。giveItemも大事なものの分岐判定用の一時所持状態だけに反映し、本番インベントリは変更しない。ループ再生時は初期状態へ戻る。分岐した経路の終了後は共通ステップへ合流する。

## ブラウザのシナリオエディタ

https://creativeai2026.github.io/game/scenario-editor/ から会話カード・演出・分岐を編集し、画像込みのプロジェクトバックアップとUnity用JSONを出力できる。右側のテスト状態でフラグ・所持品を設定して各経路を再生できる。戦闘と武器3D獲得表示は模擬表示、camera.* / background.*はUnity側の受け手が未実装と表示する。

利用手順: https://github.com/CreativeAI2026/game/blob/feature/scenario-editor-branches/tools/scenario-editor/README.md
