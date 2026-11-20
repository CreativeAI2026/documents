# イベントレシピ(やりたいこと別の書き方)

`events.json` で「こういうイベントを作りたい」から、書き方を探すページ。項目の細かい仕様は [`ScenarioReference.md`](./ScenarioReference.md)、置き方は [`EventPlacement.md`](./EventPlacement.md) を見る。

## まず3つの軸で決める

イベントは、次の3つを決めると作れる。

| 軸 | 選ぶもの | 書く場所 |
| -- | -------- | -------- |
| **いつ出るか** | 進行度・フラグ・持ち物 | `conditions` |
| **どう始まるか** | 範囲に入ったら自動 / [E] を押したら | `activation` |
| **何回出るか** | ストーリーで1回 / 1回だけ / 何度でも | `repeat` と `nextProgress` |

| やりたいこと | `activation` | `repeat` | `nextProgress` | `conditions` の `progress` |
| ------------ | ------------ | -------- | -------------- | -------------------------- |
| ストーリーを順番に進める | 省略(自動) | 省略 | **必須** | `value`(必須) |
| 場所を調べる(何度でも) | `"interact"` | `"always"` | **書けない** | 任意(`min` / `max` も可) |
| 場所を調べる(1回だけ) | `"interact"` | `"once"` | 任意 | 任意(`min` / `max` も可) |
| 範囲に入ったら1回だけ出る | 省略(自動) | `"once"` | 任意 | 任意(`min` / `max` も可) |

> `repeat` を**省略**したイベントだけ、`progress` を `value` で書くのが必須。`repeat` を書いたイベントは、ストーリーの順番に縛られない。

---

## レシピ

### 1. ストーリーを順番に進める

進行度 5 のときだけ発火し、終わると進行度が 6 になる。5 でなくなるので、二度と発火しない。

```jsonc
{
  "id": "station_awakening",
  "conditions": [ { "type": "progress", "value": 5 } ],
  "steps": [ { "kind": "line", "text": "雨音の向こうで、古いレコードが鳴っている。" } ],
  "nextProgress": 6
}
```

### 2. いつでも何度でも調べられる(進行度 N 以上)

置き物を調べるイベント。進行度は進めない。`min` を書くと「その進行度から先はずっと」出る。

```jsonc
{
  "id": "bench_note",
  "activation": "interact",
  "repeat": "always",
  "conditions": [ { "type": "progress", "min": 5 } ],
  "steps": [ { "kind": "line", "text": "ベンチの裏に、古いメモが挟まっている。" } ]
}
```

### 3. ある期間(進行度の範囲)だけ出る

`min` と `max` を両方書くと、範囲(両端を含む)になる。片方だけなら「以上」「以下」。

```jsonc
{
  "id": "rain_window",
  "activation": "interact",
  "repeat": "always",
  "conditions": [ { "type": "progress", "min": 5, "max": 8 } ],   // 進行度 5〜8 の間だけ
  "steps": [ { "kind": "line", "text": "窓の外は、まだ雨が降っている。" } ]
}
```

> ここでの「期間」は**進行度の範囲**。現実の時間や日数では決められない(下の「できないこと」)。

### 4. 1回だけ調べられる

見終えたら二度と出ない。進行度は進めなくてよい。

```jsonc
{
  "id": "old_locker",
  "activation": "interact",
  "repeat": "once",
  "conditions": [],
  "steps": [ { "kind": "giveItem", "itemKey": "apple", "message": "ロッカーの奥で、りんごを見つけた。" } ]
}
```

### 5. 選択肢で分岐する

`choice` で選んだ値をフラグに書き、`branch` で出し分ける。分岐は1段まで。枝のあとは共通の続きに合流する。

```jsonc
{ "kind": "choice", "flag": "girl_choice", "options": [
    { "text": "一緒に行く",   "value": "together" },
    { "text": "ひとりで行く", "value": "alone" }
]},
{ "kind": "branch", "flag": "girl_choice", "cases": {
    "together": [ { "kind": "line", "text": "一緒に行けるなら心強いです。" } ],
    "alone":    [ { "kind": "line", "text": "気をつけてくださいね。" } ]
}}
```

選択肢は2〜3個。後のイベントで同じフラグを `conditions` に使うこともできる(`{ "type": "flag", "key": "girl_choice", "value": "together" }`)。

### 6. 持ち物やフラグで出し分ける

- **出るかどうか**を決める → `conditions` に `hasItem` / `flag` を足す
- **会話の中身**を変える → `branch` の `hasItem` + `then` / `else`

```jsonc
"conditions": [
  { "type": "progress", "min": 5 },
  { "type": "hasItem", "itemKey": "mysterious_key" }   // 謎の鍵を持っているとき
]
```

`hasItem` は**大事なものだけ**が対象。

### 7. 「〜でないとき」だけ出す(`not`)

どの条件にも `"not": true` を付けると、「満たさないとき」に成立する。

```jsonc
"conditions": [
  { "type": "progress", "min": 5 },
  { "type": "hasItem", "itemKey": "card_key", "not": true }   // カードキーをまだ持っていないとき
]
```

### 8. アイテムを使わせる

持ち物から1つ選ばせ、正解なら進む。`activation` は **`"interact"` が必須**。詳しくは [`ItemUse.md`](./ItemUse.md)。

### 9. 戦闘を入れる

`{ "kind": "battle" }` を会話の途中に置く。敵は JSON に書かず、シーンの `EventTrigger` に配線する。`activation: "interact"` のイベントには書けない。

---

## できること / できないこと

| やりたいこと | できる? | 書き方 |
| ------------ | ------- | ------ |
| 進行度がちょうど N のとき | ○ | `progress` の `value` |
| 進行度 N 以上 / N 以下 | ○ | `progress` の `min` / `max` |
| 進行度 N〜M の間だけ | ○ | `progress` の `min` と `max` |
| フラグの値で出し分ける | ○ | `flag` 条件 / `branch` |
| 大事なものを持っているか | ○ | `hasItem`(大事なものだけ) |
| 「〜でないとき」 | ○ | 各条件に `"not": true` |
| 何度でも / 1回だけ | ○ | `repeat` の `"always"` / `"once"` |
| 現実の時間・日数・曜日で出す | ✕ | なし(進行度かフラグで代用) |
| 「AまたはB」(OR)の条件 | ✕ | なし(同じ内容のイベントを2つ書く) |
| 装備品・食材を持っているか | ✕ | なし(大事なものだけ) |
| 分岐の中に分岐 / 分岐の中で `useItem` や `battle` | ✕ | 分岐は1段まで |
