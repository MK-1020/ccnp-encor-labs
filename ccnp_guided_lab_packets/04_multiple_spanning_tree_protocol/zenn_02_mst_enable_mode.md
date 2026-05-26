# CCNP ENCOR 試験対策 第19回: Cisco MST設定：spanning-tree mode mstでMSTを有効化する

## この回の位置づけ

Chapter 4 の共通設計シナリオのうち、この回では **STP mode を MST に切り替える** ところだけを扱います。  
まだ `MST region` や `MSTI` mapping には進まず、まずは VLAN 単位の見え方から MST の世界へ入る入口を作ります。

## 今回の確認ポイント

- `spanning-tree mode mst` を入れること
- なぜ最初に mode 切替だけを切り出して確認するのか
- `show spanning-tree summary` のどこを見れば、MST へ切り替わったと分かるか

## 公式テキスト回収表

| OCG論点 | 重要度 | この記事で確認すること | 確認方法 | 扱い方 | 理由 |
|---|---|---|---|---|---|
| Multiple Spanning Tree Protocol | 高 | mode を MST に切り替えること | `show spanning-tree summary` | 本編で検証 | 4章の入口になるため |
| MST Configuration | 中 | `spanning-tree mode mst` の投入 | `show spanning-tree summary` | 本編で検証 | 後続記事の前提になるため |

## 前回とのつながり

前回は Chapter 4 全体の共通設計シナリオを整理しました。  
今回はその最初の小さな設計判断として、スイッチを MST mode へ切り替えます。

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
| 主に見るノード | SW1 |
| 主な確認コマンド | `show spanning-tree summary` |

## トポロジ

Chapter 4 の共通トポロジを使います。  
この回では、代表として SW1 の見え方を確認します。ほかのスイッチでも同様に確認できますが、この段階では主な違いは port role の見え方くらいです。

![[スクリーンショット 2026-05-26 210406.png]]

## 初期状態

まずは、MST を有効化する前の状態を確認します。  
ここでは「いまどの STP mode で動いているか」「VLAN ごとに spanning tree 情報が見えているか」を設定前の基準状態として押さえます。  
各スイッチには事前に SW1〜SW4 の hostname を設定しておきます。

確認コマンド:

```cisco
show vlan brief
show spanning-tree summary
show spanning-tree
```

期待する結果:

- まだ初期状態に近い構成であることが確認できる
- 現在の STP mode が確認できる
- `show spanning-tree` で VLAN ごとに STP 情報が見えていることが確認できる

## 実際の結果

ここでは代表として SW1 の出力を載せます。  
他のスイッチでも同様に確認できますが、この段階では主な違いは `show spanning-tree` の port role の見え方くらいです。

SW1 の結果:

```text
SW1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/1, Gi0/2, Gi0/3
                                                Gi1/0, Gi1/1, Gi1/2, Gi1/3
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
SW1#
SW1#show spanning-tree summary
Switch is in pvst mode
Root bridge for: VLAN0001
Extended system ID                      is enabled
Portfast Default                        is disabled
Portfast Edge BPDU Guard Default        is disabled
Portfast Edge BPDU Filter Default       is disabled
Loopguard Default                       is disabled
PVST Simulation Default                 is enabled but inactive in pvst mode
Bridge Assurance                        is enabled but inactive in pvst mode
EtherChannel misconfig guard            is enabled
Configured Pathcost method used is short
UplinkFast                              is disabled
BackboneFast                            is disabled

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     0         0        0          8          8
---------------------- -------- --------- -------- ---------- ----------
1 vlan                       0         0        0          8          8
SW1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5000.0001.0000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0001.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p
Gi0/1               Desg FWD 4         128.2    P2p
Gi0/2               Desg FWD 4         128.3    P2p
Gi0/3               Desg FWD 4         128.4    P2p
Gi1/0               Desg FWD 4         128.5    P2p
Gi1/1               Desg FWD 4         128.6    P2p
Gi1/2               Desg FWD 4         128.7    P2p
Gi1/3               Desg FWD 4         128.8    P2p
```

この時点ではまだ trunk 設定を入れていないので、`show interfaces trunk` が空でも問題ありません。

## 設定

MST の後続記事では、`MST region` や `MSTI` を扱います。  
その前提として、まず全スイッチを MST mode に切り替えます。

全てのスイッチで入力:

```cisco
configure terminal
spanning-tree mode mst
vlan 10
 name USERS_10
vlan 20
 name USERS_20
vlan 30
 name USERS_30
end
write memory
```

## 設定反映・期待結果の確認

ここでは、`show spanning-tree summary` を使って **mode が PVST から MST へ切り替わったか** を確認します。  
この回では `show spanning-tree mst` の詳細な読み方までは進まず、まず `Switch is in mst mode` を確認することに絞ります。

確認コマンド:

```cisco
show spanning-tree summary
```

期待する結果:

- `Switch is in mst mode` と読めること
- まずは STP mode が MST に切り替わったことを確認できること

## 実際の結果

SW1 の結果:

```text
SW1#show spanning-tree summary
Switch is in mst mode (IEEE Standard)
Root bridge for: MST0
Extended system ID                      is enabled
Portfast Default                        is disabled
Portfast Edge BPDU Guard Default        is disabled
Portfast Edge BPDU Filter Default       is disabled
Loopguard Default                       is disabled
PVST Simulation                         is enabled
Bridge Assurance                        is enabled
EtherChannel misconfig guard            is enabled
Configured Pathcost method used is short (Operational value is long)
UplinkFast                              is disabled
BackboneFast                            is disabled

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
MST0                         0         0        0          8          8
---------------------- -------- --------- -------- ---------- ----------
1 mst                        0         0        0          8          8
```

## 出力の見方

- `Switch is in mst mode` から、STP mode が MST へ切り替わったことが分かります
- `Root bridge for: MST0` と見えることで、PVST の VLAN 単位表示とは違う見え方へ移ったことも分かります

## 試験で問われそうな形

- MST を有効化するコマンドは `spanning-tree mode mst`
- mode の切替確認は `show spanning-tree summary`
- MST では `MST0` が見えるところから読み始める

## 読み終えたあとに説明できること

- なぜ最初に `spanning-tree mode mst` だけを切り出して確認するのか
- `show spanning-tree summary` のどこを見れば mode 切替が分かるのか
- PVST 的な見え方から MST の見え方へ入る入口

## まとめ

この回では、`spanning-tree mode mst` でスイッチを MST mode へ切り替えました。  
次回は、切り替え直後の `MST0` / `IST` を `show spanning-tree mst` でどう読むかを確認します。

## 関連ファイル

- この回で使った MST mode 有効化確認用コンフィグ一式
- https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/04_multiple_spanning_tree_protocol/configs/02_mst_enable_mode
