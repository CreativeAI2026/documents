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
- 選択肢のフラグ・終了時の進行度更新: 記録されない
- `battle`: 敵がいないので警告を出してスキップ
