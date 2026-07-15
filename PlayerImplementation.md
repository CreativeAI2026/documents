# プレイヤーの実装(手順とフロー)

プレイヤーを Unity 上でどう実装するか。ステータスの **仕様**(素の値 + 装備の補正の合算)は [Specification.md](./Specification.md)「プレイヤーステータス」。

---

## 1. プレイヤーリグの配線(Unity Editor でやること)

**① `PlayerRig` Prefab を作る**
- **モデル + コントローラ + `PlayerStatus` + メインカメラ** を1つの Prefab にまとめ、Project に置く(**どのフィールドシーンにも置かない**)
- ルートの **Tag を `Player`** にする(`EventTrigger` が侵入判定にこのタグを使う → [EventImplementation.md](./EventImplementation.md))
- **Collider + Rigidbody** を付ける(`OnTriggerEnter` が飛ぶ前提)

**② Prefab をスロットにドラッグ**
- `01_Title` を開く → `GameStarter` を選択 → Inspector の **Player Rig Prefab** スロットへ、①の `PlayerRig` Prefab をドラッグ
- ドラッグ参照は GUID 保持でリネーム・移動に強い。**未割当なら `None` 表示**で、Play 時に「playerRigPrefab 未割当」警告が出る(フィールドは読み込めるがプレイヤーは出ない)

### 確認(Play)
- Title →「はじめる」→ フィールドにプレイヤーが **1体**出る
- エリアを移動しても **増えない/消えない**(同じ1体が持ち越される)
- 「タイトルに戻る → 再開」でも二重化しない

---

## 2. 関数レベルのフロー(Title からのリグ生成・常駐・単一化)

「はじめる/続きから」でプレイヤーリグを **1体だけ**生成して常駐させる流れ。生成順は **マネージャ → Inventory → プレイヤー**(プレイヤーが `Start` で `GameModeManager`/`Inventory` を読むため。spec §6.1)。既に `Player` タグが居れば作らない(連打・タイトル復帰での二重化防止=単一化)。

```mermaid
sequenceDiagram
    autonumber
    participant T as TitleUI(はじめる/続きから)
    participant SB as SessionBootstrap
    participant INV as InventoryManager
    participant BR as BattleRunner
    participant GS as GameStarter
    participant SC as SceneController

    T->>SB: EnsureSession()
    Note over SB: ProgressManager / GameModeManager / EventPlayer を生成<br/>(Instance で二重生成ガード=冪等)
    T->>INV: EnsureResident()
    T->>BR: new BattleRunner() を BattleRunnerService に登録
    T->>GS: EnsurePlayer()
    alt 既に Player タグが居る
        GS-->>T: 既存リグを返す(生成しない=単一化)
    else 未生成
        GS->>GS: Instantiate(PlayerRig Prefab)
        GS->>GS: DontDestroyOnLoad(隠しシーンへ移し常駐)
        GS-->>T: 生成したリグを返す
    end
    T->>SC: LoadScene(フィールド)
    Note over T,SC: 「続きから」は Load() で進行度・所持品を復元し<br/>シーン起動後に RestorePlayerState(座標・現在HP)
```

- 生成はすべて Title シーンが担う。フィールドシーンにはリグを置かない(→ [Specification.md](./Specification.md)「常駐アーキテクチャ」)。
- 「タイトルに戻る」でセッション常駐だけ破棄し、次の開始で作り直す。

---

## 3. 関数レベルのフロー(PlayerStatus)

**`PlayerStatus` が最終値を持つ唯一の場所**(素の値 + 装備の補正の合算 → [Specification.md](./Specification.md)「プレイヤーステータス」)。

```mermaid
sequenceDiagram
    autonumber
    participant CU as キャラクターUI(装備変更)
    participant INV as InventoryManager
    participant FOOD as 食材使用(インベントリ/戦闘食材UI)
    participant PS as PlayerStatus
    participant BT as 戦闘システム
    participant HUD as HUD

    Note over HUD: OnStatsChanged / OnHpChanged を購読
    CU->>INV: SetEquipped(item, on/off)
    INV-->>PS: EquipmentChanged(静的イベント)
    PS->>PS: RecalculateFromInventory(装備補正を再計算)
    PS-->>HUD: OnStatsChanged
    FOOD->>PS: Heal(amount)
    PS-->>HUD: OnHpChanged(HP表示更新)
    BT->>PS: CurrentAttackPower / CurrentDefense / CurrentHp を読む
    BT->>PS: TakeDamage(amount, isCritical)
    PS-->>HUD: OnHpChanged(HP表示更新)
```
