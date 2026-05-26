# CCNP ENCOR 試験対策 第20回: Cisco MSTの見方：MST0 / ISTをshow spanning-tree mstで確認する

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **MST mode にした直後の `MST0` / `IST` の見え方** を確認します。  
まだ `MST region` の詳細設定は入れず、default の MST configuration をどう読むかに絞ります。

## 今回の確認ポイント

- `show spanning-tree mst` で `MST0` がどう見えるか
- `show spanning-tree mst configuration` の `Name []`、`Revision 0`、`Instances configured 1` の意味
- 明示的に `MSTI` mapping していない VLAN が `MST0` / `IST` に属すること

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| IST | 高 | `MST0` / `IST` の見え方 | `show spanning-tree mst` | 本編で検証 | 後続記事の土台になるため |
| MSTI | 高 | まだ mapping していない状態では `MST0` を見ること | `show spanning-tree mst configuration` | 本編で検証 | instance の考え方の入口になるため |
| MST Verification | 高 | `show spanning-tree mst` と `show spanning-tree mst configuration` の読み方 | `show spanning-tree mst` | 本編で検証 | 4章の基本確認手順になるため |

## 前回とのつながり

前回は `spanning-tree mode mst` による mode 切替だけを確認しました。  
今回はその続きとして、MST へ切り替えた直後に何が見えるのかを確認します。

## 検証ステータス

| 項目 | 状態 |
|---|---|
| EVE-NG構築 | TODO |
| 設定投入 | 完了 |
| show確認 | TODO |
| 疎通確認 | 対象外 |
| Wireshark確認 | 対象外 |
| 結果反映 | TODO |

## 検証環境

| 項目 | 内容 |
|---|---|
| 使用イメージ | IOSvL2 |
| 使用ノード | SW1〜SW4 |
| 主に見るノード | SW1 |
| 主な確認コマンド | `show spanning-tree mst` / `show spanning-tree mst configuration` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、`MST region` をまだ明示設定していない状態で、代表として SW1 の `MST0` / `IST` の見え方を確認します。

![[スクリーンショット 2026-05-26 210406.png]]

## 初期状態

前回までに、全スイッチで `spanning-tree mode mst` と VLAN 10 / 20 / 30 の作成までは入っている前提で進めます。  
ここでは、まだ name / revision / VLAN-to-instance mapping を明示設定していない状態を確認します。

確認コマンド:

```cisco
show spanning-tree mst
show spanning-tree mst configuration
```

期待する結果:

- `MST0` が見えること
- `show spanning-tree mst configuration` に default の設定が見えること
- 明示的に別の `MSTI` へ割り当てていない VLAN は、default では `MST0` / `IST` に属すること

## 実際の結果

SW1 の結果:

```text
SW1#show spanning-tree mst

##### MST0    vlans mapped:   1-4094
Bridge        address 5000.0001.0000  priority      32768 (32768 sysid 0)
Root          this switch for the CIST
Operational   hello time 2 , forward delay 15, max age 20, txholdcount 6
Configured    hello time 2 , forward delay 15, max age 20, max hops    20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Gi0/0            Desg FWD 20000     128.1    P2p
Gi0/1            Desg FWD 20000     128.2    P2p
Gi0/2            Desg FWD 20000     128.3    P2p
Gi0/3            Desg FWD 20000     128.4    P2p
Gi1/0            Desg FWD 20000     128.5    P2p
Gi1/1            Desg FWD 20000     128.6    P2p
Gi1/2            Desg FWD 20000     128.7    P2p
Gi1/3            Desg FWD 20000     128.8    P2p

SW1#
SW1#show spanning-tree mst configuration
Name      []
Revision  0     Instances configured 1

Instance  Vlans mapped
--------  ---------------------------------------------------------------------
0         1-4094
-------------------------------------------------------------------------------
SW1#
```

## 出力の見方

- `##### MST0    vlans mapped:   1-4094` から、まだ全 VLAN が `MST0` 側に見えていることが分かります
- `Name []`、`Revision 0` は、まだ `MST region` の詳細設定を明示していない default 状態です
- `Instances configured 1` は、この段階では追加の `MSTI` をまだ作っていない、という読み方になります

## 概念整理

### IST と MST0

`IST` は `MST0` のことです。  
同一 `MST region` 内で必ず存在する基本 instance であり、「未使用」ではなく、MST の土台として意味を持ちます。

### `vlans mapped: 1-4094` の意味

この段階では、まだ `instance 1 vlan ...` のような mapping を入れていないため、明示的に別の `MSTI` へ割り当てていない VLAN は default で `MST0` / `IST` に属します。

## 試験で問われそうな形

- `MST0` は `IST`
- `show spanning-tree mst configuration` の default 状態では `Name []`、`Revision 0`、`Instance 0 = 1-4094`
- 明示的に mapping していない VLAN は `MST0` / `IST`

## 読み終えたあとに説明できること

- `MST0` / `IST` をどこで確認するか
- `show spanning-tree mst` と `show spanning-tree mst configuration` の役割の違い
- default の MST configuration をどう読むか

## まとめ

この回では、MST mode に切り替えた直後の `MST0` / `IST` の見え方を確認しました。  
次回は、複数スイッチを同じ `MST region` として扱うための3条件を確認します。

## 関連ファイル

- この回で使った MST0 / IST 確認用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/03_mst0_ist_show
