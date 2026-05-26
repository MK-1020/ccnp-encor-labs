# CCNP ENCOR 試験対策 第25回: Cisco MST設定：spanning-tree mst port-priorityで同一コスト時の選択を調整する

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **同一コストのリンクがあるときに、port-priority で選択を調整する** ところを扱います。  
前回の cost tuning と違い、ここでは同じ cost 条件の中で、どのポートを優先させるかを決めます。

## 今回の確認ポイント

- `spanning-tree mst <instance-number> port-priority` をどこで使うか
- 同一コスト時の比較要素として port-priority をどう考えるか
- `show spanning-tree mst interface` のどこで設定反映を確認するか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| MST Tuning | 高 | port-priority を使った調整 | `show spanning-tree mst 1` / `mst interface ...` | 本編で検証 | tuning 論点として回収するため |
| MST port priority | 高 | 同一条件時の比較要素として使うこと | `show spanning-tree mst interface <interface-id>` | 本編で検証 | OCG の tuning 論点を分離して拾うため |

## 前回とのつながり

前回は interface cost で経路選択を変えました。  
今回は、cost をそろえたまま、どのポートを選ばせるかを port-priority で調整します。

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
| 例として調整するノード | SW4 |
| 主な確認コマンド | `show spanning-tree mst 1` / `show spanning-tree mst interface gi0/0` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、SW4 から上流へ向かう 2 本の uplink を例に、同一コスト条件でどちらを優先させるかを `MSTI 1` 単位で調整します。

## 初期状態

前回までに基本の `MST region` と `MSTI` 設計が入っている前提で進めます。  
ここでは、port-priority 調整前の role / state と interface detail を確認します。

確認コマンド:

```cisco
show spanning-tree mst 1
show spanning-tree mst interface gi0/0
show spanning-tree mst interface gi0/1
```

期待する結果:

- 調整前の uplink の見え方が確認できること
- これから同一コスト時の比較要素を変える前提が分かること

## 実際の結果

```markdown
TODO: 調整前の `show spanning-tree mst 1`、`show spanning-tree mst interface gi0/0`、`show spanning-tree mst interface gi0/1` の出力を貼る。
```

## 設定

ここでは例として、SW4 の片側 interface に対して `MSTI 1` の port-priority を下げ、選ばれやすくします。

SW4で入力:

```cisco
configure terminal
interface gi0/0
 spanning-tree mst 1 port-priority 64
end
write memory
```

## 設定反映・期待結果の確認

ここでは、`show spanning-tree mst 1` と `show spanning-tree mst interface` を使って **同一コスト時の選択に port-priority が効くこと** を確認します。

確認コマンド:

```cisco
show spanning-tree mst 1
show spanning-tree mst interface gi0/0
show spanning-tree mst interface gi0/1
```

期待する結果:

- 同一コスト条件の比較要素として port-priority が使われること
- `MSTI 1` に対する interface 単位の tuning として見えること

## 実際の結果

```markdown
TODO: 設定後の `show spanning-tree mst 1`、`show spanning-tree mst interface gi0/0`、`show spanning-tree mst interface gi0/1` の出力を貼る。
```

## 出力の見方

- `show spanning-tree mst interface ...` では、対象 interface の port priority を確認します
- `show spanning-tree mst 1` では、role / state の見え方がどう変わるかを確認します
- cost tuning と違い、ここでは同一条件時の比較要素を変えていると読むのがポイントです

## 試験で問われそうな形

- `spanning-tree mst <instance-number> port-priority`
- port-priority は同一コスト時の選択を調整する
- MST tuning には priority / cost / port-priority がある

## 読み終えたあとに説明できること

- MST の port-priority tuning をどんな場面で使うか
- cost tuning と port-priority tuning の違い
- どの `show` 出力を見れば設定反映を確認できるか

## まとめ

この回では、同一コスト時の選択を `spanning-tree mst port-priority` で調整する考え方を整理しました。  
次回は、VLAN を `IST` / `MST0` に残すとどう見えるかを確認します。

## 関連ファイル

- この回で使った MST port priority 調整用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/08_mst_port_priority_tuning
