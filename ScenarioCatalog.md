# 物語班カタログ(立ち絵キー・アイテム/武器キー)

物語班が `events.json` に書く **キーの一覧**(`portrait` の立ち絵キー、`itemKey` / `weaponKey` のアイテム・武器キー)。書式そのものは [`ScenarioReference.md`](./ScenarioReference.md) を見る。

アセットやアイテムが増えたら、このページに足す。

---

## 登場人物と立ち絵

各キャラに用意する表情と、`line` ステップの `portrait` に書くキーの一覧。
キーは `{キャラ}_{表情}` の snake_case。

> 実行時はこのキー**だけ**で立ち絵・立ち位置(左/右)・名前色・タイプ音・表示名が決まる。
> **表の見出しの名前 = 画面に出る表示名**(`speaker` を書けばそちらが優先される)。

### 主人公 — `hero`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔 | `hero_normal` |
| 2 | 笑顔(口角が上がる程度) | `hero_smile` |
| 3 | 困惑 | `hero_confused` |
| 4 | 驚き | `hero_surprised` |
| 5 | 疑問 | `hero_question` |
| 6 | 怒り・覚悟(眉尻が上がる) | `hero_anger` |

### 儚げな少女 — `girl`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔(立ち絵の顔) | `girl_normal` |
| 2 | 怯え(涙が浮かぶ程度) | `girl_fear` |
| 3 | 覚悟 | `girl_resolve` |
| 4 | 笑顔 | `girl_smile` |
| 5 | 困り笑い | `girl_wry_smile` |
| 6 | 驚き | `girl_surprised` |

### ロボ — `robot`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔 | `robot_normal` |
| 2 | 思考 | `robot_thinking` |

### 蓄音機の紳士 — `gramophone`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔 | `gramophone_normal` |

---

## アイテム・武器キーのカタログ

`giveItem` の `itemKey` / `giveWeapon` の `weaponKey` に書く **文字列キーの一覧**。立ち絵の `portrait` キーと同じく、物語班はこの表のキーだけを書く。キーは英字 snake_case。

### 武器 — `giveWeapon` の `weaponKey`

| 武器 | key |
| ---- | --- |
| 剣 | `sword` |
| 弓 | `bow` |
| 鎌 | `scythe` |
| ハンマー | `hammer` |

### 装備品 — `giveItem` の `itemKey`

| 装備品 | key |
| ------ | --- |
| 蛍光灯 | `fluorescent_light` |
| 磁石 | `magnet` |
| ノート | `note` |
| マイク | `microphone` |
| メガネ | `glasses` |
| 風船 | `balloon` |
| 懐中時計 | `clock` |
| 全身タイツ | `full_body_tights` |
| 傘 | `umbrella` |
| 帽子 | `hat` |

### 食材 — `giveItem` の `itemKey`

| 食材 | key |
| ---- | --- |
| りんご | `apple` |
| ぶどう | `grapes` |
| 桃 | `peach` |
| バナナ | `banana` |
| クリーム玄米クッキー | `brown_rice_cream_cookie` |
| コーヒー | `coffee` |
| みかんジュース | `orange_juice` |
| 白米 | `white_rice` |
| いちごゼリー | `strawberry_jelly` |
| 味噌汁 | `miso_soup` |

### 大事なもの — `giveItem` の `itemKey`

| 大事なもの | key |
| ---------- | --- |
| カードキー | `card_key` |
| 形見のアクセサリー | `keepsake_accessory` |
| 汚れたレコード | `dirty_record` |
| 壊れたロボット | `broken_robot` |
| 機密書類１ | `classified_document_1` |
| 機密書類２ | `classified_document_2` |
| 機密書類３ | `classified_document_3` |
| 謎の手記１ | `mysterious_note_1` |
| 謎の手記２ | `mysterious_note_2` |
| 謎の手記３ | `mysterious_note_3` |
| 謎の鍵 | `mysterious_key` |

