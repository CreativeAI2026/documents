# イベント配置手順

`events.json` に書いたイベントを、フィールド上の場所に置いて発火させる手順。

## 置き場所

- イベントは各エリアの **イベント用シーン** に置く(`Scenes/Field/Field_Area01_Events.unity`)
- このシーンは `Field_Area01` の再生時に自動で重ねて読み込まれる

## 手順

1. `events.json` を取り込んで `Features/Event/Data/` にイベントの `.asset` を作る(手順は [`ConversationPreview.md`](./ConversationPreview.md))
2. `Field_Area01` を開いたまま、Project の `Field_Area01_Events` を右クリック → **Open Scene Additive**
3. `Field_Area01_Events` の `Events` の子に空の GameObject を作り、イベント id がわかる名前を付ける
4. Box Collider などの Collider を付け、発火させたい範囲に合わせて位置・大きさを調整する
5. `EventTrigger` を付ける(Collider の **Is Trigger** は自動でオンになる)
6. Inspector の **Event** 欄にイベントの `.asset` をドラッグ
7. `battle` を含むイベントなら **Enemy** に敵 Prefab を入れる(出現位置を変えたいときだけ **Spawn Point** に空の GameObject を指定)
8. `Field_Area01_Events` を保存する
9. メニュー **Tools > CreativeAI > Dev State Window** を開き、イベントを選んで「このイベントの発火直前の状態にする」を押す。発火する状態(進行度・フラグ・大事なもの)と開始位置が入る。武器・HP・開始位置は手でも直せる(HP は 1〜1000)。「保存して Play」で自分用の `Features/Event/dev_state.local.json` に書いて再生する(共有の `dev_state.json` は書き換えない)
10. `Field_Area01` を開いた状態で Play し、範囲に入って発火を確認する

```json
{
  "progress": 8,
  "flags": [ { "key": "girl_choice", "value": "together" } ],
  "importantItems": [ "mysterious_key" ],
  "weapons": [ "sword", "bow" ],
  "selectedWeapon": "sword",
  "hp": 50,
  "useSpawn": true,
  "spawnPosition": { "x": -592, "y": -60.8, "z": -30.5 },
  "spawnYaw": 0
}
```

- `dev_state.local.json`(無ければ共有の `dev_state.json`)が効くのは `Field_Area01` を直接 Play したときだけ(タイトルから始めたときは使われない)
- `useSpawn` を `true` にすると、開始位置を `spawnPosition`(足元の座標)・`spawnYaw`(向き)で上書きする
- 武器・HP は本番のプレイヤーリグが入るまでは反映されない

## 注意

- マップ本体(`Field_Area01`)にはイベントを置かない(担当者ごとにシーンを分けて git の競合を防ぐため)
- `Field_Area01` に差分が出てもコミットしない
- 小物の置き場所を変えたら NavMesh の焼き直し(Tools > CreativeAI > Map > Bake NavMesh Field_Area01)が必要。`Field_Area01` と NavMesh のデータが変わるので、**焼き直しとそのコミットはマップ担当だけが行う**。それ以外の人は差分が出てもコミットしない
- 共有の `dev_state.json` の変更はコミットしない。自分の設定は `dev_state.local.json`(`.gitignore` 済み)に書く

## 発火のしかた

- プレイヤーが範囲に入ったとき、`events.json` の条件(`conditions`)をすべて満たせば発火する
- 範囲の中に居る間に進行度やフラグが変わったり、前のイベントが終わったりしたときも条件を見直す(続けて起きるイベントを同じ場所に重ねて置ける)
- 会話中・戦闘中は発火しない

## 合成装置を置く

合成装置に近づくと「[E] 合成する」を出し、押すと合成画面を開く。

1. 装置の近く(装置のあるプロップのシーン)に、**Is Trigger** の Collider を持つ GameObject を置く
2. `CraftStationInteractor` を付ける(文言は Inspector の **Label** で変えられる)

移動中(Field)で、会話中でないときだけ操作できる(扉の `DoorInteractor` と同じ)。
