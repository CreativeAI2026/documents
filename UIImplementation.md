# UI の実装(手順)

UI の **仕様**(どんな UI がいつ出るか・呼び出しの設計)は [Specification.md](./Specification.md)「UI / オーバーレイ」。ここでは各 UI を Unity 上でどう Prefab 化し、どう出し分けるかをまとめる。

---

## セットアップの順番

1. UI を Prefab 化(Project に `.prefab`)
2. `UiRouter` / 切替役スクリプトを用意(`[SerializeField]` スロット)
3. スクリプトをオブジェクトに載せる
4. Inspector で各 Prefab をスロットにドラッグして紐づけ(← 参照は実体が揃ってから。**最後**)

> 参照を張れるのは「既に存在するもの」だけなので、Prefab(1)とスロット(2)が揃って初めて紐づけ(4)ができる。この「アセット作成 → コード → 最後に Inspector で配線」という順番は Player / Enemy の実装でも同じ。

---

## 1. Prefab / Canvas 構成

- **1つの UI = 1つの Prefab**(ルートに `Canvas` を持つ)。中身(ボタン・タブ・テキスト)ごと Prefab に合成し、`.prefab` として Project に置く。シーンには埋め込まない。
- HUD・会話UI など**シーン中に出す UI** は、シーン側(または UI 管理オブジェクト)が Prefab を `Instantiate` して重ねる。開閉は GameObject の表示/非表示で行い、シーン遷移はしない。
- **ロードオーバーレイだけは `DontDestroyOnLoad` の常駐 Canvas**(アプリ常駐 → [Specification.md](./Specification.md)「常駐アーキテクチャ」)。全シーンのロードを上から覆うため、他の UI より前面のソート順に置く。

| UI | Prefab | 出し方 |
| --- | --- | --- |
| 移動HUD / 戦闘HUD / 戦闘食材UI | 各1つ | モード連動で自動切替(下記「HUD の自動切替」) |
| キャラクター / インベントリ / セーブ / 調合 | 各1つ | `UiRouter.Open(id)` で排他表示(下記「UiRouter」) |
| 会話UI | 1つ | `EventPlayer` が表示/非表示 |
| ロードオーバーレイ | 1つ(常駐) | `SceneManager.LoadSceneAsync` の進捗に同期 |

---

## 2. UiRouter(操作で開く UI の排他制御)

「操作で開く UI」(キャラ/インベ/セーブ/調合)は **`UiRouter.Open(id)` の1本**を入口にする。今開いている UI を閉じてから次を開く＝**常に1つだけ**(排他)。

`UiRouter` は **MonoBehaviour** にし、「`UiId` → Prefab」の対応を `[SerializeField]` で持つ。**Inspector で各 UI Prefab をスロットにドラッグして紐づける**(GUID 参照。リネーム・移動に強く、未割当は `None` と出て目視できる ― `Resources.Load` の文字列指定より安全。理由は [PlayerImplementation.md](./PlayerImplementation.md))。`Dictionary` は直接シリアライズできないので **`Entry[]`(id + prefab)** で並べる。

```csharp
public sealed class UiRouter : MonoBehaviour
{
    [Serializable] struct Entry { public UiId id; public GameObject prefab; }
    [SerializeField] Entry[] entries;         // ← Inspector で各UIのPrefabをドラッグして埋める

    GameObject current;                       // 今開いている UI(無ければ null)

    public void Open(UiId id) {
        if (current != null) Close();         // 排他: 開く前に今のを閉じる
        var prefab = System.Array.Find(entries, e => e.id == id).prefab;
        current = Instantiate(prefab);
    }
    public void Close() { if (current) Destroy(current); current = null; }
}
```

- 「今どれが開いているか」を持つだけの **薄いルート**。全 UI の中身を知る重い UIManager は作らない。
- 呼び出し側(HUD の右上アイコン・調合機の接近判定)は `UiRouter.Open(id)` を叩くだけ。
- HUD(移動/戦闘/戦闘食材UI)も同様に、切替役が各 Prefab/オブジェクトへの参照を `[SerializeField]` で持ち、Inspector でドラッグして紐づける(下記「HUD の自動切替」)。

---

## 3. HUD の自動切替(モード連動)

移動HUD ⇄ 戦闘HUD・戦闘食材UI は、`GameModeManager.OnModeChanged` を**購読して**アクティブを切り替える(→ [Specification.md](./Specification.md)「常駐アーキテクチャ」の `GameModeManager`)。プレイヤー操作では開かない。

```csharp
gameMode.OnModeChanged += m => {
    fieldHud.SetActive(m == GameMode.Field);
    battleHud.SetActive(m == GameMode.Battle);   // 戦闘食材UIも同時に
};
```

- **Battle で開かせない UI**(セーブ・インベントリ・キャラ・調合)は、移動HUD が戦闘HUD に差し替わり**右上アイコンが画面から消える**ことで担保する(中央で個別に禁止しない)。
- 戦闘中に開けるのはセット済みの戦闘食材UI(最大3枠)のみ。

---

## 4. 会話UI・ロードオーバーレイ

- **会話UI** … `EventPlayer` が `line`/`choice` ステップで表示し、テキスト・立ち絵(`portrait` キー)・選択肢を描く。再生フローは [EventImplementation.md](./EventImplementation.md)。
- **ロードオーバーレイ** … 常駐 Canvas 上のプログレス表示。`SceneManager.LoadSceneAsync` の `progress` を購読してバー等を更新し、完了で隠す。シーン化しない(「ロード画面を出すためにロード画面をロードする」入れ子を避ける)。
