# CCNP ENCOR 試験対策 第4回: Packet Forwardingの細かい論点をExtra Checkで整理する

# EVE-NGで確認するPacket Forwarding Extra Check: 第1回〜第3回で拾いきれなかったポイント

## はじめに

今回は、CCNP ENCOR Chapter 1 **Packet Forwarding** の第4回として、第1回から第3回までで拾いきれなかった細かい内容をまとめて確認しました。

これまでの記事では、L2 Forwarding、L3 Forwarding、CEF / FIB / Adjacency という大きな流れを確認しました。

ただ、Chapter 1 にはそれ以外にも **collision domain**、**unknown unicast flooding**、**CEFの最低限の見え方**、**TCAM** や **SDM Template** の位置づけのように、単体では記事1本にしにくいものの、学習の抜けになりやすい項目があります。

そこで今回は、細かい確認を1つの記事にまとめる形でExtra Checkを作りました。

## 今回確認したいこと

- **collision domain** という考え方をスイッチ視点で整理する
- 宛先未学習時の unknown unicast flooding を確認する
- **CEF / Adjacency Table / ARP** の関係を短くおさらいする
- EVE-NGでは見えにくい **TCAM / SDM Template** を、実機でどのコマンドから確認するか整理する

## 検証環境

- EVE-NG
- IOSvL2
- IOSv
- VPCS
- 必要に応じて Wireshark
- 実機 Catalyst 2960-XR

## 検証トポロジ

今回は1本の記事の中で複数の細かい確認を行うため、構成も2パターン使いました。

### 構成A: L2確認用

![[Pasted image 20260501003814.png]]


## 設定

今回は、構成Aと構成Bを同時に作れるように、使う設定を先にまとめておきます。

### 構成A: L2確認用

SW1で入力:

```cisco
enable
configure terminal
hostname SW1
interface range gi0/0 - 2
 switchport mode access
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

PC3で実行:

```text
PC3> set pcname PC3
PC3> ip 192.168.10.13/24
PC3> save
```

## 検証1: Collision Domainを整理する

まずは、Chapter 1で出てくる **collision domain** を整理しました。

今どきの学習環境では、ハブを使って **全員で1本の共有媒体を使う** 状況をそのまま再現することはあまりありませんが、スイッチでは各ポートごとに端末が収容され、ポート単位で学習や転送が行われることを確認しておくのは大事です。

SW1で入力:

```cisco
show mac address-table dynamic
```

SW1の実行結果:

```markdown
SW1#show mac address-table dynamic
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6801    DYNAMIC     Gi0/0
   1    0050.7966.6803    DYNAMIC     Gi0/1
   1    0050.7966.6804    DYNAMIC     Gi0/2
Total Mac Addresses for this criterion: 3
```

今回の EVE-NG / IOSvL2 環境では、`show interfaces status` の表示だけだと、端末がどのポートにつながっているか判断しにくいことがありました。

そのため、この確認では `show mac address-table dynamic` に出てくる学習結果を主な確認材料にして、どの端末がどのポートに収容されているかを確認しています。

ここで重要なのは、スイッチが各端末を **どのポートの先にいるか** という形で認識していることです。

**collision domain** という言葉だけを見ると古い概念にも見えますが、**スイッチは共有媒体ではなく、ポートごとに区切られた転送装置として動いている** と理解する助けになります。

## 検証2: Unknown Unicast / Floodingを確認する

次に、宛先MACアドレスが未学習のときにスイッチがどう動くかを確認しました。

まず、MAC address tableをクリアします。

SW1で入力:

```cisco
clear mac address-table dynamic
show mac address-table dynamic
```

そのあと、端末間で通信を発生させます。

PC1で実行:

```text
PC1> ping 192.168.10.13
```

再度、MAC address tableを確認します。

SW1で入力:

```cisco
show mac address-table dynamic
```

SW1の実行結果:

```markdown
SW1#clear mac address-table dynamic
SW1#show mac address-table dynamic
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
```

PC1の実行結果:

```markdown
PC1> ping 192.168.10.13

84 bytes from 192.168.10.13 icmp_seq=1 ttl=64 time=3.959 ms
84 bytes from 192.168.10.13 icmp_seq=2 ttl=64 time=3.119 ms
84 bytes from 192.168.10.13 icmp_seq=3 ttl=64 time=4.726 ms
84 bytes from 192.168.10.13 icmp_seq=4 ttl=64 time=3.401 ms
84 bytes from 192.168.10.13 icmp_seq=5 ttl=64 time=5.102 ms
```

SW1の実行結果:

```markdown
SW1#show mac address-table dynamic
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0050.7966.6801    DYNAMIC     Gi0/0
   1    0050.7966.6804    DYNAMIC     Gi0/2
