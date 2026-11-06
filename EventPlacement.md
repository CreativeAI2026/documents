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
9. メニュー **Tools > CreativeAI > Dev State Window** を開き、イベントを選んで「このイベントの発火直前の状態にする」を押す。そのイベントが発火する状態(進行度・フラグ・大事なもの)と、開始位置(そのイベントのトリガーの位置)が入る。確認したいプレイヤーの状態(武器・HP)や開始位置は手でも直せる(HP は 1〜1000)。「保存して Play」で保存先に書いて再生する。保存先は **自分用の `Features/Event/dev_state.local.json`**(`.gitignore` 済み。共有の `dev_state.json` は書き換えない)
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
- `useSpawn` を `true` にすると、プレイヤーの開始位置を `spawnPosition`(足元の座標)・`spawnYaw`(向き)で上書きする。近くに歩ける床(NavMesh)があればその高さに合わせる。ウィンドウでイベントを選ぶと、そのトリガーの位置から入る
- 武器・HP は本番のプレイヤーリグが入るまでは反映されない

## 注意

- マップ本体(`Field_Area01`)にはイベントを置かない(担当者ごとにシーンを分けて git の競合を防ぐため)
- `Field_Area01` に差分が出てもコミットしない
- 共有の `dev_state.json` は書き換えない(変更はコミットしない)。自分の設定は `dev_state.local.json` に書く(`.gitignore` 済みなので、開始位置のような人ごとの設定も git に入らない)

## 発火のしかた

- プレイヤーが範囲に入ったとき、`events.json` の条件(`conditions`)をすべて満たせば発火する
- 範囲の中に居る間に進行度が変わったり、前のイベントが終わったりしたときも条件を見直す(続けて起きるイベントを同じ場所に重ねて置ける)
- 会話中・戦闘中は発火しない