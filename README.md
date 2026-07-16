# ゲーム仕様ドキュメント

このリポジトリは、本プロジェクトの **仕様をまとめたもの** です。

---

## 書き方のルール

- **1つのトピックは1ドキュメントにだけ書く**。同じ内容を複数の場所に書かない(重複は矛盾のもと)。
- 他トピックに触れるときは **本文をコピーせず、そのドキュメントへリンク** する。
- 実装・制作で仕様と食い違いを見つけたら、**コードや素材ではなく、まずドキュメントを直す**。

---

## 読み方

- **まず [`Specification.md`](./Specification.md)(仕様書)を必ず読む**。ゲーム全体の前提・仕様はここに集約されている。
- そのうえで、**自分の役割に応じて**下記の実装・その他ドキュメントを必要なぶんだけ読む(例: プレイヤー担当 → `PlayerImplementation.md`、イベント担当 → `EventImplementation.md`)。
- 物語班は [`CharactersAndEvents.md`](./CharactersAndEvents.md) のみを読めばよい。

## ドキュメント一覧

### 仕様(What / Why)

| ドキュメント | 扱う範囲 |
| ------------ | ---------- |
| [`Specification.md`](./Specification.md) | **仕様書の本体**。プロジェクト前提(確定事項)/ プレイヤーステータス / インベントリ・アイテム・装備 / シーン / 進行管理 / UI・オーバーレイ / 常駐アーキテクチャ |
| [`CharactersAndEvents.md`](./CharactersAndEvents.md) | **`events.json` フォーマット** + 登場人物・立ち絵(`portrait`)キーのカタログ(物語班が手書きする書式) |

### 実装・アルゴリズム(How)

| ドキュメント | 扱う範囲 |
| ------------ | ---------- |
| [`PlayerImplementation.md`](./PlayerImplementation.md) | **プレイヤー実装**。リグ生成・常駐・単一化の手順 + Title からの生成フロー |
| [`EventImplementation.md`](./EventImplementation.md) | **イベント発火の実装**。取り込み〜トリガー設置〜再生の手順(`battle` の **敵の作成・配置・戦闘** を含む)+ 呼び出しフロー図 |
| [`UIImplementation.md`](./UIImplementation.md) | **UI 実装**。Prefab/Canvas 構成・`UiRouter`(排他)・`HudIconBar` のモード連動(右上アイコンバーの出し入れ) |
| [`CraftingArchitecture.md`](./CraftingArchitecture.md) | **調合アーキテクチャ**。カタログDB・画像/名前の事前生成・実行時フロー |
| [`CraftingStatAlgorithm.md`](./CraftingStatAlgorithm.md) | **調合ステータス算出**。素材2つから結果ステータスを決める数理モデル |