```

この確認で分かるのは、最初からスイッチがすべてを知っているわけではなく、通信をきっかけに送信元MACアドレスを学習していくということです。

宛先MACアドレスが分からない間は、同一VLAN内へ広く流す必要があります。これが **unknown unicast flooding** の基本的な考え方です。

## 検証3: CEF / Adjacency / ARP の関係を短くおさらいする

次に、第3回で詳しく見た **CEF** を、短い確認としてもう一度おさらいしました。

ここで見ている表とコマンドの対応は、次のように整理すると分かりやすいです。

- `show ip route` で見ているものが **RIB**
- `show ip cef` で見ているものが **FIB**
- `show adjacency` で見ているものが **Adjacency Table**
- `show arp` は、**Adjacency Table** が利用するL2解決情報を確認するための表

### 構成B: CEF確認用

![[Pasted image 20260501003903.png]]

### 構成B: CEF確認用

R1で入力:

```cisco
enable
configure terminal
hostname R1
interface gi0/0
 ip address 192.168.40.1 255.255.255.0
 no shutdown
interface gi0/1
 ip address 10.12.12.1 255.255.255.252
 no shutdown
ip route 2.2.2.2 255.255.255.255 10.12.12.2
end
write memory
```

R2で入力:

```cisco
enable
configure terminal
hostname R2
interface gi0/0
 ip address 10.12.12.2 255.255.255.252
 no shutdown
interface loopback0
 ip address 2.2.2.2 255.255.255.255
ip route 192.168.40.0 255.255.255.0 10.12.12.1
end
write memory
```

PC4で実行:

```text
PC4> set pcname PC4
PC4> ip 192.168.40.10/24 192.168.40.1
PC4> save
```


R1で入力:

```cisco
show ip route
show ip cef
show adjacency
show arp
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
      192.168.40.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.40.0/24 is directly connected, GigabitEthernet0/0
L        192.168.40.1/32 is directly connected, GigabitEthernet0/0
R1#
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
192.168.40.0/24      attached             GigabitEthernet0/0
192.168.40.0/32      receive              GigabitEthernet0/0
192.168.40.1/32      receive              GigabitEthernet0/0
192.168.40.10/32     attached             GigabitEthernet0/0
192.168.40.255/32    receive              GigabitEthernet0/0
224.0.0.0/4          drop
224.0.0.0/24         receive
240.0.0.0/4          drop
255.255.255.255/32   receive
R1#
R1#show adjacency
Protocol Interface                 Address
IP       GigabitEthernet0/0        192.168.40.10(7)
IP       GigabitEthernet0/1        10.12.12.2(10)
R1#
R1#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.12.12.1              -   5000.0006.0001  ARPA   GigabitEthernet0/1
Internet  10.12.12.2              7   5000.0007.0000  ARPA   GigabitEthernet0/1
Internet  192.168.40.1            -   5000.0006.0000  ARPA   GigabitEthernet0/0
Internet  192.168.40.10           0   0050.7966.6805  ARPA   GigabitEthernet0/0
```

### `show ip cef` の見方

ここで `show ip cef` の **Next Hop** 列を補足しておくと、表示の意味が追いやすくなります。

- `2.2.2.2/32 -> 10.12.12.2` は、宛先 `2.2.2.2` に送るとき、R1は **次ホップ 10.12.12.2** に向けて `GigabitEthernet0/1` から転送する、という意味です。
- `attached` は、そのプレフィックスが **送信インターフェースに直結している** ことを表します。たとえば `192.168.40.10/32 attached` は、同一セグメント上の相手としてL2解決して送れる状態です。
- `receive` は、そのアドレスあてのパケットを **ルータ自身が受信処理する** ことを表します。自分のインターフェースIPやブロードキャストに対してよく見えます。
- `drop` は、CEF上で **破棄するエントリ** です。予約アドレス帯などに対して見えることがあります。
- `no route` は、その宛先に対して **転送先が見つかっていない** 状態です。

つまり、`show ip cef` では単に「経路があるか」だけでなく、**次にどこへ渡すか**、または **自分で受けるのか / 捨てるのか** まで含めて転送動作が表現されています。

ここで改めて見えてくるのは、**どこへ送るか** を決める表と、**どうやって送るか** を支える表が分かれていることです。

**RIB** が経路情報の元になり、**FIB** が高速転送に使われ、**Adjacency Table** がL2情報を補います。さらに、そのL2情報の一部は **ARP** による解決結果に依存しています。

これまでの記事で一度理解した内容を、こうして短く再確認しておくと知識が定着しやすいです。

参考までに、今回 `show ip cef` に見えていた予約アドレス帯を簡潔に整理すると次のとおりです。

| アドレス帯 | 用途 | CEF上で見えた動き |
| --- | --- | --- |
| `127.0.0.0/8` | ループバック用 | `drop` |
| `224.0.0.0/4` | マルチキャスト用 | `drop` |
| `224.0.0.0/24` | ローカルセグメント制御用マルチキャスト | `receive` |
| `240.0.0.0/4` | 予約済み範囲 | `drop` |
| `255.255.255.255/32` | 限定ブロードキャスト | `receive` |

## 検証4: TCAM / SDM Templateを実機でどこから見るか整理する

最後に、EVE-NGでは見えにくい **TCAM** と **SDM Template** を、実機 Catalyst 2960-XR でどのコマンドから確認するか整理しました。

ここは設定変更ではなく、**どこを見ればよいか** を把握するための確認です。

この検証で見たいのは、**スイッチ内部の資源配分がどのように決まっているか** と、**その資源が今どれくらい使われているか** です。

**SDM Template** は、スイッチの資源をL2寄り、IPv4寄り、バランス型のどれに近い形で割り当てるかを決める考え方です。

**TCAM** は、ACL、QoS、ルーティング、MAC学習のような高速判定を支えるハードウェア資源です。

### まず機種とIOSを確認する

実機で入力:

```cisco
show version
```

実機の実行結果:

```markdown
C2960-2#show version
Cisco IOS Software, C2960X Software (C2960X-UNIVERSALK9-M), Version 15.2(7)E0a, RELEASE SOFTWARE (fc1)
Technical Support: http://www.cisco.com/techsupport
Copyright (c) 1986-2019 by Cisco Systems, Inc.
Compiled Fri 12-Apr-19 04:09 by prod_rel_team

