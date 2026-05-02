# CCNP ENCOR 試験対策: VLANとMAC Address TableでL2 Forwardingを理解する

# EVE-NGで確認するL2 Forwarding：MACアドレス学習と802.1Qタグ

## はじめに
- CCNP ENCOR official Cert Guideの内容に従って、ラボを構成して学習しています。
- L2に注目し、スイッチがフレームをどのように転送するのかをラボで検証、確認していきます。

## 今回確認したいこと
- スイッチが送信元MACアドレスをもとにMACアドレステーブルを学習すること
- 同じVLAN内では通信できるが、異なるVLANとは通信できないこと
- access portではVLANタグを扱わないこと
- trunk portでは802.1Qタグが付くこと

## 検証トポロジ
![](https://static.zenn.studio/user-upload/95a31c33a09f-20260425.png)

## IPアドレス・VLAN設計
| Device | VLAN | IP Address |
|---|---:|---|
| PC1 | 10 | 192.168.10.11/24 |
| PC3 | 10 | 192.168.10.13/24 |
| PC2 | 20 | 192.168.20.12/24 |
| PC4 | 20 | 192.168.20.14/24 |

## スイッチの設定
- SW1ではPC1側をVLAN 10、PC2側をVLAN 20に割り当て、SW2向けのリンクをtrunkとして設定した。

## 検証1: VLANとaccess portの確認
- まず、access portが意図したVLANに所属しているかを確認
```cisco
SW1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/3, Gi1/0, Gi1/1, Gi1/2
                                                Gi1/3
10   USERS                            active    Gi0/0   ←VLAN10ですね
20   SERVERS                          active    Gi0/1  ←こっちはVLAN20です
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```
```cisco
SW2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/3, Gi1/0, Gi1/1, Gi1/2
                                                Gi1/3
10   USERS                            active    Gi0/0　　←SW1と同じく
20   SERVERS                          active    Gi0/1　 ←意図通りです
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

```
## 検証2: trunk portの確認
- SW1-SW2間のリンクがtrunkとして動作しており、VLAN 10とVLAN 20が許可されていることを確認。
```cisco
SW1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/2       on               802.1q         trunking      1
↑Gi0/2がtrunkポートであるということ
Port        Vlans allowed on trunk
Gi0/2       10,20

Port        Vlans allowed and active in management domain
Gi0/2       10,20

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/2       10,20
```
```cisco
SW2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/2       on               802.1q         trunking      1
↑SW1と同じくtrunkポート
Port        Vlans allowed on trunk
Gi0/2       10,20

Port        Vlans allowed and active in management domain
Gi0/2       10,20

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/2       10,20
```
## 検証3: MACアドレステーブルの学習を確認
- 以下のコマンドで確認します。
```cisco
SW1#clear mac address-table dynamic　　←MACアドレステーブルを消去
SW1#show mac address-table dynamic　　←消えているか確認
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
```
- ↑消えていますね。↓そしてPC1からPC3へpingします。
```pc1
PC1> ping 192.168.10.13

84 bytes from 192.168.10.13 icmp_seq=1 ttl=64 time=8.922 ms
84 bytes from 192.168.10.13 icmp_seq=2 ttl=64 time=8.129 ms
84 bytes from 192.168.10.13 icmp_seq=3 ttl=64 time=10.402 ms
84 bytes from 192.168.10.13 icmp_seq=4 ttl=64 time=12.815 ms
84 bytes from 192.168.10.13 icmp_seq=5 ttl=64 time=11.575 ms
```
- ↓そしてもう一度SW1のMACアドレステーブルを見ます。
```cisco
SW1#show mac address-table dynamic
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
  10    0050.7966.6803    DYNAMIC     Gi0/0　　←PC1のMACアドレスが登録されている
  10    0050.7966.6805    DYNAMIC     Gi0/2　　←同じようにPC3のMACアドレスが登録されている
Total Mac Addresses for this criterion: 2
```
- PC1からPC3へ通信が行われたことによって、SW1でそれぞれのポートに接続されているノードのMACアドレスを学習、登録されたことが確認できた。
## パケットキャプチャしてみた
### ↓access port側(Gi0/0)には802.1Qタグが付いていない
![](https://static.zenn.studio/user-upload/69d19edeb71a-20260425.png)

### ↓trunk port側(Gi0/2)には802.1Qタグが付いてる（赤枠）
![](https://static.zenn.studio/user-upload/129150e00ca5-20260425.png)
- 最初は間違ってaccess port側のキャプチャを見ていて、なんでタグが付かないのだろう～？と思っていたが、ちゃんとtrunk port側のキャプチャについてた、、つまり・・・
- access portではVLANの識別はスイッチ内で行うので、タグを付けたりしない
- trunk portでは複数のVLANを識別する必要があるので、それぞれに応じたVLAN IDのタグを付加する

## 検証5: VLANが違うとARP、pingが届かないことを確認
- 次はPC2のIPアドレスを一時的に192.168.10.12/24へ変更し、PC1と同じIPネットワークに見える状態にするが、PC2の接続ポートはVLAN 20のままとする。リンク異常では無い事を確認するために、IPアドレスを変更する前に、PC4へpingで疎通確認しておく。
```pc2
PC2> ping 192.168.20.14

84 bytes from 192.168.20.14 icmp_seq=1 ttl=64 time=8.812 ms
84 bytes from 192.168.20.14 icmp_seq=2 ttl=64 time=14.026 ms
84 bytes from 192.168.20.14 icmp_seq=3 ttl=64 time=8.643 ms
84 bytes from 192.168.20.14 icmp_seq=4 ttl=64 time=8.974 ms
84 bytes from 192.168.20.14 icmp_seq=5 ttl=64 time=15.829 ms

PC2> set pcname PC2
PC2> ip 192.168.10.12/24
PC2> save
Checking for duplicate address...
PC2 : 192.168.10.12 255.255.255.0

PC2> ping 192.168.10.11

host (192.168.10.11) not reachable
```
![](https://static.zenn.studio/user-upload/a078cd8f5de6-20260425.png)
- PC2でPC1と同じセグメントのIPアドレスに変更しても、VLANが異なるため疎通ができないことを確認できた

## まとめ

今回の検証で、以下を確認できた。

- スイッチは送信元MACアドレスをもとにMACアドレステーブルを学習する
- MACアドレステーブルはVLANごとに管理される
- access portでは端末にタグなしフレームを渡す
- trunk portでは802.1QタグによってVLANを識別する
- VLANが異なるとARP broadcastは届かない
- 同じIPネットワークに見えても、L2が分離されていれば通信できない

## 関連ファイル

今回の設定ファイルはGitHubに置いています。

https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/01_packet_forwarding/configs/l2_forwarding
