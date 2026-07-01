# 登場人物・立ち絵と events.json フォーマット

物語班が手書きする `events.json` の **正本フォーマット** と、`portrait`(立ち絵)に書くキーの **カタログ** を定める。
システム全体での位置づけ・進行度・条件・会話ステップの考え方は `StoryProgressionSystem.md` を参照。

---

## 登場人物と立ち絵

各キャラに用意する表情と、`line` ステップの `portrait` に書くキーの一覧。
キーは `{キャラ}_{表情}` の snake_case。実体(PNG)は視覚班が用意し、システム班がキーとして登録する(`StoryProgressionSystem.md` のカタログ参照)。

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

### ロボット(未確定) — `robot`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔 | `robot_normal` |

### 異形(蓄音機) — `gramophone`

| # | 表情 | portrait キー |
| - | ---- | ------------- |
| 1 | 通常顔 | `gramophone_normal` |

> ロボットはキャラ自体が未確定。表情が追加され次第ここに追記する。
> `speaker`(表示名)は表中のキャラ名に準拠。正式名称が決まったら差し替える。

---

## events.json フォーマット

```jsonc
{
  "events": [
    {
      "id": "cave_encounter",      // シーン上のトリガーとこの id を対応させる
      "conditions": [              // すべて満たすと発火(AND)
        { "type": "progress", "value": 5 },           // 進行度 >= 5
        { "type": "hasItem",  "itemKey": "old_key" }
      ],
      "startBgm": "bgm_tense",     // 任意(省略可)
      "steps": [
        { "kind": "line",     "speaker": "主人公",       "portrait": "hero_surprised", "text": "…誰だ?" },
        { "kind": "line",     "speaker": "はかなげ少女", "portrait": "girl_fear",      "text": "来ないで……っ" },
        { "kind": "battle",   "enemyKey": "wolf_boss" },
        { "kind": "line",     "speaker": "はかなげ少女", "portrait": "girl_resolve",   "text": "……ありがとう。これを持っていって。" },
        { "kind": "giveItem", "itemKey": "old_key" },
        { "kind": "choice", "flag": "girl_choice", "options": [   // 選んだ値を flags["girl_choice"] に書く
            { "text": "一緒に行く",   "value": "together" },
            { "text": "ひとりで行く", "value": "alone" }
        ]},
        { "kind": "line",     "speaker": "主人公", "portrait": "hero_normal", "text": "…そうか。" }
      ],
      "nextProgress": 6            // 終了時に進める進行度(任意。省略時は進めない)
    },
    {
      "id": "girl_reunion",
      "conditions": [
        { "type": "progress", "value": 8 },
        { "type": "flag", "key": "girl_choice", "value": "together" }   // 「一緒に行く」を選んだ人だけ発火
      ],
      "steps": [
        { "kind": "line", "speaker": "はかなげ少女", "portrait": "girl_smile", "text": "ここまで一緒に来られたね。" }
      ]
    }
  ]
}
```

### フィールド

| フィールド | 必須 | 内容 |
| ---------- | ---- | ---- |
| `id` | ○ | イベント識別子。シーン上のトリガーとこの id を対応させる(座標は JSON に書かない) |
| `conditions[]` | ○ | 発火条件。すべて満たす(AND)と発火。`progress` / `hasItem` / `flag` のみ |
| `startBgm` | | イベント開始時の BGM キー。省略可 |
| `steps[]` | ○ | 会話ステップの並び。`line` / `battle` / `giveItem` / `choice` |
| `nextProgress` | | 終了時に進める進行度。省略時は進めない |

ステップ・条件の各タイプの意味は `StoryProgressionSystem.md` の表を参照。

---

## Importer の検証(違反は取り込みエラー)

- `steps` の先頭・末尾は必ず `line`(`battle` / `giveItem` / `choice` は単独・末尾にならない)
- `portrait` / `enemyKey` / `itemKey` / `startBgm` はカタログに存在するキーのみ
- `conditions[].type` は `progress` / `hasItem` / `flag` のみ

> 進行度の比較は `>=`(以上)。