ROM: Bootstrap program is C2960X boot loader
BOOTLDR: C2960X Boot Loader (C2960X-HBOOT-M) Version 15.2(6r)E, RELEASE SOFTWARE (fc1)

C2960-2 uptime is 34 minutes
System returned to ROM by power-on
System restarted at 12:04:32 UTC Sat May 2 2026
System image file is "flash:c2960x-universalk9-mz.152-7.E0a.bin"
Last reload reason: power-on

cisco WS-C2960XR-24TS-I (APM86XXX) processor (revision X0) with 524288K bytes of memory.
Last reset from power-on
2 Virtual Ethernet interfaces
1 FastEthernet interface
28 Gigabit Ethernet interfaces
2 Ten Gigabit Ethernet interfaces
The password-recovery mechanism is enabled.

512K bytes of flash-simulated non-volatile configuration memory.
Model revision number           : X0
Motherboard revision number     : C0
Model number                    : WS-C2960XR-24TS-I
Top Assembly Part Number        : 68-100318-03
Top Assembly Revision Number    : K0
Version ID                      : V06
CLEI Code Number                : CMMK110ARE
Daughterboard revision number   : B0
Hardware Board Revision Number  : 0x22


Switch Ports Model                     SW Version            SW Image           
------ ----- -----                     ----------            ----------         
*    1 30    WS-C2960XR-24TS-I         15.2(7)E0a            C2960X-UNIVERSALK9-M


Configuration register is 0xF
```

※ 公開向けに、MACアドレスや各種シリアル番号などの個体識別情報につながる部分は省略しています。

ここでは全行を細かく読むというより、`WS-C2960XR-24TS-I` と `15.2(7)E0a` を見て、**どの機種・どのIOSで確認した結果か** を押さえるのが主な目的です。

### 現在のSDM Templateを確認する

まず、設定でSDM Templateを明示変更しているかを確認します。

実機で入力:

```cisco
show running-config | include sdm
```

実機の実行結果:

```markdown
C2960-2#show running-config | include sdm
```

今回は何も返っていません。これは、**running-config上でSDM Templateを明示変更していない** と読めます。

続けて、現在どのテンプレートが有効かを確認します。

実機で入力:

```cisco
show sdm prefer
```

実機の実行結果:

```markdown
C2960-2#show sdm prefer
 The current template is "default" template.
 The selected template optimizes the resources in
 the switch to support this level of features for
 8 routed interfaces and 1024 VLANs.

  number of unicast mac addresses:                  24K
  number of IPv4 IGMP groups + multicast routes:    1K
  number of IPv4 unicast routes:                    5.25K
    number of directly-connected IPv4 hosts:        4K
    number of indirect IPv4 routes:                 1.25K
  number of IPv6 multicast groups:                  1K
  number of IPv6 unicast routes:                    5.25K
    number of directly-connected IPv6 addresses:    4K
    number of indirect IPv6 unicast routes:         1.25K
  number of IPv4 policy based routing aces:         0.5K
  number of IPv4/MAC qos aces:                      0.5K
  number of IPv4/MAC security aces:                 1K
  number of IPv6 policy based routing aces:         0.25K
  number of IPv6 qos aces:                          0.25K
  number of IPv6 security aces:                     0.5K
