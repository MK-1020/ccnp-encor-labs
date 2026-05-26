# CCNP ENCOR 試験対策 第27回: Cisco MSTとtrunk pruning：VLAN allowedとMSTI topologyのズレを確認する

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **trunk allowed VLAN の設定と `MSTI` topology の見え方が必ずしも同じ粒度ではない** ことを確認します。  
MST は instance 単位で topology を計算しますが、trunk pruning は VLAN 単位で設定するため、そのズレを理解する回です。

## 今回の確認ポイント

- trunk allowed VLAN は VLAN 単位で設定すること
- それでも MST の topology は `MSTI` 単位で計算されること
- 同じ `MSTI` に属する VLAN をバラバラに prune すると何が困るか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| Trunk Link Pruning | 高 | VLAN allowed と `MSTI` topology のズレ | `show interfaces trunk` / `show spanning-tree mst 2` | 本編で検証 | OCG の設計上の注意点だから |
| MST Tuning | 中 | `MSTI` 単位で topology を読むこと | `show spanning-tree mst interface <interface-id>` | 本編で検証 | pruning の見方に直結するため |

## 前回とのつながり

前回は、VLAN 40 を `IST` / `MST0` に残す考え方を見ました。  
今回は、VLAN 20 / 30 が同じ `MSTI 2` に属している前提を使い、trunk pruning のズレを確認します。

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
| 主に見る VLAN | 20, 30 |
| 主な確認コマンド | `show interfaces trunk` / `show spanning-tree mst 2` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、VLAN 20 / 30 が同じ `MSTI 2` に属している前提で、ある trunk から VLAN 30 だけを外す例を使います。これは推奨設定ではなく、**ズレを観察するための例** です。

## 初期状態

前回までに、VLAN 20 / 30 が `MSTI 2` に割り当てられている前提で進めます。  
まずは pruning 前の trunk 状態と `MSTI 2` の見え方を確認します。

確認コマンド:

```cisco
show interfaces trunk
show spanning-tree mst 2
show spanning-tree mst interface gi0/0
```

期待する結果:

- trunk 上で VLAN 20 / 30 が通っていることを確認できること
- `MSTI 2` としての topology を読めること

## 実際の結果

```markdown
TODO: pruning 前の `show interfaces trunk`、`show spanning-tree mst 2`、`show spanning-tree mst interface gi0/0` の出力を貼る。
```

## 設定

ここでは例として、SW1 の trunk から VLAN 30 だけを外します。  
これは推奨設定ではなく、**同じ `MSTI` に属する VLAN をバラバラに prune するとどう考えるべきか** を確認するための例です。

SW1で入力:

```cisco
configure terminal
interface gi0/0
 switchport trunk allowed vlan 1,10,20
end
write memory
```

## 設定反映・期待結果の確認

ここでは、`show interfaces trunk` で VLAN 単位の allowed 状態を見つつ、`show spanning-tree mst 2` で `MSTI 2` の topology を読みます。  
この 2 つの粒度が同じではないことがポイントです。

確認コマンド:

```cisco
show interfaces trunk
show spanning-tree mst 2
show spanning-tree mst interface gi0/0
```

期待する結果:

- trunk allowed VLAN の設定は VLAN 単位で反映されること
- それでも MST の topology は `MSTI` 単位で計算されること
- 同じ `MSTI` に属する VLAN をバラバラに prune すると、STP 上の topology と実際の VLAN 到達性がズレる可能性があること

## 実際の結果

```markdown
TODO: pruning 後の `show interfaces trunk`、`show spanning-tree mst 2`、`show spanning-tree mst interface gi0/0` の出力を貼る。
```

## 出力の見方

- `show interfaces trunk` では VLAN 単位の allowed 状態を見ます
- `show spanning-tree mst 2` では VLAN 20 / 30 を載せた `MSTI 2` の topology を見ます
- 粒度が違うので、同じ `MSTI` に属する VLAN は pruning でもまとめて考える必要があると読めればよいです

## 概念整理

### なぜズレが起きるのか

trunk allowed VLAN の設定は VLAN 単位で行います。  
ただし、MST の topology は `MSTI` 単位で計算されるため、**同じ `MSTI` に属する VLAN を trunk 上でバラバラに許可・除外すると、STP 上の topology と実際の VLAN 到達性がズレる可能性があります。**

## 試験で問われそうな形

- trunk allowed VLAN は VLAN 単位
- MST topology は `MSTI` 単位
- 同じ `MSTI` に属する VLAN は、pruning でもまとめて考える

## 読み終えたあとに説明できること

- trunk pruning と `MSTI` topology の粒度の違い
- なぜ同じ `MSTI` に属する VLAN をバラバラに prune すると困るのか
- どの `show` で VLAN 単位 / instance 単位の見え方を確認するか

## まとめ

この回では、trunk pruning を VLAN 単位で設定しつつ、MST は `MSTI` 単位で topology を計算することを確認しました。  
次回は、revision 不一致による `MST region boundary` を確認します。

## 関連ファイル

- この回で使った MST trunk pruning 確認用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/10_mst_trunk_pruning
