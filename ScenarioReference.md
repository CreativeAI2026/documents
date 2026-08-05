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
    },
    {
      "id": "locked_door",
      "conditions": [
        { "type": "progress", "value": 10 },
        { "type": "hasItem", "itemKey": "mysterious_key" }   // 「謎の鍵」(大事なもの)を所持していれば発火
      ],
      "steps": [
        { "kind": "line", "speaker": "主人公", "portrait": "hero_normal", "text": "この鍵で開きそうだ。" }
      ],
      "nextProgress": 11
    }
  ]
}
```

### フィールド

| フィールド | 必須 | 内容 |
| ---------- | ---- | ---- |
| `id` | ○ | イベント識別子。**物語班が命名する**(イベント内容がわかる英字 snake_case。例: `cave_encounter`)。シーン上のトリガーとこの id を対応させる(座標は JSON に書かない)。**全イベントで一意**にする |
| `conditions[]` | ○ | 発火条件。すべて満たす(AND)と発火。**`progress` を必ず1つ含む**(進行度が `value` に**一致**したとき真)。`flag` / `hasItem` は同じ進行度での分岐に任意で足す(下の「条件タイプ」参照) |
| `steps[]` | ○ | 会話ステップの並び。`line` / `battle` / `giveItem` / `giveWeapon` / `choice` |
| `nextProgress` | ○ | 終了時に進める進行度。**全イベント必須**で、`progress` の `value` **より大きく**する。イベントは進行度が `value` に一致したときだけ発火し、終了で進行度が進んで一致しなくなるので、**どのイベントもちょうど1回だけ発火する** |

### 条件タイプ (`conditions[]` の `type`)

| type | フィールド | 真になる条件 |
| ---- | ---------- | ------------ |
| `progress` | `value`(整数) | 進行度が `value` に**ちょうど一致**(必須・各イベント1つ以上) |
| `flag` | `key` / `value`(文字列) | フラグ `key` の値が `value` に一致(`choice` で書き込んだ値の分岐に使う) |
| `hasItem` | `itemKey`(文字列) | その **`itemKey` の「大事なもの」を1つ以上所持**していれば真 |

> **`hasItem` の制約**: `itemKey` は **大事なものカタログ**。**「どれか1つ持っていれば」= 特定の1 key を指定**する。複数 `hasItem` を並べると AND(=全部所持)になる。「AのkeyかBのkeyのどちらか」のような OR は無い。

> **`battle` ステップの制約**: `steps[]` の並び順は自由だが、**`battle` は必ず会話の途中に置く**(先頭・末尾は `line`。戦闘が単独・末尾にならない)。**1イベントにつき `battle` は最大1つ**。

---