```

ここでまず見るのは `The current template is "default" template.` の行です。これで、**現在は default template が有効** だと分かります。

その下の `number of ...` は、そのテンプレートでどの機能にどれくらい資源を割り当てる想定かを示しています。つまり `show sdm prefer` は、**今の配分方針** を見るコマンドです。

### テンプレートごとの差を見比べる

次に、`default`、`ipv4`、`vlan` の各テンプレートがどんな配分になるかを見比べます。

実機で入力:

```cisco
show sdm prefer default
show sdm prefer ipv4
show sdm prefer vlan
```

実機の実行結果:

```markdown
C2960-2#show sdm prefer default
 "default" template:
 The selected template optimizes the resources in
 the switch to support this level of features for
 8 routed interfaces and 1024 VLANs.

  number of unicast mac addresses:                  24K
  number of IPv4 IGMP groups + multicast routes:    1K
  number of IPv4 unicast routes:                    5.25K
    number of directly-connected IPv4 hosts:        4K
    number of indirect IPv4 routes:                 1.25K
  number of IPv6 multicast groups:                  1K
  number of IPv6 unicast routes:                    5.25K
    number of directly-connected IPv6 addresses:    4K
    number of indirect IPv6 unicast routes:         1.25K
  number of IPv4 policy based routing aces:         0.5K
  number of IPv4/MAC qos aces:                      0.5K
  number of IPv4/MAC security aces:                 1K
  number of IPv6 policy based routing aces:         0.25K
  number of IPv6 qos aces:                          0.25K
  number of IPv6 security aces:                     0.5K

C2960-2#show sdm prefer ipv4
 "ipv4" template:
 The selected template optimizes the resources in
 the switch to support this level of features for
 8 routed interfaces and 1024 VLANs.

  number of unicast mac addresses:                  16K
  number of IPv4 IGMP groups + multicast routes:    1K
  number of IPv4 unicast routes:                    24K
    number of directly-connected IPv4 hosts:        16K
    number of indirect IPv4 routes:                 8K
  number of IPv6 multicast groups:                  0
  number of IPv6 unicast routes:                    0
    number of directly-connected IPv6 addresses:    0
    number of indirect IPv6 unicast routes:         0
  number of IPv4 policy based routing aces:         0.375k
  number of IPv4/MAC qos aces:                      0.5K
  number of IPv4/MAC security aces:                 0.875k
  number of IPv6 policy based routing aces:         0
  number of IPv6 qos aces:                          0
  number of IPv6 security aces:                     60

C2960-2#show sdm prefer vlan
 "vlan" template:
 The selected template optimizes the resources in
 the switch to support this level of features for
 8 routed interfaces and 1024 VLANs.

  number of unicast mac addresses:                  32K
  number of IPv4 IGMP groups + multicast routes:    1K
  number of IPv4 unicast routes:                    0.5K
    number of directly-connected IPv4 hosts:        0.25K
    number of indirect IPv4 routes:                 0.25K
  number of IPv6 multicast groups:                  1K
  number of IPv6 unicast routes:                    0.5K
    number of directly-connected IPv6 addresses:    0.25K
    number of indirect IPv6 unicast routes:         0.25K
  number of IPv4 policy based routing aces:         0.5K
  number of IPv4/MAC qos aces:                      0.5K
  number of IPv4/MAC security aces:                 1K
  number of IPv6 policy based routing aces:         0
  number of IPv6 qos aces:                          0.5K
  number of IPv6 security aces:                     0.5K
