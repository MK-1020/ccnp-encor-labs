# CCNP ENCOR 試験対策: CEFとFIBとAdjacency TableでPacket Forwardingを理解する

# EVE-NGで確認するCEF: RIB・FIB・Adjacency Tableの関係

## はじめに

今回は、CCNP ENCORのPacket Forwardingの後半として、CEFをEVE-NGで確認しました。

L2 ForwardingやL3 Forwardingは端末間の通信としてイメージしやすい一方で、CEFは少し抽象的に感じやすい部分だと思います。

ただ、`RIB`、`FIB`、`Adjacency Table` の役割を分けて見ると、ルーターがどのように高速転送を実現しているかが整理しやすくなります。

今回は、`show` コマンドを中心に、RIBとFIBの関係、Adjacency Tableが持つL2情報、ARP解決の前後で何が変わるかを確認しました。

## 今回確認したいこと

- RIBの経路情報をもとにFIBが作られていること
- Adjacency Tableがnext-hopのL2情報を持っていること
- ARP解決の前後でAdjacency Tableの見え方が変わること

## 検証環境

- EVE-NG
- IOSv
- VPCS

## 検証トポロジ

![](https://static.zenn.studio/user-upload/072da7791b09-20260430.png)



## IPアドレス設計

| Node | Interface | IP Address | Note |
|---|---|---|---|
| PC1 | e0 | 192.168.10.10/24 | GW 192.168.10.1 |
| R1 | Gi0/0 | 192.168.10.1/24 | PC1側 |
| R1 | Gi0/1 | 10.12.12.1/30 | R2側 |
| R2 | Gi0/0 | 10.12.12.2/30 | R1側 |
| R2 | Lo0 | 2.2.2.2/32 | 疎通確認先 |

## 設定

今回の構成では、R1からR2のLoopbackへの静的ルートと、R2からPC1側ネットワークへの戻りルートを設定して確認しました。

R1で入力:

```R1
enable
configure terminal
hostname R1
interface gi0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface gi0/1
 ip address 10.12.12.1 255.255.255.252
 no shutdown
ip route 2.2.2.2 255.255.255.255 10.12.12.2
end
write memory
```

R2で入力:

```R2
enable
configure terminal
hostname R2
interface gi0/0
 ip address 10.12.12.2 255.255.255.252
 no shutdown
interface loopback0
 ip address 2.2.2.2 255.255.255.255
ip route 192.168.10.0 255.255.255.0 10.12.12.1
end
write memory
```

このラボでは、派手なトポロジよりも `show` コマンドで見える差を丁寧に追うことを優先しています。

今回見ている表とコマンドの対応は、次のように整理すると分かりやすいです。

- `show ip route` で見ているものが RIB
- `show ip cef` と `show ip cef 2.2.2.2 detail` で見ているものが FIB
- `show adjacency` と `show adjacency detail` で見ているものが Adjacency Table
- `show arp` は、Adjacency Tableが利用するL2解決情報を確認するための表

## 検証1: RIBとFIBの確認

まず、通常のルーティングテーブルとCEFテーブルを確認します。

R1で入力:

```R1
show ip route
show ip cef
show ip cef 2.2.2.2 detail
```

R1の実行結果:

```markdown
R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      2.0.0.0/32 is subnetted, 1 subnets
S        2.2.2.2 [1/0] via 10.12.12.2
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.12.12.0/30 is directly connected, GigabitEthernet0/1
L        10.12.12.1/32 is directly connected, GigabitEthernet0/1
      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, GigabitEthernet0/0
L        192.168.10.1/32 is directly connected, GigabitEthernet0/0
R1#show ip cef
Prefix               Next Hop             Interface
0.0.0.0/0            no route
0.0.0.0/8            drop
0.0.0.0/32           receive
2.2.2.2/32           10.12.12.2           GigabitEthernet0/1
10.12.12.0/30        attached             GigabitEthernet0/1
10.12.12.0/32        receive              GigabitEthernet0/1
10.12.12.1/32        receive              GigabitEthernet0/1
10.12.12.2/32        attached             GigabitEthernet0/1
10.12.12.3/32        receive              GigabitEthernet0/1
127.0.0.0/8          drop
192.168.10.0/24      attached             GigabitEthernet0/0
192.168.10.0/32      receive              GigabitEthernet0/0
192.168.10.1/32      receive              GigabitEthernet0/0
192.168.10.10/32     attached             GigabitEthernet0/0
192.168.10.255/32    receive              GigabitEthernet0/0
224.0.0.0/4          drop
224.0.0.0/24         receive
240.0.0.0/4          drop
255.255.255.255/32   receive
R1#show ip cef 2.2.2.2 detail
2.2.2.2/32, epoch 0
  recursive via 10.12.12.2
    attached to GigabitEthernet0/1
```

ここでは、`show ip route` で見ているRIBと、`show ip cef` で見ているFIBを並べて確認します。

RIBにある `2.2.2.2/32` 宛の経路が、FIBでも転送情報として参照されていることを見るのが目的です。

`show ip cef 2.2.2.2 detail` まで確認すると、単に経路があるだけでなく、`2.2.2.2` 宛て通信がどの next-hop と出力インターフェースへ解決されるかまで追えるので、RIBとFIBの違いが見えやすくなります。

## 検証2: Adjacency Tableの確認

次に、`show adjacency` で見ている Adjacency Table を確認します。

R1で入力:

```R1
show adjacency
show adjacency detail
```

R1の実行結果:

```markdown
R1#show adjacency
Protocol Interface                 Address
IP       GigabitEthernet0/0        192.168.10.10(7)
IP       GigabitEthernet0/1        10.12.12.2(10)
R1#
R1#show adjacency detail
Protocol Interface                 Address
IP       GigabitEthernet0/0        192.168.10.10(7)
                                   9 packets, 882 bytes
                                   epoch 0
                                   sourced in sev-epoch 1
                                   Encap length 14
                                   0050796668015000000200000800
                                   ARP
IP       GigabitEthernet0/1        10.12.12.2(10)
                                   0 packets, 0 bytes
                                   epoch 0
                                   sourced in sev-epoch 2
                                   Encap length 14
                                   5000000300005000000200010800
                                   ARP
```

ここでは、next-hopへフレームを送るためのL2情報がAdjacency Table側にあることを確認します。

つまり、FIBが **どこへ送るか** を持ち、Adjacency Tableが **どうやって送るか** を補っているイメージです。

## 検証3: 疎通発生後の変化

PC1からR2のLoopbackへpingを実行し、その後にCEFやARPを再確認します。

R2には `192.168.10.0/24` への戻りルートも入れて、疎通が成功する前提で確認します。

PC1で実行:

```PC1
PC1> ping 2.2.2.2
```

R1で入力:

```R1
show arp
show ip cef 2.2.2.2 detail
show adjacency detail
```

PC1とR1の実行結果:

```markdown
VPCS> ping 2.2.2.2

84 bytes from 2.2.2.2 icmp_seq=1 ttl=254 time=3.671 ms
84 bytes from 2.2.2.2 icmp_seq=2 ttl=254 time=3.329 ms
84 bytes from 2.2.2.2 icmp_seq=3 ttl=254 time=3.352 ms
84 bytes from 2.2.2.2 icmp_seq=4 ttl=254 time=3.392 ms
84 bytes from 2.2.2.2 icmp_seq=5 ttl=254 time=3.267 ms
```
```cisco
R1#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.12.12.1              -   5000.0002.0001  ARPA   GigabitEthernet0/1
Internet  10.12.12.2              2   5000.0003.0000  ARPA   GigabitEthernet0/1
Internet  192.168.10.1            -   5000.0002.0000  ARPA   GigabitEthernet0/0
Internet  192.168.10.10           1   0050.7966.6801  ARPA   GigabitEthernet0/0
R1#show ip cef 2.2.2.2 detail
2.2.2.2/32, epoch 0
  recursive via 10.12.12.2
    attached to GigabitEthernet0/1
R1#show adjacency detail
Protocol Interface                 Address
IP       GigabitEthernet0/0        192.168.10.10(7)
                                   24 packets, 2352 bytes
                                   epoch 0
                                   sourced in sev-epoch 1
                                   Encap length 14
                                   0050796668015000000200000800
                                   ARP
IP       GigabitEthernet0/1        10.12.12.2(10)
                                   15 packets, 1470 bytes
                                   epoch 0
                                   sourced in sev-epoch 2
                                   Encap length 14
                                   5000000300005000000200010800
                                   ARP
```

ここでは、実際に通信を流したあとに、`show arp` で見ているIP-MAC対応表と、`show adjacency detail` で見ているL2書き換え情報がどう結びついているかを見ます。

`show arp` に next-hop の `10.12.12.2` が載り、`show adjacency detail` に `Encap` 情報が見えるようになれば、ARPで得たL2情報が adjacency に取り込まれていると説明できます。

ここでいう `Encap` は encapsulation のことで、実際にフレームを送るときに使うL2ヘッダー情報を指します。つまり、宛先MACアドレスや送信元MACアドレスを含む「どう包んで送るか」の情報が adjacency に入っていると見れば大丈夫です。

## 検証4: インターフェース停止時の変化

今回は、R2側インターフェースを一時的に停止して、next-hop のL2情報が使えないときに表示がどう変わるかを確認します。

通常状態では `clear arp-cache` を打ってもR1がすぐ next-hop を再学習することがあるため、この方法のほうが差分を観察しやすいです。

まずR2側でインターフェースを停止します。

R2で入力:

```R2
interface gi0/0
 shutdown
```

そのあとR1で確認します。

R1で入力:

```R1
clear arp-cache
show arp
show adjacency detail
```

次に、R2側インターフェースを戻して再度通信を流します。

R2で入力:

```R2
interface gi0/0
 no shutdown
```

PC1で実行:

```PC1
PC1> ping 2.2.2.2
```

R1とPC1の実行結果:

```R1
R1#clear arp-cache
R1#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.12.12.1              -   5000.0002.0001  ARPA   GigabitEthernet0/1
Internet  192.168.10.1            -   5000.0002.0000  ARPA   GigabitEthernet0/0
Internet  192.168.10.10           0   0050.7966.6801  ARPA   GigabitEthernet0/0
R1#
R1#show adjacency detail
Protocol Interface                 Address
IP       GigabitEthernet0/0        192.168.10.10(7)
                                   24 packets, 2352 bytes
                                   epoch 0
                                   sourced in sev-epoch 1
                                   Encap length 14
                                   0050796668015000000200000800
                                   ARP
IP       GigabitEthernet0/1        10.12.12.2(7) (incomplete)
                                   15 packets, 1470 bytes
                                   epoch 0
                                   sourced in sev-epoch 2
                                   punt (rate-limited) packets
                                   no src set
```

```PC1
PC1> ping 2.2.2.2

84 bytes from 2.2.2.2 icmp_seq=1 ttl=254 time=3.104 ms
84 bytes from 2.2.2.2 icmp_seq=2 ttl=254 time=3.210 ms
84 bytes from 2.2.2.2 icmp_seq=3 ttl=254 time=3.163 ms
84 bytes from 2.2.2.2 icmp_seq=4 ttl=254 time=3.247 ms
84 bytes from 2.2.2.2 icmp_seq=5 ttl=254 time=3.087 ms
```

```R1
R1#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.12.12.1              -   5000.0002.0001  ARPA   GigabitEthernet0/1
Internet  10.12.12.2              0   5000.0003.0000  ARPA   GigabitEthernet0/1
Internet  192.168.10.1            -   5000.0002.0000  ARPA   GigabitEthernet0/0
Internet  192.168.10.10           0   0050.7966.6801  ARPA   GigabitEthernet0/0
R1#
R1#show adjacency detail
Protocol Interface                 Address
IP       GigabitEthernet0/0        192.168.10.10(7)
                                   29 packets, 2842 bytes
                                   epoch 0
                                   sourced in sev-epoch 1
                                   Encap length 14
                                   0050796668015000000200000800
                                   ARP
IP       GigabitEthernet0/1        10.12.12.2(10)
                                   20 packets, 1960 bytes
                                   epoch 0
                                   sourced in sev-epoch 2
                                   Encap length 14
                                   5000000300005000000200010800
                                   ARP
```

この確認で、CEFはRIBだけでは完結せず、実際にフレームを作るためにAdjacency Tableの情報も必要であることが見えてきます。

また、next-hop に到達できない状態と復旧後の状態を見比べることで、ARPとAdjacency Tableの関係を追いやすくなります。

## 分かったこと

- RIBは経路情報の元になる表で、**どこへ送るかを決めるための判断材料**になる
- FIBはRIBをもとに作られる高速転送用の表で、**どこへ送るか** をすぐ引ける
- Adjacency Tableは、next-hopへ実際にフレームを送るためのL2書き換え情報を持ち、**どうやって送るか** を補う
- ARP解決が終わると、adjacency に具体的なL2書き換え情報が入り、実際の転送に使える状態になる
- next-hop に到達できなくなると、Adjacency Tableの見え方も変わるため、CEFの転送は到達性とL2解決の両方に依存していると分かる
- CEFは `RIB -> FIB -> Adjacency` と役割を分けることで、毎回CPUで細かく判断しなくても高速に転送できる

## まとめ

今回の検証で、以下を確認できました。

- RIBはルーティング情報の元になる表である
- FIBは転送判断に使う高速な表である
- Adjacency Tableはnext-hopへ送るためのL2情報を持つ
- CEFは `RIB -> FIB -> Adjacency` の連携で高速転送を実現している
- ARPとAdjacency Tableは結びついており、L2情報が揃って初めて実際の転送が成立する
- next-hop に到達できない状態を作ると、Adjacency Tableの見え方も変わる

CEFは、毎回CPUがルーティングテーブルやARPテーブルを見に行くのではなく、転送に必要な情報を事前に整理して高速に使う仕組みだと理解できました。

特に、`RIB -> FIB -> Adjacency` の関係を分けて見ることで、ルーターが **どこへ送るか** と **どうやって送るか** を別々に管理していることがよく分かります。

Packet Forwardingの章の締めとして、かなり重要なラボだと感じました。

## 関連ファイル

今回の設定ファイルはGitHubに置いています。

https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/01_packet_forwarding/configs/cef_fib_adjacency
