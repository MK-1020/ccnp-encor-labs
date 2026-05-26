# CCNP ENCOR 試験対策 第24回: Cisco MST設定：spanning-tree mst costで経路を調整する

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **`MSTI` ごとの経路選択を interface cost で調整する** ところを扱います。  
前回の root bridge 設計だけではなく、同じ root へ向かう複数経路のどちらを選ばせるかも instance ごとに調整できます。

## 今回の確認ポイント

- `spanning-tree mst <instance-number> cost` をどこで使うか
- なぜ priority とは別に cost tuning が必要になるのか
- `show spanning-tree mst` と `show spanning-tree mst interface` のどこで、経路変更を読むか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| MST Tuning | 高 | interface cost で経路を変えること | `show spanning-tree mst 1` / `mst interface ...` | 本編で検証 | tuning 論点の中心のため |
| MST interface cost | 高 | `MSTI` 単位で cost を変えること | `show spanning-tree mst interface <interface-id>` | 本編で検証 | instance 単位 tuning の理解に必要なため |

## 前回とのつながり

前回は、`MSTI` ごとに root bridge を分けました。  
今回はその続きとして、同じ root へ向かう経路のうち、どちらを選ばせるかを interface cost で調整します。

## 検証ステータス

| 項目 | 状態 |
|---|---|
| EVE-NG構築 | TODO |
| 設定投入 | TODO |
| show確認 | TODO |
| 疎通確認 | 対象外 |
| Wireshark確認 | 対象外 |
| 結果反映 | TODO |

## 検証環境

| 項目 | 内容 |
|---|---|
| 使用イメージ | IOSvL2 |
| 使用ノード | SW1〜SW4 |
| 例として調整するノード | SW3 |
| 主な確認コマンド | `show spanning-tree mst 1` / `show spanning-tree mst interface gi0/0` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、例として SW3 から `MSTI 1` の root へ向かう経路を見ます。直接 SW1 へ向かう経路と、SW2 経由の経路のどちらを選ばせるかを cost で調整します。

## 初期状態

前回までに `MSTI 1` / `MSTI 2` の root bridge 設計が入っている前提で進めます。  
まずは tuning 前の path cost と role / state を確認します。

確認コマンド:

```cisco
show spanning-tree mst 1
show spanning-tree mst interface gi0/0
show spanning-tree mst interface gi0/1
```

期待する結果:

- tuning 前の cost が確認できること
- SW3 がどちらの uplink を使って root へ向かっているか読めること

## 実際の結果

```markdown
TODO: tuning 前の `show spanning-tree mst 1`、`show spanning-tree mst interface gi0/0`、`show spanning-tree mst interface gi0/1` の出力を貼る。
```

## 設定

ここでは例として、SW3 の SW1 直結側 interface に対して `MSTI 1` の cost を上げ、SW2 経由を選ばせる前提を作ります。

SW3で入力:

```cisco
configure terminal
interface gi0/0
 spanning-tree mst 1 cost 30000
end
write memory
```

## 設定反映・期待結果の確認

ここでは、`show spanning-tree mst 1` と `show spanning-tree mst interface` を使って **`MSTI 1` の経路選択が変わったか** を確認します。

確認コマンド:

```cisco
show spanning-tree mst 1
show spanning-tree mst interface gi0/0
show spanning-tree mst interface gi0/1
```

期待する結果:

- `MSTI 1` で SW3 の root path 選択が変わること
- `spanning-tree mst 1 cost` が interface 単位 / instance 単位の tuning であることが分かること

## 実際の結果

```markdown
TODO: 設定後の `show spanning-tree mst 1`、`show spanning-tree mst interface gi0/0`、`show spanning-tree mst interface gi0/1` の出力を貼る。
```

## 出力の見方

- `show spanning-tree mst 1` では、role / state / cost の変化を見ます
- `show spanning-tree mst interface gi0/0` では、特定 interface に設定した cost がどう見えるかを確認します
- `MSTI 1` にだけ効かせていることも重要です

## 試験で問われそうな形

- `spanning-tree mst <instance-number> cost`
- root bridge だけでなく interface cost でも `MSTI` ごとの経路選択を変えられる
- `show spanning-tree mst interface <interface-id>` で interface 単位の見え方を確認する

## 読み終えたあとに説明できること

- MST の cost tuning をどんな場面で使うか
- priority と cost の役割の違い
- どの `show` 出力を見れば経路変更を確認できるか

## まとめ

この回では、`MSTI` ごとの経路選択を interface cost で調整する考え方を整理しました。  
次回は、同一コスト時の選択を port-priority で調整する方法へ進みます。

## 関連ファイル

- この回で使った MST interface cost 調整用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/07_mst_interface_cost_tuning
