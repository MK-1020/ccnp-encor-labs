# CCNP ENCOR 試験対策 第23回: Cisco MST設定：MSTIごとにroot bridgeを設計する

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **`MSTI` ごとに root bridge を分ける** ところを扱います。  
同じ `MST region` の中でも、instance ごとに異なる転送方針を持たせられることが MST の設計上の利点です。

## 今回の確認ポイント

- `MSTI 1` と `MSTI 2` で root bridge を分けること
- なぜ root 設計は VLAN 単位ではなく `MSTI` 単位で考えるのか
- `show spanning-tree mst 1` / `mst 2` のどこを見れば、意図した root 設計になったと分かるか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| MST Tuning | 高 | `MSTI` ごとに root を分けること | `show spanning-tree mst 1` / `mst 2` | 本編で検証 | MST 設計の中心になるため |
| MSTI priority / root bridge | 高 | `MSTI` 単位で priority を変えること | `show spanning-tree mst 1` / `mst 2` | 本編で検証 | VLAN 単位との違いを説明するため |

## 前回とのつながり

前回は、VLAN 10 / 20 / 30 を `MSTI` に割り当てました。  
今回はその土台を使って、`MSTI 1` と `MSTI 2` の root bridge を別々に設計します。

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
| 設計対象 | `MSTI 1` / `MSTI 2` |
| 主な確認コマンド | `show spanning-tree mst 1` / `mst 2` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、同一 `MST region` 内の SW1〜SW3 を中心に、`MSTI 1` は SW1、`MSTI 2` は SW2 を root として読む構成を作ります。

## 初期状態

前回までに、VLAN 10 が `MSTI 1`、VLAN 20 / 30 が `MSTI 2` に割り当てられている前提で進めます。  
ここでは、priority を変える前の `MSTI 1` / `MSTI 2` の見え方を確認します。

確認コマンド:

```cisco
show spanning-tree mst 1
show spanning-tree mst 2
```

期待する結果:

- tuning 前の root / role / state が確認できること
- これから `MSTI` 単位で root を変える前提が分かること

## 実際の結果

```markdown
TODO: tuning 前の `show spanning-tree mst 1` と `show spanning-tree mst 2` の出力を貼る。
```

## 設定

ここでは例として、`MSTI 1` は SW1、`MSTI 2` は SW2 が root になるように priority を調整します。

SW1で入力:

```cisco
configure terminal
spanning-tree mst 1 priority 24576
end
write memory
```

SW2で入力:

```cisco
configure terminal
spanning-tree mst 2 priority 24576
end
write memory
```

必要なら、次のように `root primary` を使う書き方でもよいです。

```cisco
spanning-tree mst 1 root primary
spanning-tree mst 2 root primary
```

## 設定反映・期待結果の確認

ここでは、`show spanning-tree mst 1` / `mst 2` を使って **instance ごとに別の root 設計ができたか** を確認します。

確認コマンド:

```cisco
show spanning-tree mst 1
show spanning-tree mst 2
```

期待する結果:

- `MSTI 1` は SW1 を root として読めること
- `MSTI 2` は SW2 を root として読めること
- root bridge 設計は VLAN 単位ではなく `MSTI` 単位で行うことが分かること

## 実際の結果

```markdown
TODO: 設定後の `show spanning-tree mst 1` と `show spanning-tree mst 2` の出力を貼る。
```

## 出力の見方

- `show spanning-tree mst 1` では VLAN 10 を載せた instance の root / role / state を見ます
- `show spanning-tree mst 2` では VLAN 20 / 30 を載せた instance の root / role / state を見ます
- ここでの主語は VLAN そのものではなく `MSTI` です

## 試験で問われそうな形

- MST の root 設計は VLAN 単位ではなく `MSTI` 単位
- `spanning-tree mst <instance-number> priority ...`
- `show spanning-tree mst <instance-number>` で instance ごとの root を確認する

## 読み終えたあとに説明できること

- `MSTI` ごとに root bridge を分ける意味
- どの設定で `MSTI` ごとの root を調整するか
- どの `show` を見れば設計意図を確認できるか

## まとめ

この回では、`MSTI 1` と `MSTI 2` の root bridge を分ける設計を確認しました。  
次回は、priority ではなく interface cost を使って経路選択を調整する方法へ進みます。

## 関連ファイル

- この回で使った MSTI ごとの root bridge 設計用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/06_msti_root_bridge_design