```

ここでは、各テンプレートの `number of unicast mac addresses` と `number of IPv4 unicast routes` を見比べると分かりやすいです。

- `default`
  - MAC、IPv4、IPv6をバランスよく配分している
- `ipv4`
  - IPv4ルート数が大きく増え、IPv6関連がかなり減る
- `vlan`
  - MACアドレス数が増え、L2寄りの配分になる

つまり、**どの機能を重視するかで、同じスイッチでも資源配分の考え方が変わる** ことが見えてきます。

### 現在のTCAM使用量を確認する

最後に、その配分の中で今どれくらい使っているかを確認します。

実機で入力:

```cisco
show platform tcam utilization
```

実機の実行結果:

```markdown
C2960-2#show platform tcam utilization

CAM Utilization for ASIC# 0                      Max            Used
                                             Masks/Values    Masks/values

 Unicast mac addresses:                      24796/24796        18/18
 IPv4 IGMP groups + multicast routes:         1072/1072          1/1
 IPv4 unicast directly-connected routes:      4096/4096          0/0
 IPv4 unicast indirectly-connected routes:    1280/1280         34/34
 IPv6 Multicast groups:                       1072/1072         11/11
 IPv6 unicast directly-connected routes:      4096/4096          0/0
 IPv6 unicast indirectly-connected routes:    1280/1280         30/30
 IPv4 policy based routing aces:               512/512          14/14
 IPv4 qos aces:                                512/512          51/51
 IPv4 security aces:                          1024/1024         77/77
 IPv6 policy based routing aces:               256/256           8/8
 IPv6 qos aces:                                256/256          44/44
 IPv6 security aces:                           512/512          18/18

Note: Allocation of TCAM entries per feature uses
a complex algorithm. The above information is meant
to provide an abstract view of the current TCAM utilization
```

ここでは、`Max` と `Used` の列を見ます。

- `Max`
  - その項目に割り当てられている最大数
- `Used`
  - 今実際に使っている数

たとえば `IPv4 unicast indirectly-connected routes: 1280/1280 34/34` は、**その用途に1280個ぶん確保され、現在34個を使っている** という意味です。

`show sdm prefer` が **どう配分するか** を見るコマンドで、`show platform tcam utilization` が **その中で今どれくらい使っているか** を見るコマンドだと考えると、役割の違いが分かりやすいです。

この段階では、設定変更までは行わず、あくまで **何が見えるのか** を把握するところまでに留めるのが安全です。

ここでいう「安全」は、**壊してはいけないから触るな** という意味ではなく、**確認のための検証としては、表示を見るだけで十分に学べる** という意味です。

**SDM Template** を変更すると、機種によっては reload が必要になったり、ルーティング用、MAC学習用、ACL用などの資源配分が変わったりします。

検証機であれば実際に変更して試すこと自体は問題ありませんが、その場合は **変更前後で何が増えて何が減るのか** を意識して見ないと、ただ再起動して終わりになりやすいです。

そのため、この段階ではまず `show sdm prefer` と `show platform tcam utilization` で **今どう配分され、今どれだけ使っているか** を把握してから、必要なら次の検証としてテンプレート変更を試す、という順番の方が理解しやすいと感じました。

## 分かったこと

- **collision domain** という言葉は、スイッチがポート単位で端末を分離していることを理解する助けになる
- スイッチは通信をきっかけに送信元MACアドレスを学習し、宛先未知の間は同一VLANへ flood する
- **CEF** では、**FIB** と **Adjacency Table** が役割分担しながら高速転送を行う
- `show` コマンドで見ている表が何なのかを整理すると、試験用語と実機出力がつながりやすい
- **TCAM** や **SDM Template** は、EVE-NGだけではなく実機で確認した方が理解しやすい

## まとめ

今回のExtra Checkでは、これまでの記事では主役にしにくかった細かい項目をまとめて確認しました。

こうした小さな確認は、単体では記事1本にしづらい一方で、放置すると試験や実務で曖昧さが残りやすい部分でもあります。

特にChapter 1は、後続のルーティング、スイッチング、QoS、セキュリティの土台になる章なので、**分かったつもり** のまま進まないように細かい項目まで回収しておく価値があると感じました。

第1回から第3回で大枠を掴み、Extra Checkで細部を拾う形にすると、Packet Forwardingの章をかなり抜け漏れなく学習できそうです。

## 関連ファイル

今回の設定ファイルはGitHubに置いています。

https://github.com/MK-1020/ccnp-encor-labs/tree/main/ccnp_guided_lab_packets/01_packet_forwarding/configs/extra_check_packet_forwarding
