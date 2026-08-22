# 物語班リファレンス(立ち絵・キーカタログ・events.json フォーマット)

物語班が手書きする `events.json` の **フォーマット** と、そこに書くキーの **カタログ**(`portrait` の立ち絵キー、`itemKey`/`weaponKey` のアイテム・武器キー)を定める。物語班はこの1ファイルだけ読めばよい。

---

## 登場人物と立ち絵

各キャラに用意する表情と、`line` ステップの `portrait` に書くキーの一覧。
キーは `{キャラ}_{表情}` の snake_case。

> **キーは立ち絵定義アセットと同じ文字列**(`Features/UI/ConversationUI/Data/Characters/*.asset` の `PortraitKey`)。
> 実行時はこのキー**だけ**で立ち絵・立ち位置(左/右)・名前色・タイプ音・表示名が決まる。
> **表の見出しの名前 = 画面に出る表示名**(`speaker` を書けばそちらが優先される)。
> 絵がまだ用意できていない表情もこの表に載せてよい。取り込みは通り、Importer が
> 「立ち絵アセット未登録」と**警告**を出し、実行時は既定の立ち絵で表示される。

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

> ⚠️ **この表のアイテムはまだ ItemData アセットが作られていない**。作られるまで `giveItem` にこのキーを書くと
> 取り込みがエラーで止まり、`hasItem` に書くと(警告どまりで通るが)実行時に必ず false になりイベントが発火しない。
> 逆に、実在する「携帯地図」はまだ `key` が未設定で JSON から指せない。

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
      "id": "station_awakening",   // シーン上のトリガーとこの id を対応させる
      "conditions": [              // すべて満たすと発火(AND)
        { "type": "progress", "value": 5 }             // 進行度がちょうど 5 のとき発火(== 判定)
      ],
      "steps": [
        // portrait を書かない line = 地の文(ナレーション)。立ち絵も名前欄も出ない
        { "kind": "line", "text": "雨音の向こうで、古いレコードが途切れ途切れに鳴っている。" },

        // speaker を書かなければ、立ち絵定義の表示名(主人公 / 儚げな少女 / ロボ / 蓄音機の紳士)がそのまま出る
        { "kind": "line", "portrait": "hero_surprised", "text": "……知らない天井だ。ここは駅舎、なのか？" },
        { "kind": "line", "portrait": "robot_normal",   "text": "覚醒ヲ確認。外傷ナシ。" },

        { "kind": "line", "text": "待合室の奥、壊れた照明の下で人影が動いた。" },

        // 正体を伏せる: 右の立ち絵を暗く沈め、名前も ？？？ に差し替える
        { "kind": "command", "command": "portrait.right.obscure" },
        { "kind": "line", "speaker": "？？？", "portrait": "girl_fear",
          "text": "<whisper>あの……倒れていたあなたを運んだのは、その子です。</whisper>" },

        { "kind": "command", "command": "portrait.right.reveal" },   // 明かす
        { "kind": "line", "portrait": "girl_wry_smile", "text": "ご、ごめんなさい。明るいところが少し苦手で……。" },

        { "kind": "battle" },                                    // 戦闘(敵はシーンのトリガーに配線。JSON に敵は書かない)

        { "kind": "command", "command": "portrait.left.jump" },  // 主人公の立ち絵を跳ねさせる
        { "kind": "line", "portrait": "hero_surprised",
          "text": "<wait=0.25><shake><shout>蓄音機までしゃべるのか……。</shout></shake>でも、敵意はなさそうだ。" },

        { "kind": "line", "portrait": "girl_resolve", "text": "……助けてくれたのね。これで、身を守って。" },
        { "kind": "giveWeapon", "weaponKey": "scythe", "message": "錆びた鎌を手に入れた。" }, // 武器を渡す → WeaponManager。最初の1本なら武器切替UIが出る。渡すと入手演出(3Dモデル+文)が出る

        { "kind": "choice", "flag": "girl_choice", "options": [   // 選んだ値を flags["girl_choice"] に書く
            { "text": "一緒に行く",   "value": "together" },
            { "text": "ひとりで行く", "value": "alone" }
        ]},
        { "kind": "line", "portrait": "hero_normal", "text": "…そうか。" }
      ],
      "nextProgress": 6            // 終了時に進行度を 6 へ。5 でなくなるので入り直しても二度と発火しない=1回きり
    },
    {
      "id": "robot_supply",
      "conditions": [
        { "type": "progress", "value": 7 }
      ],
      "steps": [
        // 場面転換: ウィンドウだけ引っ込めて間を置き、戻して地の文から入り直す
        { "kind": "command", "command": "window.hide" },
        { "kind": "command", "command": "wait", "arg": "0.45" },   // wait は arg(秒)が必須
        { "kind": "command", "command": "window.show" },
        { "kind": "line", "text": "少女と蓄音機が、ホーム脇の倉庫から旅支度を運んできた。" },

        { "kind": "line", "portrait": "robot_normal", "text": "補給物資ヲ渡ス。受ケ取レ。" },
        { "kind": "giveItem", "itemKey": "apple", "message": "傷のあるりんごを手に入れた。" }, // itemKey で ItemData を引いて1個渡し、入手演出(絵+文)を出す
        { "kind": "giveItem", "itemKey": "umbrella" },   // message 省略 = 「傘を手に入れた。」を自動生成。1ステップ1個なので複数渡すなら並べる
        { "kind": "line", "portrait": "hero_smile", "text": "助かるよ。" }
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
        { "kind": "line", "portrait": "girl_smile", "text": "ここまで<emphasis>一緒に</emphasis>来られたね。" }
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
        { "kind": "line", "portrait": "hero_normal", "text": "この鍵で開きそうだ。" }
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
| `steps[]` | ○ | 会話ステップの並び。`line` / `choice` / `giveItem` / `giveWeapon` / `battle` / `command` |
| `nextProgress` | ○ | 終了時に進める進行度。**全イベント必須**で、`progress` の `value` **より大きく**する。イベントは進行度が `value` に一致したときだけ発火し、終了で進行度が進んで一致しなくなるので、**どのイベントもちょうど1回だけ発火する** |

### ステップの種類 (`steps[]` の `kind`)

| kind | フィールド | 動き |
| ---- | ---------- | ---- |
| `line` | `text`(必須) / `speaker` / `portrait` | 1行しゃべる。送り入力まで待つ |
| `choice` | `flag`(必須) / `options[]`(必須・**2〜3個**・`text` と `value`) | 選択肢を出し、選ばれた `value` を `flags[flag]` に書く |
| `giveItem` | `itemKey`(必須) / `message` | アイテムを1個渡し、**入手演出**(絵 + 文)を出す。1ステップ1個・複数渡すなら並べる |
| `giveWeapon` | `weaponKey`(必須) / `message` | 武器を渡し、**入手演出**(3Dモデル + 文)を出す |
| `battle` | なし | 戦闘。敵は JSON に書かず `EventTrigger` に配線する |
| `command` | `command`(必須) / `arg` | 会話UIの**演出コマンド**を1つ実行する(下表) |

> **`message` は省略可**。省略すると `giveItem` は `{アイテム名}を手に入れた。` を自動で組み立てる。
> `giveWeapon` の 3Dモデルは `WeaponData` にまだモデル参照が無いため、当面はダミーモデルで表示される。

> **`choice` の選択肢は 2個か3個**。選択肢UIが**3択ぶんの高さを基準に中央寄せ**する作りなので、
> 4つ以上だと会話ウィンドウに被る(取り込み時にエラーで弾く)。1個だけの `choice` も弾く
> (選ばせる意味が無く、書き間違いの方が多いため。フラグを無条件に立てたいなら分岐せず進行度で表す)。
> **1つの `text` は全角19文字くらいで改行が入り、2行(≒38文字)を超えると文字が縮む**。長い選択肢は
> 要点だけを書き、詳しい説明は直前の `line` に置く。

### text の演出タグ (`line` の `text` / `message` で使える)

| タグ | 効果 |
| ---- | ---- |
| `<wait=0.5>` | その位置でタイプ送出を 0.5 秒止める(0〜10秒) |
| `<shake>〜</shake>` | 囲んだ範囲の文字を揺らす |
| `<whisper>〜</whisper>` | 小さく・淡い色にする(囁き) |
| `<shout>〜</shout>` | 大きく・太字にする(叫び) |
| `<emphasis>〜</emphasis>` | 水色の太字で強調する |

- 上記以外の `<...>` は **TextMeshPro のタグとしてそのまま通す**(`<color=#FF0000>` など)。
- `portrait` を**書かない**と**地の文(ナレーション)**になる。立ち絵も名前欄も出ず、履歴では `NOTE` 扱いになる。
- `speaker` を `？？？` にすると**立ち絵を隠したまま**しゃべる(正体を伏せる演出)。

### 演出コマンド (`command` ステップの `command`)

| command | `arg` | 効果 |
| ------- | ----- | ---- |
| `window.hide` / `window.show` | — | 会話ウィンドウだけ隠す/戻す(立ち絵と背景を見せる) |
| `portrait.left.hide` / `portrait.right.hide` | — | 左/右の立ち絵を消す |
| `portrait.left.obscure` / `portrait.right.obscure` | — | 左/右の立ち絵を暗く伏せる(正体を隠す) |
| `portrait.left.reveal` / `portrait.right.reveal` | — | 伏せた立ち絵を戻す |
| `portrait.left.shake` / `portrait.right.shake` | — | 立ち絵を揺らす(動揺・被弾) |
| `portrait.left.jump` / `portrait.right.jump` | — | 立ち絵を跳ねさせる(驚き・喜び) |
| `wait` | **必須**(秒。0〜10) | その秒数だけ間を置く |
| `conversation.close` | — | 会話ウィンドウを閉じる(イベントは続く) |
| `camera.*` / `background.*` | 任意 | 会話UIの外へ委譲する枠。**受け手は未実装**なので、書いても今は何も起きない |

> 表に無いコマンド名は取り込み時にエラーで弾く(`camera.` / `background.` で始まるものだけは通す)。

### 条件タイプ (`conditions[]` の `type`)

| type | フィールド | 真になる条件 |
| ---- | ---------- | ------------ |
| `progress` | `value`(整数) | 進行度が `value` に**ちょうど一致**(必須・各イベント1つ以上) |
| `flag` | `key` / `value`(文字列) | フラグ `key` の値が `value` に一致(`choice` で書き込んだ値の分岐に使う) |
| `hasItem` | `itemKey`(文字列) | その **`itemKey` の「大事なもの」を1つ以上所持**していれば真 |

> **`hasItem` の制約**: `itemKey` は **大事なものカタログ**。**「どれか1つ持っていれば」= 特定の1 key を指定**する。複数 `hasItem` を並べると AND(=全部所持)になる。「AのkeyかBのkeyのどちらか」のような OR は無い。

> **`battle` ステップの制約**: `steps[]` の並び順は自由だが、**`battle` は必ず会話の途中に置く**(先頭・末尾は `line`。戦闘が単独・末尾にならない)。**1イベントにつき `battle` は最大1つ**。

---