# アイテム画像・素材の差し替え手順

全アイテムのデータは `Resources/ItemCatalog.asset` の1ファイルにまとまっていて、画像と `items.json` から同期で自動生成する。レシピは合成後アイテムの素材から決まる。

## 画像のルール

パスは `game/Assets/_Project/` から。画像は正方形・背景透過の PNG。

| 種類 | 合成前の素材 | 合成後 |
| ---- | ------------ | ------ |
| 装備品 | `Art/UI/Items/Equipment/PreCraft/item_equipment_<素材>.png` | `Art/UI/Items/Equipment/PostCraft/item_equipment_<素材1>_<素材2>.png` |
| 食材 | `Art/UI/Items/Food/PreCraft/item_food_<素材>.png` | `Art/UI/Items/Food/PostCraft/item_food_<素材1>_<素材2>.png` |
| 大事なもの | `Art/UI/Items/Important/item_importance_<名前>.png` | — |

- `<素材>` は英小文字と数字だけ。単語の区切りはハイフン(例: `fluorescent-light`)
- ファイル名がそのままアイテムの key になる(ハイフンはアンダースコアに変わる。例: `fluorescent-light` → `fluorescent_light`)
- 合成後の `<素材1>_<素材2>` はアルファベット順に並べる(例: `clock_lantern`)
- 画像を消すときは **Unity の Project ウィンドウで消す**

## 名前・説明のルール

`game/Assets/_Project/Features/Inventory/items.json` に、key ごとに名前と説明を書く。合成前・合成後・大事なもの、すべてのアイテムが対象。

```json
{
  "balloon": { "name": "風船", "description": "" },
  "balloon_clock": { "name": "風船と懐中時計", "description": "" }
}
```

- key は画像のファイル名から決まるもの(上の「画像のルール」)と一致させる
- 画像があるのに `items.json` に無い key、`items.json` にあるのに画像が無い key は同期でエラーになる

## 名前・説明だけ直す場合

`items.json` を書き換えて、同期する。

## 絵だけ描き直す場合

同じファイル名で PNG を上書きするだけ。同期は要らない。

## 素材を別のアイテムに変える場合

例: 装備品の風船(`balloon`)をランタン(`lantern`)に変える。素材1つにつき、その素材を使う合成後アイテム9個も入れ替わる。

1. 古い素材の画像1枚と、合成後の画像9枚を消す
   - `game/Assets/_Project/Art/UI/Items/Equipment/PreCraft/item_equipment_balloon.png`
   - `game/Assets/_Project/Art/UI/Items/Equipment/PostCraft/item_equipment_balloon_*.png`(`balloon` を含む9枚)
2. 新しい素材の画像1枚と、合成後の画像9枚を置く
   - `game/Assets/_Project/Art/UI/Items/Equipment/PreCraft/item_equipment_lantern.png`
   - `game/Assets/_Project/Art/UI/Items/Equipment/PostCraft/item_equipment_clock_lantern.png` など9枚
3. `game/Assets/_Project/Features/Inventory/items.json` の該当 key を書き換える
   - 古い key(`balloon`、`balloon_clock` など10件)を消し、新しい key(`lantern`、`clock_lantern` など10件)の名前と説明を書く
4. Unity で `Tools > CreativeAI > アイテムを同期` を実行する
   - 画像に合わせて `ItemCatalog` の中身が作り直される(新しい素材・合成後のアイテムができ、古いものは消える)
5. key の参照を新しい key に直す
   - documents リポジトリの [`ScenarioReference.md`](./ScenarioReference.md) の「アイテム・武器キーのカタログ」
   - `game/Assets/_Project/Features/Event/events.json` の `giveItem` の `itemKey`