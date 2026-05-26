# CCNP ENCOR 試験対策 第22回: Cisco MST設定：VLAN-to-instance mappingを確認する

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **VLAN を `MSTI` にどう割り当てるか** を扱います。  
前回そろえた `MST region` の土台の上で、どの VLAN をどの instance で扱うかを決めます。

## 今回の確認ポイント

- VLAN 10 を `MSTI 1` に入れること
- VLAN 20 / 30 を `MSTI 2` に入れること
- `show spanning-tree mst configuration` と `show spanning-tree mst <instance-number>` で、mapping をどう確認するか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| MST Configuration | 高 | VLAN-to-instance mapping を設定すること | `show spanning-tree mst configuration` | 本編で検証 | instance 設計の中心になるため |
| MSTI assignment | 高 | VLAN 10 / 20 / 30 を `MSTI` に割り当てること | `show spanning-tree mst 1` / `mst 2` | 本編で検証 | 以後の tuning 前提になるため |
| MST Verification | 高 | verification を VLAN 単位ではなく `MSTI` 単位で読むこと | `show spanning-tree mst 1` / `mst 2` | 本編で検証 | 4章の主語を切り替えるため |

## 前回とのつながり

前回は、同じ `MST region` を構成する条件として `name` / `revision` / `VLAN-to-instance mapping` を確認しました。  
今回はそのうち、実際に VLAN を `MSTI` に割り当てる部分を設定します。

## 検証ステータス

| 項目 | 状態 |
|---|---|
| EVE-NG構築 | TODO |
| 設定投入 | TODO |
| show確認 | TODO |
| 疎通確認 | 対象外 |
| Wireshark確認 | 任意 |
| 結果反映 | TODO |

## 検証環境

| 項目 | 内容 |
|---|---|
| 使用イメージ | IOSvL2 |
| 使用ノード | SW1〜SW4 |
| 対象VLAN | 10, 20, 30 |
| 主な確認コマンド | `show spanning-tree mst configuration` / `mst 1` / `mst 2` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、SW1〜SW4 を同じ `MST region` のまま、VLAN 10 / 20 / 30 をどの `MSTI` に載せるかを確認します。

Wireshark で MST BPDU を補助確認する場合は、同一 region 内の trunk link である SW1-SW2 または SW2-SW3 を観測点にします。

![[スクリーンショット 2026-05-26 210406 1.png]]

## 初期状態

前回までに、SW1〜SW4 で `name MST-LAB`、`revision 1` まではそろっている前提で進めます。  
ここでは、まだ VLAN-to-instance mapping を入れていない状態を確認してから設定します。

確認コマンド:

```cisco
show spanning-tree mst configuration
```

期待する結果:

- `name` / `revision` はそろっていること
- まだ VLAN 10 / 20 / 30 の mapping は入っていないこと

## 実際の結果

```markdown
TODO: mapping 設定前の `show spanning-tree mst configuration` の出力を貼る。
```

## 設定

今回は、VLAN 10 を `MSTI 1`、VLAN 20 / 30 を `MSTI 2` に割り当てます。  
これは、関連する VLAN を instance 単位でまとめて扱うための設定です。

SW1 / SW2 / SW3 / SW4 で入力:

```cisco
configure terminal
spanning-tree mst configuration
 instance 1 vlan 10
 instance 2 vlan 20,30
end
write memory
```

## 設定反映・期待結果の確認

ここでは、`show` を使って **VLAN ごとではなく `MSTI` ごとに topology を読む前提が整ったか** を確認します。

確認コマンド:

```cisco
show spanning-tree mst configuration
show spanning-tree mst 1
show spanning-tree mst 2
```

期待する結果:

- VLAN 10 が `MSTI 1` に入っていること
- VLAN 20 / 30 が `MSTI 2` に入っていること
- verification では VLAN 単位ではなく `MSTI` を主語にして読む必要があること
- 同じ region の全スイッチで mapping を一致させる必要があること

## 実際の結果

```markdown
TODO: 設定後の `show spanning-tree mst configuration`、`show spanning-tree mst 1`、`show spanning-tree mst 2` の出力を貼る。
```

## 出力の見方

- `show spanning-tree mst configuration` では、instance ごとにどの VLAN が割り当てられているかを確認します
- `show spanning-tree mst 1` と `mst 2` では、どの VLAN を載せた instance を見ているのかを意識して読みます
- ここでの主語は VLAN ではなく `MSTI` です

## 概念整理

### VLAN をまとめる単位は MSTI

MST では、VLAN ごとに別の spanning tree を動かすのではなく、関連する VLAN を `MSTI` にまとめます。  
そのため、以後の tuning や verification も VLAN 単位ではなく instance 単位で考えます。

## 試験で問われそうな形

- VLAN-to-instance mapping は同じ `MST region` の全スイッチで一致が必要
- VLAN 10 を `MSTI 1`、VLAN 20 / 30 を `MSTI 2` に載せる例
- `show spanning-tree mst configuration` と `show spanning-tree mst <instance-number>` の使い分け

## 読み終えたあとに説明できること

- VLAN-to-instance mapping をなぜ入れるのか
- VLAN と `MSTI` の対応をどこで確認するのか
- verification で `MSTI` を主語にして読む理由

## まとめ

この回では、VLAN 10 / 20 / 30 を `MSTI` へ割り当てるところまで確認しました。  
次回は、`MSTI` ごとに root bridge をどう設計するかへ進みます。

## 関連ファイル

- この回で使った VLAN-to-instance mapping 確認用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/05_mst_vlan_to_instance_mapping
