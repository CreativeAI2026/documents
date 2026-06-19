# ストーリー進行システム

進行度(整数1つ)で物語の進行を管理し(分岐はフラグで併用)、**エリア侵入時に条件付きでイベント(会話・戦闘)を発火**する。
物語班が `events.json` を手書きし、システム班が Importer で取り込む。

```
[events.json]  物語班がテキストエディタで直接記述
   ↓ 取り込み + 検証
[Importer]  Unityエディタ拡張で JSON → EventDefinition(ScriptableObject)
   ↓ 同梱
[ランタイム]  ProgressManager + EventTrigger が進行度・条件でイベントを発火
```

**分業**: 物語班 = `events.json` を書く / 絵班・音班・敵班 = 立ち絵・BGM・敵データを用意 / システム班 = データモデル・カタログ登録・Importer・ランタイム。境界を JSON に置くことで作業が干渉しない。

---

## 進行度とフラグ

メインストーリーの進行は **進行度(整数1つ)** で表す(例: 0→1→2…)。分岐はフラグで持つ(後述)。

- 進行度さえ保存すればどこからでも再開でき、デバッグも書き換えるだけ
- 「ボスを倒したか」のような状態も進行度で判定する(勝たないと進行度が進まない。例: 進行度6以上なら撃破済み)
- 「特定アイテム所持」は進行度とは別軸なので、条件ではインベントリ側を読む

進行度に畳めない分岐は **フラグ(key → 値)** で併用する。プレイヤーの2択など、進行度の大小では表せない独立した状態を保存する(例: `fox_choice = "help"`)。選択は会話中の `choice` ステップで書き込み、`flag` 条件で後のイベントを分岐させる。

実装は `ProgressManager`(進行度・フラグの保持と条件判定)と `EventTrigger`(`Features/Core/Scripts/EventSystem/`)。

---

## イベント(EventDefinition)

イベント = **条件** + **会話** + **終了時に進める進行度**。会話はイベント内に直接持つ(1イベント1会話)。ScriptableObject で表す。

**発火**: プレイヤーが対象トリガーに侵入し、条件を **すべて満たす** と発火する。どのトリガー(エリア)でどのイベントが発火するかは、システム班がシーン上のトリガーにイベントの `id` を対応させて決める(座標は JSON に書かない)。

発火条件は固定の3タイプ(物語班は選んで値を埋める)。

| 条件タイプ | 意味 |
| ---------- | ---- |
| `progress` | メイン進行度が指定値以上 |
| `hasItem`  | 指定アイテムを持っている |
| `flag`     | 指定フラグが指定値(`choice` で書いた分岐結果) |

新しい条件タイプ(例:「HP半分以下」)が要るときは評価ロジックごとコードで追加する。

**会話ステップ**: 会話は4種類のステップの並び。任意の順に挟める。

| ステップ | 内容 |
| -------- | ---- |
| `line` | セリフ(話者名・立ち絵キー・テキスト)。立ち絵はセリフごとに切替可(表情差分) |
| `battle` | 戦闘(敵キー)。戦闘が終わると会話の続きから再生 |
| `giveItem` | アイテム付与(アイテムキー)。別イベントの `hasItem` 条件と対になる |
| `choice` | 2択以上を提示し、選んだ値をフラグに書く。別イベントの `flag` 条件と対になる |

- **戦闘は必ず会話の途中**(先頭・末尾は必ず `line`)。戦闘が単独・末尾にならない
- 勝利 → 会話の続き → 終了時に進行度UP / 敗北 → 直近セーブから再開(`GameSystems.md` のリスポーン仕様)
- 戦闘は勝敗を記録しない(状態は進行度で判定する)

> 戦闘は **同じフィールドシーン上の戦闘モード**(別シーンへは遷移しない)。会話途中で戦闘ステップに到達したら戦闘モードに入り、決着後にモードを抜けて会話を続ける。

---

## カタログ(キー参照)

立ち絵・BGM・敵の **実体は各班が用意** し、システム班がキーを登録する。物語班は `events.json` に **キーを書くだけ**で、実ファイルには触らない。

```
[各班] 立ち絵PNG/BGM/敵データを用意
   → [システム班] カタログに登録(キー ⇔ 実体)  例: fox_smile → portraits/fox_smile.png
   → [物語班] events.json にカタログのキーを書き写す
```

システム班は有効キー一覧(立ち絵/BGM/敵/アイテム)を物語班に共有する。手書きなので打ち間違いは前提とし、**存在しないキーは Importer が弾く**。

---

## events.json(正本フォーマット)

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
        { "kind": "line",     "speaker": "キツネ", "portrait": "fox_smile",   "text": "やあ、よく来たね。" },
        { "kind": "battle",   "enemyKey": "wolf_boss" },
        { "kind": "line",     "speaker": "キツネ", "portrait": "fox_smile",   "text": "見事だ。これを持っていけ。" },
        { "kind": "giveItem", "itemKey": "old_key" },
        { "kind": "choice", "flag": "fox_choice", "options": [   // 選んだ値を flags["fox_choice"] に書く
            { "text": "一緒に行く",   "value": "together" },
            { "text": "ひとりで行く", "value": "alone" }
        ]},
        { "kind": "line",     "speaker": "キツネ", "portrait": "fox_smile",   "text": "…そうか。" }
      ],
      "nextProgress": 6            // 終了時に進める進行度(任意。省略時は進めない)
    },
    {
      "id": "fox_reunion",
      "conditions": [
        { "type": "progress", "value": 8 },
        { "type": "flag", "key": "fox_choice", "value": "together" }   // 「一緒に行く」を選んだ人だけ発火
      ],
      "steps": [
        { "kind": "line", "speaker": "キツネ", "portrait": "fox_smile", "text": "ここまで一緒に来られたな。" }
      ]
    }
  ]
}
```

**Importer の検証(違反は取り込みエラー)**:

- `steps` の先頭・末尾は必ず `line`(`battle` / `giveItem` / `choice` は単独・末尾にならない)
- `portrait` / `enemyKey` / `itemKey` / `startBgm` はカタログに存在するキーのみ
- `conditions[].type` は `progress` / `hasItem` / `flag` のみ

> 進行度の比較は `>=`(以上)。

---

## Importer

`events.json` を取り込む Importer はエディタ拡張として `Features/Scenario/` 配下に実装する。検証内容は上記のとおり。
