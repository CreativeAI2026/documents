# 敵の実装(手順)

敵の **仕様**(雑魚なし・中ボス以上のみ・1戦闘につき1体 等)は [Specification.md](./Specification.md)「プロジェクト前提」。ここでは敵を Unity 上でどう作り、`events.json` の `battle` ステップに書く `enemyKey`(→ [CharactersAndEvents.md](./CharactersAndEvents.md))とどう繋ぐかをまとめる。

---

## enemyKey → EnemyData → Prefab の2段解決

物語班が書くのは **キー文字列だけ**(例 `{ "kind": "battle", "enemyKey": "wolf_boss" }`)。立ち絵や BGM は「キー → 1ファイル」の1段だが、**敵は1ファイルではない**ので2段になる。

| 段 | 何 | 中身 |
|---|---|---|
| 1段目 | `enemyKey` → **`EnemyData`**(ScriptableObject) | ステータス + 敵 Prefab への参照 |
| 2段目 | `EnemyData` → **Prefab** | 3Dモデル(`.fbx`) + マテリアル + アニメ + コンポーネントを合成したもの |

敵は「モデル + マテリアル + アニメ + 挙動」を **Prefab に合成**し、`EnemyData` がその **Prefab への参照とステータス**を持つ。だから `enemyKey → EnemyData → Prefab` の2段解決になる。

## 手順

| 手順 | 担当 | やること |
|---|---|---|
| 1 | 視覚班 | 敵の 3Dモデル・マテリアル・アニメ・コンポーネントを **Prefab に合成** |
| 2 | システム班 | **`EnemyData`(ScriptableObject)** を作り、ステータス + 手順1の Prefab 参照を設定。`id` を `enemyKey` にする |
| 3 | システム班 | `enemyKey` の有効一覧を物語班へ共有(手書きの打ち間違い対策) |
| 4 | 物語班 | `events.json` の `battle` ステップに `enemyKey` を**書くだけ** |

- **Prefab 参照は Inspector にドラッグ**(実体は GUID なのでリネーム・移動しても切れない。`PlayerRig` を差すのと同じ原理 → [PlayerImplementation.md](./PlayerImplementation.md))。
- **専用カタログは新設しない**。`EnemyData` を規約フォルダに置き `id`(= `enemyKey`)で引く運用にする。存在しないキーは Importer が弾く(→ [EventImplementation.md](./EventImplementation.md))。

## 実行時

`EventPlayer` が `battle` ステップに来たら、`enemyKey` で `EnemyData` を引き、その Prefab を出して戦闘を開始する:

```csharp
if (enemyCatalog.TryGet("wolf_boss", out var data))   // 1段目: key → EnemyData
    var enemy = Instantiate(data.prefab);             // 2段目: EnemyData → Prefab
```

戦闘モード(Field/Battle)の入り方・決着後の会話継続は [Specification.md](./Specification.md)「常駐アーキテクチャ」の `GameModeManager` と、再生フロー([EventImplementation.md](./EventImplementation.md))を参照。
