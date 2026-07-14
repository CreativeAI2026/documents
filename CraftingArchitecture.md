# 調合システム アーキテクチャ

2つの素材を合成して新しいアイテムを作る。

- **実行場所** … **調合場所(フィールド上の固定地点)でのみ実行可能**。それ以外の場所では調合UIを開けない
- **対応カテゴリ** … **装備品同士 / 食材同士** のみ(カテゴリを跨いだ調合は不可・**武器は調合不可**)
- **ステータス** … **装備品は実行時に式でロール**(端末・個体差あり)。**食材はHP即時回復のみで固定ルール**(合成前後で一意に決まる。ロールもデータ保存も不要)。
- **素材(合成前)** … **ベースは決め打ちで固定: 食材10種 + 装備品15種 = 計25種**。名前・画像・説明とも同梱
- **結果(合成後)** … 名前・説明・画像を**事前生成してゲームに同梱**(local)。ステータスはデータに持たない(装備品は端末でロール、食材は固定ルールでその都度決まる)

---

## 全体像

```
【実行時：ゲーム(調合場所)・完全ローカル】
  A,B を選択 →(カテゴリ検証)→ RecipeHash を算出
     → 同梱カタログから結果(名前・説明・画像)を引く
     → ステータスを決定(装備品=端末でロール / 食材=固定ルール)
     → 結果を表示 → 確定 → 素材を消費
```

実行時に生成はしない(結果は事前生成・同梱済み)。有効なペアは全て用意済みなので **必ず結果がある**。ネット接続は一切不要。

---

## 関数レベルのフロー

**入口は `ICraftingService` の2関数だけ**。UI（調合画面）はこの2つしか知らない。内部の協力者（検証・ハッシュ・カタログ解決・ロール・インベントリ）は全部その裏に隠す。

**計算と確定を分ける**のが要点：
- `Craft` … 結果を**プレビューするだけ**。ここでは**状態を一切変えない**（素材は減らない）。
- `Confirm` … プレビューを**確定**。ここで初めて**素材消費・結果付与**が起きる（＝トランザクション境界＝「素材を減らす」と「結果を足す」が一括で成立）。ユーザーが結果を見てキャンセルできる余地を残すため。

> **セーブ(ディスク保存)は調合では起こさない。** Confirm が変えるのは**実行時インベントリ(メモリ上)だけ**で、ディスクへ書き込まない。永続化は**マニュアルセーブのみ**で、`Inventory` はセーブ時に全書きする。調合が勝手にセーブすると「オートセーブなし」の前提を破るため、ここでは分離する。

```mermaid
sequenceDiagram
    autonumber
    participant UI as 調合画面(UI)
    participant SVC as CraftingService
    participant VAL as CategoryValidator<br/>(ローカル)
    participant CAT as ResultCatalog<br/>(同梱・ローカル)
    participant ROLL as StatRoller<br/>(ローカル)
    participant INV as Inventory<br/>(実行時・メモリ上)

    UI->>SVC: Craft(a, b)
    SVC->>VAL: CanCraft(a, b)
    VAL-->>SVC: true / false
    Note over SVC: false なら CraftException で即 return（先に弾く）
    SVC->>SVC: RecipeHash.Of(a, b) → hash
    SVC->>CAT: GetResult(hash)
    Note over CAT: 同梱カタログから引く（全ペア用意済み → 必ず命中）
    CAT-->>SVC: ResultDef {name, desc, image, category}
    alt 装備品
        SVC->>ROLL: Roll(a.stats, b.stats)
        ROLL-->>SVC: StatVector（個体差あり）
    else 食材
        Note over SVC: Roll しない。HP回復量は固定ルール<br/>(合成前後で一意 → CraftingStatAlgorithm.md)
    end
    SVC-->>UI: CraftPreview {result, stats}（まだ消費しない）
    Note over UI: 結果を表示（ローカル完結なので待ちは無い。<br/>演出を入れるなら任意）
    UI->>SVC: Confirm(preview)
    SVC->>INV: Consume(a, b) + Add(result, stats)
    Note over INV: メモリ上の所持品を更新するだけ。<br/>ディスク保存はマニュアルセーブ時のみ(Specification プロジェクト前提)
    SVC-->>UI: 完了
```

---