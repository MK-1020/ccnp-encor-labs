# CCNP ENCOR 試験対策 第21回: Cisco MST設定：MST regionを成立させる3条件

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **複数スイッチを同じ `MST region` として扱う前提** を作ります。  
ここではまず、`MST region` を成立させる 3 条件のうち、特に **name / revision / VLAN-to-instance mapping をそろえる必要がある** という考え方に焦点を当てます。

## 今回の確認ポイント

- `MST region` を成立させる3条件は何か
- なぜ複数スイッチで同じ `MST region` をそろえる必要があるのか
- `show spanning-tree mst configuration` のどこを見れば一致条件を確認できるのか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| MST region | 高 | 同じ `MST region` を構成する条件 | `show spanning-tree mst configuration` | 本編で検証 | 4章後半の前提になるため |
| MST Configuration | 高 | region name / revision の設定 | `show spanning-tree mst configuration` | 本編で検証 | 次回の mapping 設定につながるため |

## 前回とのつながり

前回は `MST0` / `IST` の default の見え方を確認しました。  
今回はそこから一歩進み、複数スイッチで同じ `MST region` を共有するための条件を確認します。

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
| 主に見るノード | SW1〜SW4 |
| 主な確認コマンド | `show spanning-tree mst configuration` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、SW1〜SW4 を同じ `MST region` として扱う前提を作ります。後続の `MSTI` tuning や `boundary` 確認は、この前提がそろっていることが出発点です。

![[スクリーンショット 2026-05-26 210406 1.png]]

## 初期状態

全スイッチが MST mode で動いている状態から始めます。  
ここでは、まだ default の `MST configuration` であることを確認し、そのあと共通の region 情報を入れます。

確認コマンド:

```cisco
show spanning-tree mst configuration
```

期待する結果:

- まだ default の `MST configuration` であることが確認できる
- これからそろえるべき項目が `name` / `revision` / `VLAN-to-instance mapping` であると説明できる

## 実際の結果

```markdown
TODO: 設定前の `show spanning-tree mst configuration` の出力を貼る。
```

## 設定

今回は、全スイッチを同じ `MST region` としてそろえる前提を作ります。  
ここではまず、region name を `MST-LAB`、revision を `1` にそろえます。

SW1 / SW2 / SW3 / SW4 で入力:

```cisco
configure terminal
spanning-tree mst configuration
 name MST-LAB
 revision 1
end
write memory
```

注意点:

- `MST region` は `name` / `revision` / `VLAN-to-instance mapping` の一致で成立します
- この回では、まず `name` と `revision` をそろえるところまでを扱います
- `VLAN-to-instance mapping` の詳細は次回へ分けます

## 設定反映・期待結果の確認

ここでは、`show spanning-tree mst configuration` を使って **同じ `MST region` を作る前提がそろったか** を確認します。  
単にコマンドを入れたことではなく、各スイッチが同じ設計情報を持ち始めたかを見るのが目的です。

確認コマンド:

```cisco
show spanning-tree mst configuration
```

期待する結果:

- `Name` が `MST-LAB` になっていること
- `Revision` が `1` になっていること
- `MST region` を成立させる条件のうち、`name` / `revision` をそろえたと説明できること

## 実際の結果

```markdown
TODO: 設定後の `show spanning-tree mst configuration` の出力を貼る。
```

## 出力の見方

- `Name` の一致は、同じ `MST region` 名を使っているかを見る項目です
- `Revision` の一致は、同じ revision を使っているかを見る項目です
- この回ではまだ mapping を入れていないので、region 成立条件のうち 2 つをそろえた段階だと読めれば十分です

## 概念整理

### MST region を成立させる3条件

複数スイッチを同じ `MST region` として扱うには、次の 3 条件が一致している必要があります。

- `name`
- `revision`
- `VLAN-to-instance mapping`

この 3 つがそろわないと、同じ `MSTI` 設計を共有しているとは見なせません。

## 試験で問われそうな形

- `MST region` を成立させる条件は `name` / `revision` / `VLAN-to-instance mapping`
- `show spanning-tree mst configuration` で region 情報を確認する
- `name` と `revision` だけでは不十分で、mapping も一致が必要

## 読み終えたあとに説明できること

- `MST region` の成立条件
- なぜ複数スイッチで同じ region 情報をそろえる必要があるのか
- `show spanning-tree mst configuration` のどこを見るべきか

## まとめ

この回では、`MST region` を成立させる 3 条件の考え方と、`name` / `revision` をそろえるところまで確認しました。  
次回は、VLAN を `MSTI` にどう割り当てるか、つまり `VLAN-to-instance mapping` の設定へ進みます。

## 関連ファイル

- この回で使った MST region 3条件確認用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/04_mst_region_three_conditions
