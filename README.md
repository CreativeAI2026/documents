# ゲーム仕様ドキュメント

このリポジトリは、本プロジェクトの **仕様の正本(SSOT = Single Source of Truth)** です。

> **仕事を始める前に、まず [`GameSystems.md`](./GameSystems.md) を読んでください。** ここが全体の索引で、各詳細ドキュメントへの入口です。

---

## SSOT のルール

- **各トピックの正本は1ドキュメントだけ**。同じ内容を複数の場所に書かない(重複は矛盾のもと)。
- 他トピックに触れるときは **本文をコピーせず、正本へリンク** する。
- 実装・制作で仕様と食い違いを見つけたら、**コードや素材ではなく、まず正本のドキュメントを直す**。
- 各ドキュメントの冒頭に「このドキュメントが何の正本か」が書いてあります。

---

## ドキュメント一覧(正本の範囲)

### システム班向け(仕組み・実装)

| ドキュメント | 正本の範囲 |
| ------------ | ---------- |
| [`GameSystems.md`](./GameSystems.md) | **全体の索引**。プロジェクト前提(確定事項)/ プレイヤー・敵 / インベントリ・アイテム・装備(カテゴリとステータス)/ シーン(一覧・描画/ロード最適化)/ 常駐システム。各詳細は下記へ参照 |
| [`GameMode.md`](./GameMode.md) | **モード(Field / Battle)の正本**。戦闘モードの定義・モード別の可否・切り替え機構 |
| [`UISystem.md`](./UISystem.md) | **UIの正本**。UI / オーバーレイ一覧(各UIの中身・レイアウト・表示元)。シーンは `GameSystems.md` §5 |
| [`StoryProgressionSystem.md`](./StoryProgressionSystem.md) | **進行制御の正本**。進行度 + フラグ + 場所到達トリガーでイベントを駆動する仕組み |
| [`NpcPlacementSystem.md`](./NpcPlacementSystem.md) | **NPC配置制御の正本**。進行度・フラグに応じてフィールド上の常設NPCを出し分ける仕組み |
| [`SaveSystem.md`](./SaveSystem.md) | **セーブの正本**。保存内容・発動タイミング・ファイル形式(単一スロットJSON) |
| [`CraftingArchitecture.md`](./CraftingArchitecture.md) | **調合アーキテクチャの正本**。カタログDB・画像/名前の事前生成・実行時フロー |
| [`CraftingStatAlgorithm.md`](./CraftingStatAlgorithm.md) | **調合ステータス算出の正本**。素材2つから結果ステータスを決める数理モデル |

### 物語班向け(手書きする JSON の書式)

物語班はまずこの2本を読めばよい。仕組みまで知りたいときは、対応するシステム班ドキュメント([`StoryProgressionSystem.md`](./StoryProgressionSystem.md) / [`NpcPlacementSystem.md`](./NpcPlacementSystem.md))へ。

| ドキュメント | 正本の範囲 |
| ------------ | ---------- |
| [`CharactersAndEvents.md`](./CharactersAndEvents.md) | **`events.json` フォーマットの正本** + 登場人物・立ち絵(`portrait`)キーのカタログ |
| [`NpcPlacements.md`](./NpcPlacements.md) | **`npc_placements.json` フォーマットの正本** + `model`(配置する登場人物)キーの考え方 |