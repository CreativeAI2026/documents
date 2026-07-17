# 物語班リファレンス(立ち絵・キーカタログ・events.json フォーマット)

物語班が手書きする `events.json` の **フォーマット** と、そこに書くキーの **カタログ**(`portrait` の立ち絵キー、`itemKey`/`weaponKey` のアイテム・武器キー)を定める。物語班はこの1ファイルだけ読めばよい。

---

## 登場人物と立ち絵

各キャラに用意する表情と、`line` ステップの `portrait` に書くキーの一覧。
キーは `{キャラ}_{表情}` の snake_case。

### 主人公 — `hero`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔 | `hero_normal` |
| 2 | 笑顔(口角が上がる程度) | `hero_smile` |
| 3 | 困惑 | `hero_confused` |
| 4 | 驚き | `hero_surprised` |
| 5 | 疑問 | `hero_question` |
| 6 | 怒り・覚悟(眉尻が上がる) | `hero_anger` |

### はかなげ少女 — `girl`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔(立ち絵の顔) | `girl_normal` |
| 2 | 怯え(涙が浮かぶ程度) | `girl_fear` |
| 3 | 覚悟 | `girl_resolve` |
| 4 | 笑顔 | `girl_smile` |
| 5 | 困り笑い | `girl_wry_smile` |
| 6 | 驚き | `girl_surprised` |

### ロボット — `robot`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔 | `robot_normal` |

### 異形(蓄音機) — `gramophone`

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

### 装備品 — `giveItem` の `itemKey`

| 装備品 | key |
| ------ | --- |
| 蛍光灯 | `fluorescent_lamp` |
| 磁石 | `magnet` |
| 辞書 | `dictionary` |
| キーボード | `keyboard` |
| メガネ | `glasses` |
| 体温計 | `thermometer` |
| 腕時計 | `wristwatch` |
| 全身タイツ | `bodysuit` |
| 傘 | `umbrella` |
| 帽子 | `hat` |

### 食材 — `giveItem` の `itemKey`

| 食材 | key |
| ---- | --- |
| りんご | `apple` |
| ぶどう | `grapes` |
| バナナ | `banana` |
| クッキー | `cookie` |
| コーヒー | `coffee` |
| みかんジュース | `orange_juice` |
| クリーム玄米 | `cream_brown_rice` |
| お米 | `rice` |
| ゼリー | `jelly` |
| 味噌汁 | `miso_soup` |

### 大事なもの — `giveItem` の `itemKey`

**現状は大事なものを1つも用意していない**。時間があれば...

---

## events.json フォーマット

```jsonc
{
  "events": [
    {
      "id": "cave_encounter",      // シーン上のトリガーとこの id を対応させる
      "conditions": [              // すべて満たすと発火(AND)
        { "type": "progress", "value": 5 }             // 進行度がちょうど 5 のとき発火(== 判定)
      ],
      "steps": [
        { "kind": "line",       "speaker": "主人公",       "portrait": "hero_surprised", "text": "…誰だ?" },
        { "kind": "line",       "speaker": "はかなげ少女", "portrait": "girl_fear",      "text": "来ないで……っ" },
        { "kind": "battle" },                                    // 戦闘(敵はシーンのトリガーに配線。JSON に敵は書かない)
        { "kind": "line",       "speaker": "はかなげ少女", "portrait": "girl_resolve",   "text": "……助けてくれたのね。これで、身を守って。" },
        { "kind": "giveWeapon", "weaponKey": "scythe" }, // 武器を渡す → WeaponManager。これが最初の1本なら武器切替UIが出る(キーは上の武器カタログ)
        { "kind": "choice", "flag": "girl_choice", "options": [   // 選んだ値を flags["girl_choice"] に書く
            { "text": "一緒に行く",   "value": "together" },
            { "text": "ひとりで行く", "value": "alone" }
        ]},
        { "kind": "line",       "speaker": "主人公", "portrait": "hero_normal", "text": "…そうか。" }
      ],
      "nextProgress": 6            // 終了時に進行度を 6 へ。5 でなくなるので入り直しても二度と発火しない=1回きり
    },
    {
      "id": "robot_supply",
      "conditions": [
        { "type": "progress", "value": 7 }
      ],
      "steps": [
        { "kind": "line",     "speaker": "ロボット", "portrait": "robot_normal", "text": "補給物資ヲ渡ス。受ケ取レ。" },
        { "kind": "giveItem", "itemKey": "apple" },      // itemKey で ItemData を引いて1個渡す。カテゴリ(装備品/食材/大事なもの)は ItemData 側が持つ(キーは上のカタログ)
        { "kind": "giveItem", "itemKey": "umbrella" },   // giveItem は1個1ステップ。複数渡すなら並べる(battle と違い個数制限なし)
        { "kind": "line",     "speaker": "主人公", "portrait": "hero_smile", "text": "助かるよ。" }
      ],
      "nextProgress": 8
    },
    {
      "id": "girl_reunion",
      "conditions": [
        { "type": "progress", "value": 8 },
        { "type": "flag", "key": "girl_choice", "value": "together" }   // 「一緒に行く」を選んだ人だけ発火
      ],
      "steps": [
        { "kind": "line", "speaker": "はかなげ少女", "portrait": "girl_smile", "text": "ここまで一緒に来られたね。" }
      ],
      "nextProgress": 9            // 必須。進行度を 9 へ進め、8 でなくなるので二度と発火しない
    }
  ]
}
```

### フィールド

| フィールド | 必須 | 内容 |
| ---------- | ---- | ---- |
| `id` | ○ | イベント識別子。シーン上のトリガーとこの id を対応させる(座標は JSON に書かない) |
| `conditions[]` | ○ | 発火条件。すべて満たす(AND)と発火。**`progress` を必ず1つ含む**(進行度が `value` に**一致**したとき真)。`flag` は同じ進行度での分岐に任意で足す |
| `steps[]` | ○ | 会話ステップの並び。`line` / `battle` / `giveItem` / `giveWeapon` / `choice` |
| `nextProgress` | ○ | 終了時に進める進行度。**全イベント必須**で、`progress` の `value` **より大きく**する。イベントは進行度が `value` に一致したときだけ発火し、終了で進行度が進んで一致しなくなるので、**どのイベントもちょうど1回だけ発火する** |

> **`battle` ステップの制約**: `steps[]` の並び順は自由だが、**`battle` は必ず会話の途中に置く**(先頭・末尾は `line`。戦闘が単独・末尾にならない)。**1イベントにつき `battle` は最大1つ**。

---