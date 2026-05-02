# CCNP ENCOR 試験対策: root bridgeとポート役割でSTPの基本動作を理解する

# EVE-NGで確認するSTP Fundamentals：root bridge・root port・designated port・blocking port

## はじめに

- CCNP ENCOR official Cert Guide の内容に従って、EVE-NGでラボを組んで学習しています。
- 今回は STP の最初の確認として、冗長リンクがあるL2ネットワークで、どうやってループを防ぐのかを見ていきます。

L2では冗長リンクを張れると安心ですが、そのままではブロードキャストや unknown unicast flooding がループし続ける危険があります。

STP は、その冗長性を残したまま、**一時的にどこかのポートを止めてループを防ぐ** ための仕組みです。

## 今回確認したいこと

- root bridge が1台選ばれること
- non-root switch では root port が1本選ばれること
- designated port と blocking port が決まること
- 冗長リンクがあってもL2ループにならないこと

## 検証トポロジ

```markdown
TODO: ここにEVE-NGのトポロジスクリーンショットを貼る
```

## 使用する構成

```text
        SW1
       /   \
     SW2---SW3
      |      |
     PC1    PC2
```

## IPアドレス設計

| Device | IP Address |
|---|---|
| PC1 | 192.168.10.11/24 |
| PC2 | 192.168.10.12/24 |

## 入力する設定

今回はSTPの基本動作を見るのが目的なので、VLAN 1 のまま、3台のスイッチを三角形につないで確認します。

各スイッチでは、少なくともリンクを `no shutdown` にして、STP モードが `rapid-pvst` で動いていることを確認しておきます。

SW1で入力:

```cisco
enable
configure terminal
hostname SW1
spanning-tree mode rapid-pvst
interface range gi0/0 - 2
 no shutdown
end
write memory
```

SW2で入力:

```cisco
enable
configure terminal
hostname SW2
spanning-tree mode rapid-pvst
interface range gi0/0 - 2
 no shutdown
end
write memory
```

SW3で入力:

```cisco
enable
configure terminal
hostname SW3
spanning-tree mode rapid-pvst
interface range gi0/0 - 2
 no shutdown
end
write memory
```

PC1で実行:

```text
PC1> set pcname PC1
PC1> ip 192.168.10.11/24
PC1> save
```

PC2で実行:

```text
PC2> set pcname PC2
PC2> ip 192.168.10.12/24
PC2> save
```

## 検証1: root bridge を確認する

SW1で入力:

```cisco
show spanning-tree vlan 1
```

SW2で入力:

```cisco
show spanning-tree vlan 1
```

SW3で入力:

```cisco
show spanning-tree vlan 1
```

SW1の実行結果:

