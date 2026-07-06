# NPC配置制御

進行度・フラグに応じて **フィールド上の常設NPCを出し分ける** 仕組みの正本。
「進行度3〜5の間はこのNPCが村に立つ」といった **常設NPCの表示制御** を扱う。
物語班が `npc_placements.json` を手書きし、システム班が Importer で取り込む(`events.json` と同じ流れ)。

```
[npc_placements.json]  物語班がテキストエディタで直接記述
   ↓ 取り込み + 検証
[Importer]  Unityエディタ拡張で JSON → 配置データ(ScriptableObject)
   ↓ 同梱
[ランタイム]  ProgressManager の変化を各NPC(表示制御)が見て自分で出/消する
```

- **思想** … `ProgressManager` は **状態(進行度・フラグ)だけ** を持ち、「その進行度でどのNPCがどこに居るか」は **知らない**。各NPC(`id`)が出現条件を引いて `ProgressManager` を見て **自分で出たり消えたりする**([GameMode.md](./GameMode.md) の自制パターン、[StoryProgressionSystem.md](./StoryProgressionSystem.md) の `EventTrigger` と同じ構造)
- このドキュメントが **NPC配置制御の仕組みの正本**。`npc_placements.json` の **書式・フィールド・検証と「誰(`model`)」の書き方** は [NpcPlacements.md](./NpcPlacements.md) にまとめる。進行度・フラグの正本は [StoryProgressionSystem.md](./StoryProgressionSystem.md)

**分業**: 物語班 = `npc_placements.json` を書く / 視覚班 = NPCモデルを用意 / システム班 = モデルカタログへの `model` キー登録・Importer・表示制御コンポーネント・配置スロット(`id`)のシーン配置。境界を JSON に置くことで作業が干渉しない。

---

## 状態・条件・配置の分離

「今 進行度いくつか」「その進行度でNPCが出るか(誰が)」「どこに居るか」は **別の軸** として分けて持つ。中央のテーブルに全部をまとめない。

| データ | どこが持つ | 常駐? | ファイル保存 |
| ------ | ---------- | ----- | ------------ |
| 進行度・フラグ(状態そのもの) | `ProgressManager` | 常駐する | する([SaveSystem.md](./SaveSystem.md)) |
| NPCの **出現条件と誰か**(進行度レンジ・フラグ・`model` キー) | `npc_placements.json`(物語班が編集 → ビルドに同梱) | ― | しない(同梱データ) |
| NPCの **配置スロット(座標)** | シーン上の配置スロット(`id`) | 常駐しない(シーンと共に生きる) | しない(シーンに焼き込む) |
| NPCの **モデル本体** | NPCモデルカタログ(視覚班が用意 → システム班が `model` キーで登録) | ― | しない(アセット) |

- **座標はデータに書かない**。シーン上に配置スロットとして置き、位置はスロット(シーン)が持つ(`events.json` に座標を書かず `id` で対応させるのと同じ方針 → [StoryProgressionSystem.md](./StoryProgressionSystem.md))
- **`npc_placements.json` が持つのは条件と `model` キーだけ**(座標は持たない)。「進行度→NPC座標表」のような **座標入りの中央テーブルは採らない**。座標を別データに二重管理することになり上記方針・単一シーンへの配置方針(実装上の注意)と衝突するため。条件と `model` を `id`(スロット)でシーンに紐付ける構図は `events.json`(条件・キーはJSON / 座標はシーン)と同一

---

## 切り替え機構

- `ProgressManager` の **変化通知**(進行度・フラグが変わったときのイベント)を各NPC(`id` を持つ表示制御コンポーネント)が **購読** する
- 通知を受けたら、**自分の `id` で引いた出現条件**(取り込んだ `npc_placements.json`)と現在値を照合し、**自分の表示/非表示を更新**する
- 中央は「状態を持って変化を通知するだけ」。出し入れの判断は各NPC側([GameMode.md](./GameMode.md) 3章の自制と同じ)

同じキャラを進行度で別の場所に移す場合は、1体を動かすのではなく **場所ごとに別スロット(別 `id`)を置き、同じ `model` を指す複数エントリで出し分ける**(書き方の例は [NpcPlacements.md](./NpcPlacements.md))。

---

## 実装上の注意

- **自分自身のGameObjectを非表示にしない**。表示制御コンポーネント(`id` を持ち条件を引く役)を **常にactiveな親(管制役)** に置き、出し入れするのは **子(見た目・当たり判定)** にする。コンポーネント自身のGameObjectを非表示にすると購読が外れ、進行度が進んでも復活できなくなる
- **起動順**: 表示制御は起動時に `ProgressManager` と取り込んだ配置データを参照するため、`ProgressManager` を先に生成しておく([GameSystems.md](./GameSystems.md) §6 のブートストラップ)。シーン開始・途中ロード時にも一度、初期評価を行う
- **常駐しない**: NPCと表示制御は単一のゲームプレイシーン上に置き、シーンと共に生きる。タイトルに戻れば一緒に消える(`EventTrigger` と同じ扱い → [GameSystems.md](./GameSystems.md) §6)。判定に必要な状態は常駐の `ProgressManager` が持つ

---

## npc_placements.json(正本フォーマット)

物語班が手書きする `npc_placements.json` の正本フォーマット・フィールド・Importer の検証ルール・「誰(`model`)」の書き方は [NpcPlacements.md](./NpcPlacements.md) にまとめる。

---

## 実装場所・Importer

- 表示制御コンポーネントは `EventTrigger` と同じ進行系として `Features/Core/Scripts/EventSystem/` 配下に実装する
- `npc_placements.json` を取り込む Importer は `events.json` と同じくエディタ拡張として `Features/Scenario/` 配下に実装する。検証内容は [NpcPlacements.md](./NpcPlacements.md) を参照
