# npc_placements.json フォーマット

物語班が手書きする `npc_placements.json` の **正本フォーマット** と、`model`(誰を置くか)に書くキーの **考え方** を定める。
仕組み全体・状態・条件・切り替えの考え方は [NpcPlacementSystem.md](./NpcPlacementSystem.md) を参照。

---

## 配置できる登場人物(`model`)

「誰を(`model`)」と「どこに(`id`)」は **別のフィールド** に分けて書く。`id` に誰かを埋め込まず、**`model` に誰か・`id` に場所** を書く。

- **`model` = 誰**。NPCモデルカタログ(視覚班が用意しシステム班が登録)のキー。**配置できるのは立ち絵カタログ([CharactersAndEvents.md](./CharactersAndEvents.md))の4キャラだけ** — `hero` / `girl` / `robot` / `gramophone`。フィールドNPCと会話立ち絵で **同じ語彙** にする。村人・通行人などのカタログ外キャラは作らない(登場人物が増えたら立ち絵カタログに追記され、それに揃える)
- **`id` = どこ**。シーン上の配置スロット名(`square` / `cave` …)。座標はスロット(シーン)が持ち、JSONには書かない
- これで `events.json` と同じ構図になる:JSONは **キーで「誰を・いつ」**(`model` + 進行度)を持ち、シーンは **位置**(スロット)を持つ

実体(モデル・アニメーション)は視覚班が用意し、システム班が `model` キーとしてカタログに登録する。物語班は `npc_placements.json` に **キーを書くだけ**で、実ファイルには触らない(`events.json` の立ち絵と同じ運用 → [CharactersAndEvents.md](./CharactersAndEvents.md))。

---

## npc_placements.json フォーマット

```jsonc
{
  "npcPlacements": [
    {
      "id": "square",              // どこに置くか。シーン上の配置スロット(位置)と対応。座標はJSONに書かない
      "model": "girl",             // 誰を置くか。NPCモデルカタログのキー
      "minProgress": 3,            // この進行度「以上」で表示(>=)
      "maxProgress": 6,            // この進行度「未満」まで表示。省略で上限なし(以降ずっと)
      "flag": { "key": "girl_choice", "value": "together" }  // 任意。指定時は進行度レンジとAND
    },
    {
      "id": "cave",
      "model": "girl",
      "minProgress": 6,
      "maxProgress": 9
    }
  ]
}
```

### フィールド

| フィールド | 必須 | 内容 |
| ---------- | ---- | ---- |
| `id` | ○ | **どこに置くか**(配置スロット)。シーン上の配置スロット(の `id` コンポーネント)と対応させる。場所が読める名前にする(例 `square` / `cave`)。**座標はJSONに書かない** |
| `model` | ○ | **誰を置くか**。NPCモデルカタログのキー(視覚班が用意しシステム班が登録。`events.json` の `portrait`/`enemyKey` と同じくキー参照)。立ち絵カタログの4キャラ `hero` / `girl` / `robot` / `gramophone` のみ([CharactersAndEvents.md](./CharactersAndEvents.md)) |
| `minProgress` | ○ | 表示を開始する進行度。比較は `>=`(以上)で [StoryProgressionSystem.md](./StoryProgressionSystem.md) と統一 |
| `maxProgress` | | 表示を終了する進行度。`[minProgress, maxProgress)` の **半開区間**(`maxProgress` 未満まで表示)。**省略で上限なし**(以降ずっと表示 = 実質無限大) |
| `flag` | | フラグ条件(任意)。`{ "key", "value" }`。指定フラグが指定値のときだけ表示(`choice` の分岐結果に連動)。**進行度レンジとは AND**(両方満たすと表示)。フラグの語彙は `events.json` の `flag` 条件と共通([CharactersAndEvents.md](./CharactersAndEvents.md)) |

> `events.json` の `progress` 条件は「指定値以上(単一しきい値)」だが、NPCは **範囲 `[min, max)`** で持つ点だけが異なる(だから `events.json` にマージせず独立JSONにする)。フラグの扱いは共通。

---

## 同じキャラが進行度で場所を変える場合

1体を移動させるのではなく、**場所ごとに別スロット(別 `id`)を置き、同じ `model` を指す2エントリで出し分ける**。JSON側も `id` ごとにエントリを分ける。

| `model`(誰) | `id`(どこ) | 位置 | 進行度レンジ |
| ------------- | ---------- | ---- | ------------ |
| `girl` | `square` | 村の広場 | `[3, 6)` |
| `girl` | `cave` | 洞窟前 | `[6, 9)` |

進行度が 5→6 になった瞬間、通知で両者が再評価され、広場の娘が消え洞窟の娘が現れる。位置は各スロット(=シーン)が持つ(切り替え機構は [NpcPlacementSystem.md](./NpcPlacementSystem.md))。

---

## Importer の検証(違反は取り込みエラー)

`npc_placements.json` の取り込みは `events.json` と同じ Importer 基盤で検証する([CharactersAndEvents.md](./CharactersAndEvents.md) の検証と揃える)。

- `id` はシーン上の配置スロットに対応するもののみ(未対応の `id`・重複した `id` は弾く)
- `model` は必須。NPCモデルカタログに存在するキーのみ(手書きの打ち間違いは前提とし Importer が弾く)
- `minProgress` は必須。`maxProgress` を指定する場合は `minProgress <= maxProgress`
- `flag` を指定する場合、`key` / `value` は `choice` で書かれるフラグと整合する(手書きの打ち間違いは前提とし Importer が弾く)
- 進行度の比較は `>=`(以上)