```markdown
SW1#show spanning-tree vlan 1

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             This bridge is the root　■このスイッチがルートですよ
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

ここでは `This bridge is the root` と、`Role` 列がすべて `Desg` になっている点を見ます。つまり、**SW1 が root bridge で、各ポートが designated port として forwarding している** と分かります。

SW2の実行結果:

```markdown
SW2#show spanning-tree vlan 1

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0002.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p
Gi0/1               Desg FWD 4         128.2    P2p
Gi0/2               Desg FWD 4         128.3    P2p
Gi0/3               Desg FWD 4         128.4    P2p
Gi1/0               Desg FWD 4         128.5    P2p
Gi1/1               Desg FWD 4         128.6    P2p
Gi1/2               Desg FWD 4         128.7    P2p
Gi1/3               Desg FWD 4         128.8    P2p
```

※ SW2 の出力では、`Root` が root bridge 方向、`Desg` が下流向きの転送側と考えると読みやすいです。

ここでは `Root ID` が SW1 を指し、`Port 1 (GigabitEthernet0/0)` と出ているので、**SW2 は Gi0/0 を root bridge 方向の root port として使っている** と分かります。

SW3の実行結果:

```markdown
SW3#show spanning-tree vlan 1

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     5000.0001.0000
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5000.0003.0000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p
Gi0/1               Altn BLK 4         128.2    P2p
Gi0/2               Desg FWD 4         128.3    P2p
Gi0/3               Desg FWD 4         128.4    P2p
Gi1/0               Desg FWD 4         128.5    P2p
Gi1/1               Desg FWD 4         128.6    P2p
Gi1/2               Desg FWD 4         128.7    P2p
Gi1/3               Desg FWD 4         128.8    P2p
```

ここで特に見たいのは `Gi0/1 Altn BLK` の行です。これは、**SW3 側では冗長リンクの片側が alternate / blocking になっており、STP がこのポートを止めることでループを防いでいる** ことを表しています。

### `show spanning-tree vlan 1` の見方

- `This bridge is the root` と出ているスイッチが root bridge
- root bridge 以外では、`Root ID` が root bridge を指す
- `Interface Role Sts` の表を見ると、各ポートの役割と状態が分かる

ここでは、**ネットワーク全体でどのスイッチが基準になるか** をまず確認します。

## 検証2: root port / designated port / blocking port を確認する

引き続き、同じ `show spanning-tree vlan 1` の結果を使って役割を見ます。

SW2の注目部分:

```markdown
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p
Gi0/1               Desg FWD 4         128.2    P2p
Gi0/2               Desg FWD 4         128.3    P2p
```

ここでは `Gi0/0 Root FWD` をまず見ます。これは、**SW2 では Gi0/0 が root bridge 方向の root port として forwarding している** ことを表しています。

SW3の注目部分:

```markdown
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p
Gi0/1               Altn BLK 4         128.2    P2p
Gi0/2               Desg FWD 4         128.3    P2p
```

ここでは `Gi0/1 Altn BLK` が重要です。これは、**SW3 側では冗長リンクの片側が alternate / blocking になり、STP がこのポートを止めることでループを防いでいる** と読めます。

- root bridge 側のポート
  - designated port になります
- non-root switch 側の上流向きポート
  - root port になります
- 冗長リンク上のどちらか一方
  - blocking または alternate 側になります

### ここで見るポイント

- `Role` 列
  - `Root` `Desg` `Altn` などが見える
- `Sts` 列
  - `FWD` や `BLK` などが見える

つまり、STP は冗長リンクを全部切るのではなく、**必要な1本だけを止めて木構造にする** と考えると分かりやすいです。

## 検証3: 疎通が維持されることを確認する

PC1で実行:

```text
PC1> ping 192.168.10.12
```

PC1の実行結果:

```markdown
PC1> ping 192.168.10.12

84 bytes from 192.168.10.12 icmp_seq=1 ttl=64 time=11.514 ms
84 bytes from 192.168.10.12 icmp_seq=2 ttl=64 time=12.553 ms
84 bytes from 192.168.10.12 icmp_seq=3 ttl=64 time=10.973 ms
84 bytes from 192.168.10.12 icmp_seq=4 ttl=64 time=13.496 ms
84 bytes from 192.168.10.12 icmp_seq=5 ttl=64 time=11.551 ms
```

ループ防止のためにポートは1本止められますが、**通信経路そのものは残る** ので、PC1とPC2の疎通は維持されます。

## パケットキャプチャしてみるなら

今回は、SW2-SW3 間の冗長リンクでキャプチャすると STP らしさが見えやすいです。

- キャプチャ場所
  - SW2-Gi0/1 と SW3-Gi0/1 の間
- 表示フィルタ
  - `stp`

SW1 Gi0/0キャプチャ結果:
![[スクリーンショット 2026-05-02 165337 1.png]]

このキャプチャでは、赤枠で囲った `Root Identifier` と `Bridge Identifier` が一致していれば、**root bridge 自身が送っているBPDU** だと分かります。

つまり、このBPDUは「自分が root bridge です」という基準情報を、そのまま下流へ流している状態です。

SW3 Gi0/1キャプチャ結果:
![[スクリーンショット 2026-05-02 165518.png]]

こちらでは、`Root Identifier` は root bridge の情報を指し、`Bridge Identifier` はこのBPDUを送っているスイッチ自身の情報を指します。

そのため、両者が一致していなければ、**non-root switch が root bridge の情報を引き継ぎながらBPDUを送っている** と読めます。


ここで見たいのは、**データ通信用には止められる側でも、STP制御用のBPDUは流れている** という点です。

つまり、STP は単にポートを止めて終わりではなく、BPDU を使ってトポロジを維持していると考えると理解しやすいです。

## 分かったこと

- STP は冗長リンクがあるL2ネットワークでループを防ぐための仕組み
- root bridge が基準になり、各スイッチで root port や designated port が決まる
- 冗長リンクの一部は blocking 側になり、木構造が作られる

## まとめ

STPは、冗長性をなくす仕組みではなく、**冗長性を残したままL2ループだけを防ぐ仕組み** と考えるとつかみやすいです。

次は `show spanning-tree` の出力をもう少し丁寧に読みながら、root bridge、root path cost、各ポートの役割を確認していきます。

## 関連ファイル

今回の設定ファイルはGitHubに置いています。

https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/02_spanning_tree_protocol/configs/stp_fundamentals
